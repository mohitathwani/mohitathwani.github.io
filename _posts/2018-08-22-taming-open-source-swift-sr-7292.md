---
title: 'Taming Open Source Swift (SR-7292)'
date: 2018-08-22 10:00:00
featured_image: '/images/swift-og.png'
categories: [Open Source, Swift]
redirect_from: open-source/taming-open-source-swift-sr-7292/
---

In December 2015, Apple made Swift an open source language and made it available on [Github](https://github.com/apple/swift). This piqued my interest, I dug around the code base for a little, cloned it and felt like I had achieved something great. Fast forward to Spring 2017, I made my first contribution to the [swift-corelibs-foundation](https://github.com/apple/swift-corelibs-foundation), two contributions to be precise. The way I made these contributions was by digging around the code base for the `NSString` class and implementing methods marked as `NSUnimplemented()`. With two accepted PRs to swift-corelibs-foundation, the next thing I did was opened up my resume and under the Key Projects section, added a new bullet point right on top – Currently contribute to Apple’s open source Swift project on Github.

Fast forward to Fall 2017, my graduation semester, I started reaching out to recruiters from the Big N companies hoping that the latest addition to my resume would fetch my some brownie points, and it indeed did. I cleared the phone screens, made it to the onsite rounds and walked out completely disappointed with myself and the broken interview process. I didn’t much feel like an imposter before but failing the interviews has hard coded the string I am an IMPOSTER in my neural network!

I took to [Reddit](https://www.reddit.com/r/swift/comments/8dgvej/ive_been_developing_ios_apps_since_2010_and_have/) to seek advice and the general advice was that if you want to work with the very best, contribute to open source projects and that rekindled the spark in me to come back to Swift, only this time I wanted to be a part of the major league, the core swift repository.

Snooping around the internet, I came across Erica Sadun’s article titled [Writing Swift: Adventure in Compiler Mods](https://ericasadun.com/2018/04/15/writing-swift-adventures-in-compiler-mods/). In this article, Erica describes in detail her approach for adding a `#dogcow` directive which will tell the compiler to insert `🐶🐮` wherever it saw that directive. I followed along but unfortunately never got it to work. The up side of this exercise was that I had read more code about the Swift compiler than I ever had in my life. Although `#dogcow` is not the focus of Erica’s article, she definitely gives a noob a starting point. Thanks Erica!

Luckily for me, just about the time I was starting to dig in to this project, [Harlan Haskins](https://twitter.com/harlanhaskins) posted a [video](https://www.reddit.com/r/swift/comments/8dqok2/the_video_of_my_talk_with_ucodafi_from/) to a talk he did with [/u/CodaFi](https://www.reddit.com/user/CodaFi) in which he explains the Swift repository project structure, and then /u/CodaFi walks us through fixing a compiler crasher. Thanks to this video I had now learn’t how the repository was structured and learn’t about a very handy tool called [lit](https://llvm.org/docs/CommandGuide/lit.html) which is a part of LLVM.

Everything I talk about in this article was done on a Mac, so to get started, head over to the [official repository](https://github.com/apple/swift) and follow the instructions to clone the project. First you will need to get `cmake` and `ninja` and I personally followed the instruction to clone with SSH. After setting up the `swift-source` directory, `cd` in to it and follow as indicated.

Cloning this repository will only get you the core Swift code base. To be able to build it and have a running executable, you will have to fetch the dependencies i.e Foundation, GCD, etc.. using the `update-checkout` tool present in the `utils` directory in the cloned `swift` directory. After this step, the final step is to run the `build-script` tool in the `utils` directory. Harlan Haskins explains the various flags that can be passed to the `build-script` and taking his advice, I pass the `--debug --xcode` flags to build for debug mode and to generate an Xcode project. Building the project for the very first time can be a very time consuming process.

As Harlan mentions in his video, there are some 700 Xcode schemes that are generated, I went ahead and selected the `swift` and `swift-refactor` schemes. Why `swift-refactor`? Read on to find out…

At this point, I highly recommend you to read the complete README file on the home page of the Github repository and also this [Testing.md](https://github.com/apple/swift/blob/master/docs/Testing.md) which talks about how to go about testing the code base.

After being able to clone and build the project, I started looking for `StarterBug` issues on the [Swift Bugs](https://bugs.swift.org/) website and came across [SR-7292](https://bugs.swift.org/browse/SR-7292). The requirement for this issue is to build a refactoring tool which will automatically generate a memberwise initializer when you right click on a class declaration and choose to apply this refactoring. Thanks to the course on compiler’s I studied in grad school, I was aware of the concept of a syntax tree and to work on this issue, I would have to figure out how to walk the syntax tree, collect all the data members of the class and I would be done! But now the question was where do I begin? One very crucial aspect of contributing to a ginormous open source project like Swift is that you have to accept the fact that you are here to learn and you must reach out for guidance and that’s exactly what I did. The reporter of this issue, [Xi Ge](https://twitter.com/xge_apple) pointed me to an article which he wrote on how to build such a refactoring tool [here](https://swift.org/blog/swift-local-refactoring/) and [here](https://github.com/apple/swift/blob/master/docs/refactoring/SwiftLocalRefactoring.md). Both these links refer to the same article and at this point, I recommend that you go and read it and then come back and read along.

I think it’s time we start doing some code now.

So the first thing we have to do is add a `CURSOR_REFACTORING()` entry to [RefactoringKinds.def](https://github.com/apple/swift/blob/60a91bb7360dde5ce9531889e0ed10a2edbc961a/include/swift/IDE/RefactoringKinds.def).

```c++
CURSOR_REFACTORING(MemberwiseInitLocalRefactoring, "Generate Memberwise Initializer", memberwise.init.local.refactoring)
```

The string *“Generate Memberwise Initializer”* will be presented to the user in Xcode if the user is trying to apply this refactoring at a valid location in their source code. How do we define a valid location? We will discuss this below. The first parameter `MemberwiseInitLocalRefactoring` is used by the toolchain to automatically generate a class `RefactoringActionMemberwiseInitLocalRefactoring` for us.

The next step is to implement the `bool RefactoringActionMemberwiseInitLocalRefactoring::isApplicable(ResolvedCursorInfo Tok, DiagnosticEngine &amp;Diag)` method:

```c++
bool RefactoringActionMemberwiseInitLocalRefactoring::
isApplicable(ResolvedCursorInfo Tok, DiagnosticEngine &Diag) {
  
  SmallVector<std::string, 8> memberNameVector; //1
  SmallVector<std::string, 8> memberTypeVector; //2
  
  return collectMembersForInit(Tok, memberNameVector,
                               memberTypeVector).isValid(); //3
}
```

1. A vector to collect the names of all the data members of the class.
2. A vector to collect the data types of the members of the class in same order as the previous vector so that things don’t mismatch.
3. The call to collectMembersForInit() will return an instance of SourceLoc which is checked for validity and this boolean is returned.

As you can see the `isApplicable()` method is pretty simple. All the logic has been put in to the `collectMembersForInit()` method which is as follows:

```c++
static SourceLoc collectMembersForInit(ResolvedCursorInfo CursorInfo,
                           SmallVectorImpl<std::string>& memberNameVector,
                           SmallVectorImpl<std::string>& memberTypeVector) {
  
  if (!CursorInfo.ValueD) //1
    return SourceLoc();
  
  ClassDecl *classDecl = dyn_cast<ClassDecl>(CursorInfo.ValueD); //2
  if (!classDecl || classDecl->getStoredProperties().empty() ||
      CursorInfo.IsRef) {
    return SourceLoc();
  }
  
  SourceLoc bracesStart = classDecl->getBraces().Start; //3
  if (!bracesStart.isValid())
    return SourceLoc();
  
  SourceLoc targetLocation = bracesStart.getAdvancedLoc(1); //4
  if (!targetLocation.isValid())
    return SourceLoc();
  
  for (auto varDecl : classDecl->getStoredProperties()) { //5
    auto parentPatternBinding = varDecl->getParentPatternBinding(); //6
    if (!parentPatternBinding)
      continue;
    
    auto varDeclIndex =
      parentPatternBinding->getPatternEntryIndexForVarDecl(varDecl); //7
    
    if (auto init = varDecl->getParentPatternBinding()->getInit(varDeclIndex)) { //8
      if (init->getStartLoc().isValid())
        continue;
    }
    
    StringRef memberName = varDecl->getName().str(); //9
    memberNameVector.push_back(memberName.str());
    
    std::string memberType = varDecl->getType().getString(); //10
    memberTypeVector.push_back(memberType);
  }
  
  if (memberNameVector.empty() || memberTypeVector.empty()) { //11
    return SourceLoc();
  }
  
  return targetLocation; //12
}
```

1. First we check that the cursor is at an instance of a `ValueDecl`.
2. We try to dynamically cast the instance of `ValueDecl` to a `ClassDecl` which is a derived type of `ValueDecl`. The important thing to note here is the conditional `CursorInfo.IsRef`, this is needed because we do not want this refactoring to be applicable on lines where an instance of this is being created for example `let x = Person()`.
3. We try to get the location of the opening brace `{` after the class declaration from the source code.
4. We now get the target location in the source code where our generated initializer will be injected.
5. We now iterate over the list of stored properties, which are instances of the `VarDecl` class.
6. For the corresponding `VarDecl`, we get the `PatternBindingDecl` which according to the documentation contains a pattern and optional initializer for a set of one or more `VarDecl`s declared together. This is important to us because, if a stored property has already been initialized at the point of declaration, there is no need for us to include it in our memberwise initializer.
7. Within the parent pattern binding, we get the index of the current `VarDecl`.
8. Here we check if the property has been initialized or not.
9. We get the name of the member and push it on to the vector.
10. We get the type of the member as a string and push it on to the vector.
11. If these vectors are empty, we return a default instance of `SourceLoc()` which starts out to be invalid.
12. If everything is fine, we return the `targetLocation`.

So far all we have done is checked if this refactoring is applicable for the location of the cursor in the source code. The next phase of this task is to actually apply the refactoring if the user should decide to do so and to do this, we must implement `bool RefactoringActionMemberwiseInitLocalRefactoring::performChange()`

```c++
bool RefactoringActionMemberwiseInitLocalRefactoring::performChange() {
  
  SmallVector<std::string, 8> memberNameVector;
  SmallVector<std::string, 8> memberTypeVector;
  
  SourceLoc targetLocation = collectMembersForInit(CursorInfo, memberNameVector,
                                         memberTypeVector);
  if (targetLocation.isInvalid())
    return true;
  
  generateMemberwiseInit(EditConsumer, SM, memberNameVector,
                         memberTypeVector, targetLocation);
  
  return false;
}
```

As you can see, this function is somewhat a repetition of the previous code sample and the bulk of this logic is implemented in the `generateMemberwiseInit()` function. One important thing to note here is the return values for this function. You will return `true` if the refactoring is not able to perform the change to the source code and return `false` if the refactoring did indeed perform a change to the source. I wonder why the core team decided to do it this way.

Moving on, here’s the code for `generateMemberwiseInit()`:

```c++
static void generateMemberwiseInit(SourceEditConsumer &EditConsumer,
                            SourceManager &SM,
                            SmallVectorImpl<std::string>& memberNameVector,
                            SmallVectorImpl<std::string>& memberTypeVector,
                            SourceLoc targetLocation) {
  
  assert(!memberTypeVector.empty()); //1
  assert(memberTypeVector.size() == memberNameVector.size());
  
  EditConsumer.accept(SM, targetLocation, "\ninternal init("); //2
  
  for (size_t i = 0, n = memberTypeVector.size(); i < n ; i++) { //3
    EditConsumer.accept(SM, targetLocation, memberNameVector[i] + ": " +
                        memberTypeVector[i]);
    
    if (i != memberTypeVector.size() - 1) { //4
      EditConsumer.accept(SM, targetLocation, ", ");
    }
  }
  
  EditConsumer.accept(SM, targetLocation, ") {\n"); //5
  
  for (auto varName: memberNameVector) { //6
    EditConsumer.accept(SM, targetLocation,
                        "self." + varName + " = " + varName + "\n");
  }
  
  EditConsumer.accept(SM, targetLocation, "}\n"); //7
}
```

1. Just some sanity checks to ensure the code below will not fail.
2. Write the text `internal init(` to the source code buffer at the `targetLocation` that was computed earlier.
3. Write out the parameter list for the initializer.
4. Manage the trailing comma for the parameter list
5. Write out `) {` followed by a new line to the source buffer.
6. Go through the members of the class and write out the text assigning them to the self version of that member in the class.
7. Finally write out `}` followed by a new line to the source buffer.

At this point you are probably wondering how did I figure out all the APIs that I used. Here’s how:

1. Look around surrounding code and see what others have done before you.
2. Cmd-click on a symbol in Xcode will take you to the definition of that symbol. Cmd-click your way up the inheritance chain until things start making sense.
3. Breakpoints! Before I started writing any code, I set up breakpoints in Xi’s implementation of generating a localized string and understood how things are flowing.
4. The most important, ASK THE COMMUNITY!

Time for some examples. Given the following class structure:Time for some examples. Given the following class structure:

```swift
class Person {
  var firstName: String!
  var lastName: String!
  var age: Int!
  var planet = "Earth", solarSystem = "Milky Way"
  var avgHeight = 175
}
```

the resulting change after applying the refactoring will be:

```swift
class Person {
internal init(firstName: String?, lastName: String?, age: Int?) {
self.firstName = firstName
self.lastName = lastName
self.age = age
}

  var firstName: String!
  var lastName: String!
  var age: Int!
  var planet = "Earth", solarSystem = "Milky Way"
  var avgHeight = 175
}
```

Don’t worry about the formatting as all we are doing is inserting characters in to the source code buffer and Xcode will automatically apply the formatting as needed.

For the following example, if the user right clicks on the `Person` symbol on line 9, the list of refactoring options will not have `Generate Memberwise Initializer` and so no change will be performed.

```swift
class Person {
  var firstName: String!
  var lastName: String!
  var age: Int!
  var planet = "Earth", solarSystem = "Milky Way"
  var avgHeight = 175
}

let _ = Person()
```

The last pice of the puzzle is to add the following line to `swift-refactor.cpp` to make the `swift-refactor` tool aware of the `memberwise-init` flag that will passed to it:

```c++
clEnumValN(RefactoringKind::MemberwiseInitLocalRefactoring, "memberwise-init", "Generate member wise initializer")));
```

And now to run and debug and do whatever you need to do, select the `swift-refactor` scheme and edit it and add the following run time arguments and you should be good to go.

```c++
-memberwise-init -source-filename <path to source file> -pos=<line>:<column>
```

A complete test input file would look something like below. The commented lines towards the end of the file are used by the test runner and is giving it information on how to run the test and where to look for the file that contains the expected results.

```swift
class Person {
  var firstName: String!
  var lastName: String!
  var age: Int!
  var planet = "Earth", solarSystem = "Milky Way"
  var avgHeight = 175
}

// RUN: %empty-directory(%t.result)
// RUN: %refactor -memberwise-init -source-filename %s -pos=1:8 > %t.result/class_members.swift
// RUN: diff -u %S/Outputs/class_members/class_members.swift.expected %t.result/class_members.swift
```

Before you submit your pull request, I recommend that you run the `build-script` mentioned above with the `-t` flag to run all 4000+ test cases to ensure you haven’t broken anything else.

I have tried to cover everything I can about this feature that I implemented with the help of members from the community. Here is the [link](https://github.com/apple/swift/pull/16983) to the complete source code for this pull request. As you can see there was a lot of back and forth with the moderators.

I would like to thank [Xi Ge](https://github.com/nkcsgexi),  [Harlan Haskins](https://github.com/harlanhaskins), [Rintaro Ishizaki](https://github.com/rintaro) and [Don Sleeter](https://github.com/dndydon) for guiding me with this PR, and doing a thorough a code review.

For anybody out there struggling with contributing to Swift, feel free to reach out to me ☺️.