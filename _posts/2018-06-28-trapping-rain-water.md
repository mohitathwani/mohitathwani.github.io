---
title: 'Leetcode - 42. Trapping Rain Water'
date: 2018-06-23 10:00:00
featured_image: '/images/trapping-rain-water.jpg'
categories: [Algorithms, Array, Recursion, Two POinters, Leetcode]
redirect_from: /leetcode/leetcode-42-trapping-rain-water/
---

There’s one thing common between this problem and the [Container With Most Water](http://mohit.athwani.net/leetcode/leetcode-11-container-with-most-water/) problem and that is the height of the shorter tower dictates the result. In this problem, the amount of water that can accumulate between two towers is limited by the height of the shorter tower.  Consider an array [4, 5, 9, 7 ,1, 0 ,4, 0 ,8 ,6 ,5 ,2 ,3] which is represented as:

|0|1|2|3|4|5|6|7|8|9|10|11|12|
|--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |
|||I|||||||||||
|||I|x|x|x|x|x|I|||||
|||I|I|x|x|x|x|I|||||
|||I|I|x|x|x|x|I|I||||
||I|I|I|x|x|x|x|I|I|I|||
|I|I|I|I|x|x|I|x|I|I|I|||
|I|I|I|I|x|x|I|x|I|I|I|x|I|
|I|I|I|I|x|x|I|x|I|I|I|I|I|
|I|I|I|I|I|x|I|x|I|I|I|I|I|

`x` in the above graph represents where water can accumulate.

At index 3, we have a tower of height 7 and 1 unit of water over it.

At index 4, we have a tower of height 1 and we have 7 units of water accumulating over it.

At index 5, there is no tower (height 0) and there’s 8 units of water accumulating over it.

At index 6, there are 4 units of water accumulating over a tower of height 4.

At index 7, there are 8 units of water over a tower of height 0.

This means that the bounding height in that region is 8.

At index 11, there is 1 unit of water over a tower of height 2. So the bounding height here is 3.

The question now is, how do we find this bounding height? Like I mentioned before, in a container with two different heights on either side, you can only fill it up as much water as the height of the shorter side. Let’s try to apply this principle to the array above.

At index 0, the tallest tower on the left is 4 (itself) and the tallest tower on the right is 9. There is a potential to have 4 units of water at this location, but there is a tower of height 4, hence no water accumulates over this tower.

At index 3, the tallest tower on the left is 9 and the tallest tower on the right is 8. There is a potential to have 8 units of water at this location, but there is a tower of height 7 hence there is just 1 unit of water at this location.

With this logic we come to the equation:

`water(i) = min(leftMax, rightMax) - a[i]`

Finding the `leftMax` and `rightMax` for every index is an expensive operation and makes this algorithm perform in N^2 time.

There are solutions out there that show you how to solve the problem using Dynamic Programming and a Stack but both these approaches use up extra memory. There is a way to solve this problem using no extra memory. Let’s look at it below keeping in mind the same logic that the height of water is dictated by the shorter side.

Let’s have a start pointer at the beginning of the array and an end pointer at the end and let’s also have two variables `leftMax` and `rightMax` initialized to 0.

Let’s compare the heights of the towers at start and end. In this case `a[start]` is greater than `a[end]`. Check if `a[end]` is greater than `rightMax`. It is, so set `rightMax` to `a[end]` and move the end pointer to the left.

Compare `a[0]` with `a[11]`, `a[11]` is smaller. `a[11]` is also smaller than `rightMax`, which means there is a potential for water accumulating over it and that value is `rightMax - a[end]`. How and Why? Let’s do some more iterations and see if we figure it out.

Compare `a[0]` with `a[10]`. Now, `a[0]` is smaller. Is `a[0]` greater than `leftMax`? Yes, so update `leftMax` and advance the `start` pointer. Now, `leftMax` is `4` and `rightMax` is `3`.

Compare `a[1]` with `a[10]`, `a[1]` is less than or equal to `a[10]`. Is `a[1]` greater than `leftMax`? Yes, so `leftMaxis` now `5` and advance the `start` pointer forward.

Still makes no sense.

Compare `a[2]` with `a[10]`, `a[2]` is greater. Is `a[10]` greater than `rightMax`? Yes, update it and decrement the `end` pointer. Every time we are in a situation where we update the max value, there is no possibility of having water over that tower because that tower is the tallest seen so far.

Compare `a[2]` with `a[9]`, perform the same steps as above.

Compare `a[2]` with `a[8]`, perform the same steps as above.

Compare `a[2]` with `a[7]`. `a[7]` is less than `a[2]`. Is `a[7]` greater than `rightMax`? No, so this means water can accumulate at this spot. How much water? `rightMax - a[7]`.

So, if you think about it we only update the pointer that is pointing to a smaller tower. This is because if we have the bigger tower fixed, we move towards it from the other side because at this point we know what the max is on the other side, and since we updated the pointer on that side, it has to be smaller than the fixed tower and are comparison comes down to the same equation we derived before:

`water(i) = min(leftMax, rightMax) - a[i]`

In the above situation where start is 2, end is 7 and rightMax is 8, ignoring all the values in between, we are in a situation that looks like this:

|0|1|2|3|4|5|6|7|8|9|10|11|12|
|--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |
|||I|||||||||||
|||I||||||I|||||
|||I||||||I|||||
|||I||||||I|I||||
||I|I||||||I|I|I|||
|I|I|I||||||I|I|I|||
|I|I|I||||||I|I|I|x|I|
|I|I|I||||||I|I|I|I|I|
|I|I|I|.|.|.|.|end|I|I|I|I|I|

So between towers of heights 9 and 8, we will always have water accumulating up to 8 units. How much water accumulates over a specific tower depends on the height of that specific tower but is limited to 8.

Another observation is that we start filling up water in open spots from a lower level to a higher level. If this makes any sense.

So the code for this looks like:

```swift
while left < right {
  if height[left] <= height[right]  {
    if height[left] >= leftMax {
      leftMax = height[left]
    } else {
      result += leftMax - height[left]
    }
    left += 1
  } else {
    if height[right] >= rightMax {
      rightMax = height[right]
    } else {
      result += rightMax - height[right]
    }
    right -= 1
  }
}
```

This makes it a `O(N)` algorithm in `O(1)` space.

Code hosted on [Github](https://github.com/mohitathwani/SwiftCodingChallenges/blob/master/trappingRainWater/TrappingRainWater.playground/Contents.swift).