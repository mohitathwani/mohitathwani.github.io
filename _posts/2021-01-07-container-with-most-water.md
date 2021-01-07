---
title: 'Leetcode - 11. Container With Most Water'
date: 2021-01-07 10:00:00
featured_image: '/images/container-with-most-water.jpg'
categories: [Algorithms, Array]
---

Think of the numbers in the array representing a height. Now, you can create an imaginary box by selecting any two numbers in the array. The max height of water that this imaginary box can hold is the minimuj of the two numbers. So what is the area of the water that can be held? 

`area = min(height1, height2) * |right - left|`.

Initialize two pointers, `left = 0` and `right = arr.size() - 1` and `result = 0` and perform the following for as long as `left < right`:

1. Caluclate area between `left` and `right`
2. `result = max(result, area)`
3. Because we want to find the biggest possible container, we must elimnate the shorter side. So, either `left++` or `right--` depending on which height is shorter.

The time complexity is `O(N)` where `N` is the total number elements in the array.

The space complexity is `O(1)` since no additional space was used.

The problem is taken from [Leetcode](https://leetcode.com/problems/container-with-most-water/).

Solution is hosted on [Github](https://github.com/mohitathwani/leetcode_submissions/tree/main/problems/container_with_most_water).
