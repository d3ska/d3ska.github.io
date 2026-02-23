---
title: "Complexity Analysis"
categories:
  - Blog
tags:
  - Complexity Analysis
  - Big O
---


It's a process in which we determine how efficient an algorithm is. There are multiple ways to solve the same issue but
the complexity analysis will likely differ, some solutions are much better than others. It's effectively used to
determine how "good" an algorithm is and whether it's "better" than another one.

There are usually two things that we will measure: time complexity and space complexity of an algorithm.

* **Time Complexity**

A measure of how fast an algorithm runs, time complexity is a central concept in the field of algorithms. It's expressed
using Big O notation.


* **Space Complexity**

A measure of how much auxiliary memory an algorithm takes up, space complexity is a central concept in the field of
algorithms. It's expressed using Big O notation.

It is worth noting that "space complexity" can be defined in two ways. Some textbooks define it as the **total** memory used, including the space taken by the input itself. In practice, however, we most often care about **auxiliary space** — the extra memory the algorithm allocates on top of the input. When you see space complexity discussed in interview or day-to-day engineering contexts, it almost always refers to auxiliary space.

<br>

### A Concrete Example

Consider a simple problem: finding a target value in an unsorted array. A straightforward approach is to scan every element from start to finish.

```
function find(array, target):
    for each element in array:
        if element == target:
            return true
    return false
```

In the worst case, we check every element before finding the target (or not finding it at all), so the **time complexity is O(n)**, where n is the length of the array. The algorithm only uses a handful of variables regardless of the input size, so the **space complexity is O(1)**.

### Time / Space Trade-offs

In many real-world scenarios, we can speed up an algorithm by using more memory, or reduce memory usage at the cost of running slower. This is known as the **time-space trade-off**. For example, we could check for duplicate values in an array by comparing every pair of elements — O(n²) time and O(1) space — or we could store seen values in a hash set and detect duplicates in a single pass — O(n) time but O(n) space. Choosing between these approaches depends on the constraints of the problem: how large the input is, how much memory is available, and how fast the solution needs to be. Recognizing these trade-offs is a core skill in algorithm design.

<br>


What is worth adding is that different data structures are going to have different time complexity and space complexity
for functions and operations that they support.
The key thing is not only to figure out what data structure best allows you
to solve the problem but also what data structure allows you to do so with the best time/space complexity.
