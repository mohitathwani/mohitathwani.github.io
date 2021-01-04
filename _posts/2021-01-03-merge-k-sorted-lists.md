---
title: 'Leetcode - 23. Merge k Sorted Lists'
date: 2021-01-03 10:00:00
featured_image: '/images/k-linked-lists.jpg'
categories: [Algorithms, Linked Lists, Priority Queue]
---

My confusion with these kinds of problems is whether or not a whole new Linked List must be created.

The first time I solved this problem, I used a modified approach to the merge 2 sorted lists problem.

The second time I solved this problem, priority queue came to my mind because the lists are already sorted.

What mistakes did I make?
1. I inserted all the nodes in all the lists in to the priority queue. This is not needed. The size of the priority queue should be at most `k` for an optimal solution.
2. I did not check the input for `NULL` before inserting nodes in to the priority queue.

The time complexity is `O(N log(k))` where `N` is the total number of nodes and `k` is the number of Linked Lists. Inserting and removing from the priority queue is a `log(k)` operation which is done `N` times.

The space complexity is `O(N + k)` or `O(N)` if `k` is much smaller than `N`.

The problem is taken from [Leetcode](https://leetcode.com/problems/merge-k-sorted-lists/).

Solution is hosted on [Github](https://github.com/mohitathwani/leetcode_submissions/tree/main/problems/merge_k_sorted_lists).
