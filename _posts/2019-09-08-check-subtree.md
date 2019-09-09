---
title: 'Check if a Tree is a sub tree of another'
date: 2019-09-08 10:00:00
featured_image: '/images/subtree.jpg'
categories: [Algorithms, Tree, Graph, DFS]
---

## Problem Statement
There exists two very large binary trees, with one tree, `T1` relatively much bigger than the other `T2`. How can we efficiently determine if `T2` is a sub tree of `T1`?

***Note***: A tree `T2` is a sub tree of `T1` if and only if there is a node in `T1` which when cut off from `T1`, the tree rooted at that node is exactly identical to `T2` in terms of data and structure.

#### Example
```
T1 =                     T2 =                T3 = 
        5                 3                   3 
      /   \              / \                 / \
     3     6            2   4               1   4
    / \                /                     \  
   2   4              1                       2
  /
 1

 T2 is a sub tree of T1 but T3 is not.
```

## Explanation

Before we determine if `T2` is a sub tree of `T1`, let's ask ourselves if we know how can we determine if a string `S2` is a sub string of a string `S1`? There is a very efficient pattern matching algorithm that exist, it is also one that I am yet to get a good grasp on. It is the [Knuth-Morris-Pratt](https://en.wikipedia.org/wiki/Knuth–Morris–Pratt_algorithm) algorithm. 

I will cover the `KMP` algorithm at some other time, but for now we can leverage the in built sub string function available in most modern languages.

Okay, now can we reduce the sub tree problem to a sub string problem? Yes! If we traverse `T1` and store it's result as a string, then all we have to do is traverse `T2`, store it's result as a string and check if `T2.string` is a sub string of `T1.string`. __But how?__

#### Traversals

Recall, there are 3 ways to traverse a tree. ***In-order***, ***Pre-order*** and ***Post-order***. Let's see how these traversal techniques might help us.

##### In-order

In-order traversal follows the `left-root-right` approach. So for `T1`, the in-order string would be `123456`, for `T2` it would be `1234` and `T3` would be `1234`.

The problem with this is that if the tree is a `Binary Search Tree`, the in-order traversal will always result in the same string, that is in ascending order. Hence, if we used this traversal, we would assert that `T3` is a sub tree of `T1` but by definition it is not!

##### Pre-order

Pre-order traversal follows the `root-left-right` approach. So for T1, we get `532146`, T2 is `3214` and T3 is `3124`. This works! Or does it? 🤔

According to the definition above, there must be a node in `T1` which when cut off from `T1`, the tree rooted at this node is identical to `T2`. So for the algorithm to proceed, it should find a node in `T1` whose value is equal to the value of the root in `T2`. Which means the root needs to be processed before any other node and that's exactly what pre-order traversal does!

```
    1       1
   /         \ 
  2           2
```
For the two examples above, the pre-order traversal will yield the same result, i.e, `12`, but by definition they're not the same.

What if we noted the position of the `NULL` pointer and represented it by `X`? What is the pre-order result now? `12X` and `1X2`.

Using this approach, the pre-order traversals for `T1`, `T2` and `T3` now become `5321X4XX6XX`, `321X4XX` and `31X24XX` respectively. Now we can use our sub string approach and be absolutely sure that we're getting the correct result. 

##### Time Complexity

Let's assume `T1` has `N` elements and `T2` has `M` elements. We visit each element in `T1` one time making it a `O(N)` process and we visit each element in `T2` one time making it a `O(M)` process making the traversal a `O(N + M)` process. The KMP algorithm is linear in terms of the size of the string and the pattern to match as well.

##### Space Complexity

Since we are building and storing strings for the traversal, the space complexity is `O(N + M)` as well.

__But T1 and T2 are ***very large*** trees! Can we afford this extra space? Can we use recursion? Yes!__

##### Recursive Approach

We can solve this problem recursively as well. All we would have to do is start `DFS` on `T1` and find the node that is equivalent to the root of `T2` and then start another recursion to match the two trees starting at that point.

The code to do this as follows:

```swift
func matchTrees(t1: Node?, t2: Node?) -> Bool {
  if t1 == nil && t2 == nil {
    return true
  }
  
  if t1 == nil || t2 == nil {
    return false
  }
  
  if t1?.value != t2?.value {
    return false
  }
  
  return matchTrees(t1: t1?.left, t2: t2?.left) && matchTrees(t1: t1?.right, t2: t2?.right)
}

func findRoot(t1: Node?, t2: Node?) -> Bool {
  if t1 == nil {
    return false
  }
  
  if t1?.value == t2?.value {
    return matchTrees(t1: t1, t2: t2)
  }
  
  return findRoot(t1: t1?.left, t2: t2) || findRoot(t1: t1?.right, t2: t2)
}

func checkSubTree(t1: Node?, t2: Node?) -> Bool {
  if t2 == nil {
    return true
  }
  
  return findRoot(t1: t1, t2: t2)
}
```

##### Time Complexity

With the recursive approach, in the worst case, we may have to visit each and every node of `T1` once to find the root of `T2` in `T1`, which is `O(N)` time. Once the root is found, we traverse `T2` as long as the data in the corresponding pointers match making it `O(N + kM)` where `k` is the number of times the pointers match.

##### Space Complexity

While we don't maintain an additinoal data structure, we definitely do build up a recursive stack frame which is definitely smaller than `O(N+M)`.

Solution is hosted on [Github](https://github.com/mohitathwani/SwiftCodingChallenges/blob/master/checkSubtree/CheckSubTree.playground/Contents.swift).