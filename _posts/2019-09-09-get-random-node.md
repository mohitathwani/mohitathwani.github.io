---
title: 'Get Random Node from Binary Search Tree'
date: 2019-09-09 10:00:00
featured_image: '/images/get-random-node.jpg'
categories: [Algorithms, Binary Search Tree]
---

## Problem Statement
The goal of this challenge is to build your own BinarySearchTree class which in addition to supporting *insert*, *find* and *delete* also supports a special operation *getRandomNode()* which returns a random node from the binary search tree with special consideration that each node in the binary search tree is **equally likely** to be selected. 

## Explanation

The first thing that comes to my mind is that at each node, we have three options. 

1. Pick the node
2. Go left
3. Go right

If you pick to go either left or right above, you recursively have the same 3 options again. Seems like a viable solution right? **No**. 

When you are at the root of the tree, the probably of selecting the root is `1/3`, the probability of going left is `1/3` and the probability of going right is `1/3` as well. What does this tell you? The root node has the highest probability. Why? The probability of going left and right is `1/3`. Which means the sum of all probabilities on the left will add up to `1/3` and so will the sum of all probabilities on the right. Thus all nodes are **not** equally likely to be chosen as the root has the highest probability of `1/3`. 

**Can we do better?**

What if we maintained a list of nodes as part of our tree class? We can certainly do this since we are the tasked with designing this class. We should update this list every time a node is inserted or deleted. While inserting an entry in to a list may be easy, deleting may not be so. We could use an ordered set to maintain this list making insert/delete/lookup a `O(1)` operation. 

When we want a random node, we just look up the size of the list, generate a random number between `0` and `list.size()` and return the element. The `[]` operator may not be overloaded for a Set so make sure you look in to the specifics of the language you are working with. 

This approach does guarantee that each node is equally likely to be picked but the problem with this approach is that we now have to maintain additional memory just for storing the nodes. 

**Can we do better? Yes!**

By definition, the probability of selecting a node should be `1/N` where `N` is the total number of nodes in the tree. So if the probability of selecting the root is `1/N` , the probability of selecting any node from the left will be `LeftCount * 1/N`. Similarly `RightCount * 1/N` for the right hand side.

And therefore, the below equation satisfies our definition and the law of probability. 

