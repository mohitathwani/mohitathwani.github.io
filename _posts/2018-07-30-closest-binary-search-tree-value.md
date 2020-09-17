---
title: 'Leetcode - 270. Closest Binary Search Tree Value'
date: 2018-07-30 10:00:00
featured_image: '/images/closest-bst-value.jpg'
categories: [Algorithms, Binary Search Tree, Leetcode]
---

If the type of the target and the type of the value in the tree nodes were the same, and the problem was to find the node in the tree that is just smaller than the target, the problem becomes fairly simple. For example, given `target = 17` and the graph:

```
    20
   / \
  9   25
 / \
5   12
   /  \
  11  14
```

the value we will return is 14. The problem can be rethought to be as finding the largest value in the tree that is just smaller than the target. Since we want to find the largest such node, the key thing to understand here is that the answer can be found only while traversing right.

To find 17, starting at 20, 17 is less than 20, we traverse left, 17 is greater than 9, what if 9 was a leaf node? 9 would be the solution, but it is not, so we traverse right. 17 is greater than 12, again if 12 was a leaf node, 12 would be the answer, but it is not so we traverse right again. 14 is less than 17. We would want to go right to get as close as possible to 17 but since 14 is a leaf node, we select this as the answer.

Suppose target was 6. 6 is less than 20, go left. 6 is less than 9 go left. 6 is greater than 5, so that means the solution lies on the right of 5. But 5 is a leaf node so return 5.

But, this problem is slightly different. The type of the target in this problem is Double whereas the type of the value of the tree node is Int and our task is to find the closest value in the tree. For example, 3.7 is closer to 4 than it is to 3.

Given target = 3.7 for the following graph,

```
    4
   / \
  2   5
 / \
1   3
```

the correct answer would be 4, but if we applied the logic explained above, we would be return 3 as the answer, which is not the case. So this means we have to keep track of probable `floor` and `ceil` values to decide the result.

3.7 is less than 4, we set the `ceil` value to 4 and move left. 3.7 is greater than 2, we set the `floor` value to 2 and move right. 3.7 is greater than 3, we update the `floor` value and now there is nowhere to go.

Now that we have `floor = 3` and `ceil = 4`, we can decide which is closer to 3.7.

Given target = 2.2, 2.2 is less than 4, we set the `ceil` to 4 and move right. 2.2 is greater than 2, set the `floor` value to 2 and go right. 2.2 is less than 3, set the `ceil` to 3 and there’s nowhere to go now which leaves us with floor = 2 and ceil = 3.

Great, so now we have a basic idea about how this algorithm works. Let’s look at the code for this:

```swift
func closestValue(_ root: TreeNode?, _ target: Double) -> Int {
  guard let root = root else {
    return 0
  }
  
  var floor: Int?
  var ceil: Int?
  var result = 0
  
  func find(root: TreeNode?) {
    guard root != nil else {
      return
    }
    
    if target == Double(root!.val) {
      result = root!.val
      return
    }
      
    else if target < Double(root!.val) {
      ceil = root!.val
      find(root: root?.left)
    } else {
      floor = root!.val
      find(root: root?.right)
    }
  }
  
  find(root: root)
  
  if result != 0 {
    return result
  }
  
  if floor != nil && ceil != nil {
    let diff1 = abs(target - Double(floor!))
    let diff2 = abs(target - Double(ceil!))
    
    if diff1 <= diff2 {
      result = floor!
    } else {
      result = ceil!
    }
  }
  
  else if floor == nil && ceil != nil {
    result = ceil!
  } else {
    result = floor!
  }
  
  return result
}
```

This code is littered with `if` statements everywhere and it even fails 2 tests on [Leetcode](https://leetcode.com/problems/closest-binary-search-tree-value/description/), which got me thinking about a better solution.

These are the steps I take to refactor the code:

1. Set the result to be the root’s value.
2. Use a wile loop until the root is not null
3. In the while loop, if the absolute difference between target and result is greater than the absolute difference between the target and the current root, then the result is probably the current root.
4. Depending on the value of the target, go left or go right.

And now, the code becomes:

```swift
func closestValue(_ root: TreeNode?, _ target: Double) -> Int {
  var root = root
  
  var result = root?.val ?? 0
  
  while root != nil {
    if Double(root!.val) == target {
      return root!.val
    }
    
    if abs(target - Double(result)) > abs(target - Double(root!.val)) {
      result = root!.val
    }
    
    if target < Double(root!.val) {
      root = root?.left
    } else {
      root = root?.right
    }
    
  }
  
  return result
}
```

Time complexity of this solution is `O(lg N)` if the BST is balanced, `O(N)` otherwise as we only need to traverse the height of the tree.

No addition space is needed.

The code is hosted on [Github](https://github.com/mohitathwani/SwiftCodingChallenges/blob/master/closestBinarySearchTreeValue/ClosestBinarySearchTreeValue.playground/Contents.swift).