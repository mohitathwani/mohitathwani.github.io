---
title: 'Leetcode - 64. Minimum Path Sum'
date: 2018-07-11 10:00:00
featured_image: '/images/minimum_path.jpg'
categories: [Algorithms, Array, Dynamic Programming]
redirect_from: dynamic-programming/leetcode-64-minimum-path-sum/
---

## Problem Statement
Given a m x n grid filled with non-negative numbers, find a path from top left to bottom right which minimizes the sum of all numbers along its path.

**Note**: You can only move either down or right at any point in time.

#### Example
```
Input:
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
Output: 7
Explanation: Because the path 1→3→1→1→1 minimizes the sum.
```

## Explanation

There were two approaches that came to my mind when I first read this problem.

1. This is a classic recursion problem and we can use a a modified DFS approach checking at each point if we can move left and down and then recurse from the new positions. The time complexity of this would be exponential of the order of O(2^(m + n)) because at each step we have 2 decisions to make and at we have to make this decision m+n times based on the number of rows and columns in the grid; hence, this approach is out of question.
2. The second approach that came to my mind is to use Dijkstra‘s algorithm since this problem is clearly a single source shortest path algorithm. To run this algorithm, I would have to do a lot of additional work to set up the nodes, edges, a priority queue, etc.. While this could be a viable solution, it just seems like there’s too much work involved.

An unexpected but efficient way to solve this problem is using [Dynamic Programming](https://en.wikipedia.org/wiki/Dynamic_programming). Let’s look at some examples.

Example 1:

A grid containing only one element : `[[3]]`. The result is 3.

Example 2:

A grid containing just one row and multiple columns: [[1, 2, 3]]. The result is 6. How? Starting at `[0,0]` the only option is to move right. At `[0,1]` the only option again is to move right, giving us the equation:

`cost += grid[i][j] + grid[i][j + 1]`

Example 3:

A grid containing just one column and multiple rows: `[[1], [2], [3]]`. The result is 6. How? Starting at `[0,0]` the only option is to move down. At `[1,0]` the only option again is to move down, giving us the equation:

`cost += grid[i][j] + grid[i + 1][j]`

Example 4:

A multiple row, and multiple column grid: `[[1,2,3], [4,5,6]]`. What’s the result? Let’s figure it out. Starting at `[0,0]` we have two options: either move left or move down. But, making a greedy decision at this point may not necessarily lead to an optimal solution. So instead, we flip the problem the other way around and start making greedy decisions from the end point tracing a way back to the starting point.

For the following grid:

```
1	3	4	8
3	2	2	4
5	7	1	9
2	3	2	3
```

3 at index `[3,3]` will always be part of the solution so this can be an ideal starting point for our dynamic programming table:

```
0	0	0	0
0	0	0	0
0	0	0	0
0	0	0	3
```

For `dp[3][2]`, we have to take `grid[3][2]` and take the minimum from `dp[3][3]` and dp`[4][3]`, since `[4,3]` is out of bounds, we get `dp[3][2] = 5`. Following this rule, we get:

```
0	0	0	0
0	0	0	0
0	0	0	0
10	8	5	3
```

For `dp[2][3]`, we have to take `grid[2][3]` and the minimum of `dp[2][4]` and `dp[3][3]`, we get `dp[2][3] = 12`.

For `dp[2][2]`, the equation now becomes clear:

`dp[i][j] = grid[i][j] + min(dp[i + 1][j], dp[i][j + 1])`

Finally, the table is:

```
14	13	12	24
13	10	8	16
15	13	6	12
10	8	5	3
```

The code will look like:

```swift
func minPathSum(_ grid: [[Int]]) -> Int {
  var result = 0
  
  guard !grid.isEmpty else {
    return result
  }
  
  let numRows = grid.count
  let numCols = grid[0].count
  
  var dp = [[Int]](repeating: [Int](repeating: 0, count: numCols), count: numRows)
  
  for i in stride(from: numRows - 1, through: 0, by: -1) {
    for j in stride(from: numCols - 1, through: 0, by: -1) {
      
      if i + 1 < numRows && j + 1 < numCols {
        dp[i][j] = grid[i][j] + min(dp[i + 1][j], dp[i][j + 1])
      }
      else if i + 1 < numRows {
        dp[i][j] = grid[i][j] + dp[i + 1][j]
      }
      else if j + 1 < numCols {
        dp[i][j] = grid[i][j] + dp[i][j + 1]
      }
      else {
        dp[i][j] = grid[i][j]
      }
    }
  }
  
  result = dp[0][0]
  
  return result
}
```

The space and time complexity is `O(mn)` in terms of the number of rows and columns.

The problem is taken from [Leetcode](https://leetcode.com/problems/minimum-path-sum/).

Solution is hosted on [Github](https://github.com/mohitathwani/SwiftCodingChallenges/blob/master/minimumPathSum/MinimumPathSum.playground/Contents.swift).