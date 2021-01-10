---
title: 'Leetcode - 10. Regular Expression Matching'
date: 2021-01-10 10:00:00
featured_image: '/images/regex-matching.jpg'
categories: [Algorithms, Recursion]
---

I've seen this problem more than a year ago and back then I kinda memorized the solution.

This time, I tried to solve the problem myself and got nearly 2/3rd of the pseudo code.

There's 3 cases to this problem:
* 1\. If `s[sI] == p[pI] || p[pI] == '.'`, simply recurse with `sI + 1` and `pI + 1`
* 2\. If `pI + 1 == '*'`, then: 
    * 2.1\. Try to match `s` with `p[pI + 2 ...]`
    * 2.2\. If above fails, check the condition in *1* is satisfied, if yes, recurse with `sI + 1` and `pI`. This solves cases like `s = aa` and `p = a*`

I was able to get *1* and *2.1* by myself.

**Tip:** Solve case *2* before *1* in code.

The time complexity is `O(N)` where `N` is the longer of the 2 strings.

The space complexity is `O(N)` where `N` is the longer of the 2 strings..

The problem is taken from [Leetcode](https://leetcode.com/problems/regular-expression-matching/).

Solution is hosted on [Github](https://github.com/mohitathwani/leetcode_submissions/tree/main/problems/regular_expression_matching).