![](https://latex.codecogs.com/svg.latex?%5Cfrac%7B1%7D%7BN%7D%20&plus;%20%28LeftCount%20*%20%5Cfrac%7B1%7D%7BN%7D%29%20&plus;%20%28RightCount%20*%20%5Cfrac%7B1%7D%7BN%7D%29%20%3D%201)

And,

![](https://latex.codecogs.com/svg.latex?1%20&plus;%20LeftCount%20&plus;%20RightCount%20%3D%20N)

Let's look at the code for this approach.

```swift
class BinarySearchTree {
  private var root: Node?
  
  init(rootValue value: Int) {
    root = Node(value: value)
  }
  
  public func insert(value: Int) {
    guard let tempRoot = self.root else {
      self.root = Node(value: value)
      return
    }
    
    func recursiveInsert(root: Node?) {
      guard let root = root else {
        return
      }
      
      if value <= root.value && root.left == nil {
        root.left = Node(value: value)
        root.left?.parent = root
      }
      else if value > root.value && root.right == nil {
        root.right = Node(value: value)
        root.right?.parent = root
      }
      else if value <= root.value {
        recursiveInsert(root: root.left)
      }
      else {
        recursiveInsert(root: root.right)
      }
      
      if let left = root.left {
        root.leftCount = left.leftCount + left.rightCount + 1
      }
      
      if let right = root.right {
        root.rightCount = right.leftCount + right.rightCount + 1
      }
    }
    
    recursiveInsert(root: tempRoot)
  }
  
  public func print() {
    func dfs(root: Node?) {
      guard let root = root else {
        return
      }
      
      dfs(root: root.left)
      Swift.print(root)
      dfs(root: root.right)
    }
    
    dfs(root: root)
  }
  
  public func find(value: Int) -> Bool {
    return findNode(value: value).toBool
  }
  
  public func delete(value: Int) {
    guard let nodeToDelete = findNode(value: value) else {
      return
    }
    
    if nodeToDelete.left == nil {
      transplant(node: nodeToDelete.right, over: nodeToDelete)
    }
    else if (nodeToDelete.right == nil) {
      transplant(node: nodeToDelete.left, over: nodeToDelete)
    }
    else {
      guard let successorNode = treeMinimum(node: nodeToDelete.right!) else {
        fatalError()
      }
      
      if let successorParent = successorNode.parent, successorParent !== nodeToDelete {
        transplant(node: successorNode.right, over: successorNode)
        successorNode.right = nodeToDelete.right
        successorNode.right?.parent = successorNode
      }
      transplant(node: successorNode, over: nodeToDelete)
      successorNode.left = nodeToDelete.left
      successorNode.left?.parent = successorNode
    }
    
    updateChildrenCount(for: root)
  }
  
  public func getRandomNode() -> Int? {
    guard let root = root else {
      return nil
    }
    
    return getRandomNode(from: root)
  }
  
  private func getRandomNode(from root: Node?) -> Int? {
    guard let root = root else {
      return nil
    }
    
    let totalCount = root.leftCount + root.rightCount
    guard totalCount != 0 else {
      return root.value
    }
    
    let random = Int.random(in: 0..<totalCount)
    if random < root.leftCount {
      return getRandomNode(from: root.left)
    }
    else if random == root.leftCount {
      return root.value
    }
    else {
      return getRandomNode(from: root.right)
    }
  }
  
  private func findNode(value: Int) -> Node? {
    var tempNode = root
    
    while tempNode != nil {
      if tempNode!.value == value  {
        return tempNode
      }
      
      if value < tempNode!.value {
        tempNode = tempNode!.left
      } else {
        tempNode = tempNode!.right
      }
    }
    return nil
  }
  
  private func treeMinimum(node: Node) -> Node? {
    var temp = node
    while temp.left != nil {
      temp = temp.left!
    }
    return temp
  }
  
  private func transplant(node: Node?, over nodeToDelete: Node) {
    if nodeToDelete.parent == nil {
      self.root = node
    }
    else if nodeToDelete === nodeToDelete.parent?.left {
      nodeToDelete.parent?.left = node
    }
    else {
      nodeToDelete.parent?.right = node
    }
    
    if node != nil {
      node?.parent = nodeToDelete.parent
    }
  }
  
  private func updateChildrenCount(for root: Node?) {
    guard let root = root else {
      return
    }
    
    if root.left == nil && root.right == nil {
      return
    }
    
    updateChildrenCount(for: root.left)
    updateChildrenCount(for: root.right)
    
    if let left = root.left {
      root.leftCount = left.leftCount + left.rightCount + 1
    }
    
    if let right = root.right {
      root.rightCount = right.leftCount + right.rightCount + 1
    }
  }
}

extension BinarySearchTree {
  class Node: CustomStringConvertible {
    public private(set) var value: Int
    var left: Node?
    var right: Node?
    weak var parent:Node?
    
    var leftCount = 0
    var rightCount = 0
    
    var description: String {
      return "Value: \(value); leftCount: \(leftCount); rightCount: \(rightCount)"
    }
    
    init(value: Int) {
      self.value = value
    }
    
    deinit {
      Swift.print("Deleting: ", self)
    }
  }
}

extension Optional where Wrapped: BinarySearchTree.Node {
  var toBool: Bool {
    guard let _ = self else {
      return false
    }
    return true
  }
}
```

##### Time Complexity

The time complexity for this algorithm is pretty straight forward. Inserting takes `O(h)` time where `h` is the height of the tree. Finding a node is `O(h)` as well, and so is deleting. Updating the node counts is `O(N)` time is all nodes have to be visited and since deleting a node triggers a node count update, delete in this case is `O(N)`.

##### Space Complexity

While we don't maintain an additinoal data structure, we definitely do build up a recursive stack frame which is at most `O(h)`.

Solution is hosted on [Github](https://github.com/mohitathwani/SwiftCodingChallenges/blob/master/getRandomNode/GetRandomNode.playground/Contents.swift).
