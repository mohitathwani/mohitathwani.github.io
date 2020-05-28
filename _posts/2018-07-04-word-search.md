---
title: 'Leetcode - 79. Word Search'
date: 2018-07-04 10:00:00
featured_image: '/images/word-search.jpg'
categories: [Algorithms, Array, Backtracking, DFS]
redirect_from: backtracking/leetcode-79-word-search/
---

###### Problem Statement
Given a 2D board and a word, find if the word exists in the grid.

The word can be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once.

#### Example
```
board =
[
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]

Given word = "ABCCED", return true.
Given word = "SEE", return true.
Given word = "ABCB", return false.
```

## Explanation

In this problem, we are given a two dimensional board of characters and we have to check if the word can be found in the board by checking adjacent cells only.

A basic strategy to solve this problem would be to start walking down the board until the first character is found. Once the first character is found, look at the character one row above, below, one column to the left and only column to the right to see if the second character in the string can be matched. If the second character is found, we look around that position and we go on and on if the characters are found. Before we go on exploring surrounding cells, we need to make sure that we have not visited it before and if we have, we just skip it. We can maintain a `[[Bool]]` to help us keep track of visited and unvisited cells.

This brings us to implementing a recursive strategy and if you think about it, it is pretty similar to `Depth First Search`.

There are some base cases that need to be handled like if the `numRows` and `numCols` are zero and if the search term is empty. Perhaps a base case that I totally missed and helps us completely avoid going into the recursion is checking if the size of the search term is greater than the product of the `numRows` and `numCols`, because if it is then there is no sense in proceeding.

The code for this as shown:

```swift
func dfs(i: Int, j: Int, index: Int, chars: [Character], board:[[Character]], map:inout [[Bool]]) -> Bool {
  
  let topRow = i - 1
  let bottomRow = i + 1
  let leftColumn = j - 1
  let rightColumn = j + 1
  
  let numRows = board.count
  let numCols = board[0].count
  
  let isLastChar = index == chars.count - 1
  
  if isLastChar {
    return board[i][j] == chars[index]
  }
  
  if topRow >= 0 && !map[topRow][j] && !isLastChar && board[topRow][j] == chars[index + 1] {
    map[topRow][j] = true
    let result = dfs(i: topRow, j: j, index: index + 1, chars: chars, board: board, map: &map)
    if result == true {
      return true
    }
    map[topRow][j] = false
  }
  
  if bottomRow < numRows && !map[bottomRow][j] && !isLastChar && board[bottomRow][j] == chars[index + 1] {
    map[bottomRow][j] = true
    let result = dfs(i: bottomRow, j: j, index: index + 1, chars: chars, board: board, map: &map)
    if result == true {
      return true
    }
    map[bottomRow][j] = false
  }
  
  if leftColumn >= 0 && !map[i][leftColumn] && !isLastChar && board[i][leftColumn] == chars[index + 1] {
    map[i][leftColumn] = true
    let result = dfs(i: i, j: leftColumn, index: index + 1, chars: chars, board: board, map: &map)
    if result == true {
      return true
    }
    map[i][leftColumn] = false
  }
  
  if rightColumn < numCols && !map[i][rightColumn] && !isLastChar && board[i][rightColumn] == chars[index + 1] {
    map[i][rightColumn] = true
    let result = dfs(i: i, j: rightColumn, index: index + 1, chars: chars, board: board, map: &map)
    if result == true {
      return true
    }
    map[i][rightColumn] = false
  }
  
  return false
}

func exist(_ board: [[Character]], _ word: String) -> Bool {
  
  let numRows = board.count
  let numCols = numRows > 0 ? board[0].count : 0
  let chars = Array(word)
  
  if numRows == 0 || numCols == 0 || word.isEmpty{
    return false
  }
  
  if word.count >  numRows * numCols {
    return false
  }
  
  var map = board.map { (row) -> [Bool] in
    row.map({ (c) -> Bool in
      return false
    })
  }
  
  for i in stride(from: 0, to: numRows, by: 1) {
    for j in stride(from: 0, to: numCols, by: 1) {
      if board[i][j] == chars[0] {
        map[i][j] = true
        let result = dfs(i: i, j: j, index: 0, chars: chars, board: board, map: &map)
        if result == true {
          return true
        }
        map[i][j] = false
      }
    }
  }
  return false
}
```

##### Time Complexity

The time complexity is `O(N)` where `N` is the length of the search term as we only have to enter recursion if all characters until `i-1` in the search term have been matched. There could be a situation in which the first character is never found while iterating through the board and in that case the time complexity is `O(RC)` in terms of the rows and columns.

##### Space Complexity

The space complexity is `O(RC)` in terms of the rows and columns in the board as we maintain an additional two dimensional array to maintain the visited states.The space complexity is O(RC) in terms of the rows and columns in the board as we maintain an additional two dimensional array to maintain the visited states.

The problem is taken from [Leetcode](https://leetcode.com/problems/word-search/description/).

Solution is hosted on [Github](https://github.com/mohitathwani/SwiftCodingChallenges/blob/master/wordSearch/WordSearch.playground/Contents.swift).