---
title: 'Leetcode - 863. All Nodes Distance K In Binary Tree'
date: 2018-07-02 10:00:00
featured_image: '/images/distance-k.jpg'
categories: [Algorithms, Binary Tree, Breadth First Search, Depth First Search]
redirect_from: tree/leetcode-863-all-nodes-distance-k-in-binary-tree/
---

###### Problem Statement
Given the `root` node of a Binary Tree, a `target` node and a distance `k`, return a list of all nodes that are a distance `k` from the `target`.

## Explanation

When I first saw this problem, I panicked! I knew that this some how had to be a `Breadth First Search` problem because `BFS` from any node gives you the shortest path to every other node that is connected to the tree. Since every node is represented as a `TreeNode` with only `left` and `right` pointers, I would need to know the `parent` too for me to do a `BFS` from the `target` node and that was the reason for my panic. In fact, I even started doing a dry run of a [Floyd-Warshall](https://en.wikipedia.org/wiki/Floyd%E2%80%93Warshall_algorithm) algorithm as this algorithm gives the shortest path between all pairs of nodes in a graph in O(N^3) time.

Depth First Search to the rescue! I later realized that I can use DFS to generate a map of a node to it’s parent and at this point, I should stop looking at the tree as a tree but as a graph.

For example, given the tree:

```
       3
    /     \
   5       1
  / \     / \
 6   2   0   8
    / \
   7   4
```

For `target = 5` and `K = 2`, we need to return `[7, 4, 1]`. Once we have performed DFS and generated the map, we should now start looking at this tree as a graph as such:

```
6 - 5 - 3 - 1 - 8
    |       |
7 - 2       0
    |
    4
```

Now, we can perform BFS from the target node and whenever we encounter a node which is `K` distance away, we can append it to our results array. We can even have an optimization to break the BFS loop when we encounter a node at distance greater than `K` as BFS guarantees that we will process all nodes at a distance `d` before going to distance `d + 1`.

The code for this problem is as follows:

```
extension TreeNode: Hashable, CustomStringConvertible {
  public var description: String {
    return "\(val)"
  }
  
  public var hashValue: Int {
    return val
  }
  
  public static func == (lhs: TreeNode, rhs: TreeNode) -> Bool {
    return lhs.val == rhs.val
  }
}

func distanceK(_ root: TreeNode?, _ target: TreeNode?, _ K: Int) -> [Int] {
  guard let root = root, let target = target else {
    return [Int]()
  }
  
  var parentMap = [TreeNode:TreeNode?]()
  var result = [Int]()
  
  func dfs(currentNode: TreeNode?, parentNode: TreeNode?) {
    guard let currentNode = currentNode else {
      return
    }
    
    parentMap[currentNode] = parentNode
    dfs(currentNode: currentNode.left, parentNode: currentNode)
    dfs(currentNode: currentNode.right, parentNode: currentNode)
  }
  
  dfs(currentNode: root, parentNode: nil)
  
  func bfs(startNode: TreeNode) {
    var q = [TreeNode]()
    var seenSet = [TreeNode]()
    
    q.append(startNode)
    seenSet.append(startNode)
    
    var distanceMap = [TreeNode:Int]()
    distanceMap[startNode] = 0
    
    if K == 0 {
      result.append(startNode.val)
      return
    }
    
    while !q.isEmpty {
      let node = q.removeFirst()
      if let left = node.left, !seenSet.contains(left) {
        q.append(left)
        seenSet.append(left)
        
        distanceMap[left] = distanceMap[node]! + 1
        if distanceMap[left]! == K {
          result.append(left.val)
        }
      }
      
      if let right = node.right, !seenSet.contains(right) {
        q.append(right)
        seenSet.append(right)
        
        distanceMap[right] = distanceMap[node]! + 1
        if distanceMap[right]! == K {
          result.append(right.val)
        }
      }
      
      if let optionalParent = parentMap[node], let parent = optionalParent, !seenSet.contains(parent) {
        q.append(parent)
        seenSet.append(parent)
        
        distanceMap[parent] = distanceMap[node]! + 1
        if distanceMap[parent]! == K {
          result.append(parent.val)
        }
      }
    }
  }
  
  bfs(startNode: target)
  return result
}
```

##### Time Complexity

The time complexity of this problem is `O(N)` for DFS and `O(N)` for BFS which gives us a total of `O(N)`.

##### Space Complexity

The space complexity is `O(lg N)` for the DFS recursive stack and `O(N)` for the BFS queue which gives us a total of `O(N)`.

Solution is hosted on [Github](https://github.com/mohitathwani/SwiftCodingChallenges/blob/master/allNodesDistanceKInBinaryTree/AllNodesDistanceKInBinaryTree.playground/Contents.swift).