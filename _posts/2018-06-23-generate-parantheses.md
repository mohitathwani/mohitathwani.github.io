---
title: 'Generate Parantheses'
date: 2018-06-23 10:00:00
featured_image: '/images/parantheses.png'
categories: [Algorithms, Backtracking, Recursion, String, Leetcode]
redirect_from: /leetcode/leetcode-22-generate-parentheses/
---

Before we start solving this problem, let’s do some math first. The problem statement clearly states that we have *N* pairs of parentheses to work with. That means for a string to be considered as part of the solution, it must be of size *2N*.

Once we have generated a string of size *2N*, for it to actually be a part of the solution, we will have to check if there is a balance in the opening and closing parentheses. If these conditions are met, the string is a part of the solution.

How many strings can be generated? At every index in the string we have 2 options, i.e. either a *(* or a *)* and we have to make this selection *2N* times, that gives us a total of *2^2N* strings.

For example, with *N = 3*, the length of the string should be 6 and there will be *2x2x2x2x2x2* or *2^6* strings that can be generated. Once a string has been generated, it will take us *O(N)* time to scan the string and validate it.

This crude algorithm will give us a time complexity of *O(Nx2^2N)* where *N* comes from scanning and *2^2N* comes from generating. The space complexity can be limited to *O(2^2N)* for holding all the generated strings.

To make this solution more efficient, we should come up with a way that allows us to check if it’s okay to add a *(* or a *)* and this can be done by keeping *count*. We know for a fact that at any given point the number of *(* has to be less than or equal to *N* and for the string to be balanced, the number of *)* has to be less than or equal to *(*. This is the crucial part. I messed up my initial implementation of the code by assuming that *)* also has to be less than or equal to *N* and as a result I started generated unbalanced strings.

This is a classic *backtracking* problem and can be solved using *recursion*. The code to do this looks something like:

```swift
func generateParenthesis(_ n: Int) -> [String] {
  var result = [String]()
  var s: String = ""
  
  func backtrack(numOpen:Int = 0, numClose: Int = 0) {
    if s.count == 2 * n {
      result.append(s)
      return
    }
    
    if numOpen < n {
      s.append("(")
      backtrack(numOpen: numOpen + 1, numClose: numClose)
      s.removeLast()
    }
    
    if numClose < numOpen {
      s.append(")")
      backtrack(numOpen: numOpen, numClose: numClose + 1)
      s.removeLast()
    }
  }
  
  backtrack()
  return result
}
```

Time and Space Complexity: We have only generated only those strings that will be a part of the valid solution which is essentially a subset of the *2^2N* elements. It turns out this is the *Nth Catalan number* which is basically the kind of math I am unfamiliar with! Leetcode has a not so detailed instruction on this.

Code hosted on [Github](https://github.com/mohitathwani/SwiftCodingChallenges/blob/master/generateParantheses/GenerateParantheses.playground/Contents.swift).