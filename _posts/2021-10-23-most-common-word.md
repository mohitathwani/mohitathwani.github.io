---
title: 'Leetcode - 819. Most Common Word'
date: 2021-10-23 10:00:00
featured_image: '/images/most-common-word.jpg'
categories: [Algorithms, Arrays, Strings]
---

Wow it's been a while sonce I wrote anything here. I have been doing a lot of functional reactive programming at work and have been wondering how I can apply these new concepts to Leetcode problems. Here is my attempt at finding the most common word from a paragraph that is **NOT** in the list of banned words.

### Attempt 1:
```swift
func mostCommonWord(_ paragraph: String, _ banned: [String]) -> String {
        let dict =
            paragraph
            .split { !($0.isLetter) }
            .filter { !banned.contains($0.lowercased()) }
            .reduce(into: [:]) { dict, substring in
                dict[substring.lowercased(), default: 0] += 1
            }
        let maxTuple = dict.max { $0.value < $1.value }

      return maxTuple?.key ?? ""
    }
```

This attempt ran in 12ms but as you can see we iterate over the given data multiple times. One time when we split, next when we filter, third time when we reduce and the fourth time to get the max key in the dictionary.

Luckily Swift gives us an optimization, which is the `lazy` keyword that that transforms the `Sequence` in to a `LazySequence` and can be used before operations like `filter`. Given this information, here is attempt number two which ran in 8ms.

### Attempt 2:
```swift
func mostCommonWord(_ paragraph: String, _ banned: [String]) -> String {
        let dict =
            paragraph
            .split { !($0.isLetter) }
            .lazy
            .filter { !banned.contains($0.lowercased()) }
            .reduce(into: [:]) { dict, substring in
                dict[substring.lowercased(), default: 0] += 1
            }
        let maxTuple = dict.max { $0.value < $1.value }

      return maxTuple?.key ?? ""
    }
```

Another potential optimization that can be added is to convert the banned list in to a `Set` so that the call to `.contains()` in the `filter` operation runs in `O(1)` time.

The time time complexity of both attempts remoains `O(NM)` where `N` is the size of the `paragraph` and `M` is the size of the banned words list. If banned words is first converted to a `Set`, we can bring the time complexity to `O(M+N)`. The spae complexity is `O(N)` as we are building a dictionary to store the frequency of words. If we transform the banned words to a `Set`, the space complexity would be `O(N+M)`.

The problem is taken from [Leetcode](https://leetcode.com/problems/most-common-word/).

Solution is hosted on [Github](https://github.com/mohitathwani/SwiftCodingChallenges/blob/master/missingWords/MissingWords.playground/Contents.swift).
