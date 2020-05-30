---
title: 'Leetcode - 33. Search In Rotated Sorted Array'
date: 2018-07-01 10:00:00
featured_image: '/images/rotated-sorted.jpg'
categories: [Algorithms, Array, Binary Search]
redirect_from: leetcode/leetcode-33-search-in-rotated-sorted-array/
---

This is straight up [Binary Search](https://en.wikipedia.org/wiki/Binary_search_algorithm) problem with some modifications. When we find the mid point of the given array, one of two things will hold true:


1. The array from start to mid is sorted.
2. The array from mid to end is sorted.

Once we know which is the sorted half, all we have to do is check if the search target lies within the sorted half. If yes, do regular binary search in that half of the problem. If no, then we have to update the 3 pointers depending on whether the lower half is sorted or upper half is sorted and than recurse again.

Let’s look at an example. In the array `nums = [4, 5, 6, 7, 8, 0, 1, 2]`, and `target = 6`, `start = 0`, `end = 7` and `mid = 3`. `nums[start...mid]` is sorted, i.e. `4, 5, 6, 7` are in sorted order and `8, 0, 1, 2` are not sorted.

We check if `6` lies between `nums[start]` and `nums[mid]`. Since it does, we do a binary search from indices `start` to `mid - 1`.

Let’s set the target as `1`. `1` appears in the unsorted half of the array. We will now recursively find the sorted and unsorted halves of the upper half of the array. Now, `start = 4`, `end = 7` and `mid = 5`. `nums[6...7]` is sorted in this case so we perform binary search in this section of the array.

The code for this is pretty straight forward:

```
func search(_ nums: [Int], _ target: Int) -> Int {
  guard nums.count > 0 else {
    return -1
  }
  
  var start = 0
  var end = nums.count - 1
  
  func search(start: Int, end: Int) -> Int {
    
    if end < start {
      return -1
    }
    
    let mid = start + (end - start) / 2
    
    print(start, mid, end)
    
    if nums[mid] == target {
      return mid
    }
    
    
    //lower half is sorted
    if nums[start] <= nums[mid] {
      if target >= nums[start] && target <= nums[mid] {
        //search in lower half
        return search(start: start, end: mid - 1)
      } else {
        //search in upper half
        return search(start: mid + 1, end: end)
      }
    }
    
    //upper half is sorted
    if nums[mid] < nums[end] {
      if target >= nums[mid] && target <= nums[end] {
        //search in upper half
        return search(start: mid + 1, end: end)
      } else {
        //search in lower half
        return search(start: start, end: mid - 1)
      }
      
    }
    
    return -1
  }
  
  return search(start: 0, end: nums.count - 1)
}
```



##### Time and Space Complexity

Since this is straight up binary search, the time complexity will be O(lg N) and since this is recursive, I would say that this is O(lg N) space as well. If we refactor the code and pass the value of mid to the nested search(start:end:) function instead of calculating it in the function, we can make the recursive call tail recursive and in that way there will be no extra space if the compiler can make those optimizations.

Problem taken from [Leetcode](https://leetcode.com/problems/search-in-rotated-sorted-array/description/).

Solution is hosted on [Github](https://github.com/mohitathwani/SwiftCodingChallenges/blob/master/allNodesDistanceKInBinaryTree/AllNodesDistanceKInBinaryTree.playground/Contents.swift).