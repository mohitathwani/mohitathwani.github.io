One important thing to note in this problem is that a path does not necessarily start at the root nor does it have to end at a leaf.

A subpath along a particular path from the root to the leaf does qualify to be a solution if the sum of weights along
this path is equal to `N`.

Brute Force Solution

In a brute force approach, we must consider each node to be a possible starting node for this path and work ourselves down from there.
For example, in the given tree

```
N = 4
        10
       /  \
    -8     2
   /   \  
  2     3
 / \   / \
2   2 1   5
```

The evaluated paths are:

```
10 -> -8 -> 2 -> 2 ✔️
10 -> -8 -> 2 -> 2 ✔️
10 -> -8 -> 3 -> 1 ✔️
10 -> -8 -> 3 -> 5 ✔️
-8 -> 2 -> 2 ✔️
-8 -> 2 -> 2 ✔️
-8 -> 3 -> 1 ✔️
-8 -> 3 -> 5 
2 -> 2 ✔️
2 -> 2 ✔️
2
2
3 -> 1 ✔️
3 -> 5
1
5
10 -> 2
2
```
