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

```










##### Time Complexity

With the recursive approach, in the worst case, we may have to visit each and every node of `T1` once to find the root of `T2` in `T1`, which is `O(N)` time. Once the root is found, we traverse `T2` as long as the data in the corresponding pointers match making it `O(N + kM)` where `k` is the number of times the pointers match.

##### Space Complexity

While we don't maintain an additinoal data structure, we definitely do build up a recursive stack frame which is definitely smaller than `O(N+M)`.

Solution is hosted on [Github](https://github.com/mohitathwani/SwiftCodingChallenges/blob/master/checkSubtree/CheckSubTree.playground/Contents.swift).