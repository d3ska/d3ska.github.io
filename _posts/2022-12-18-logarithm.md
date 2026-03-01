---
title: "Logarithms in Algorithm Complexity"
categories:
  - Data Structures
tags:
  - Data Structures
  - Complexity Analysis
  - Algorithms
  - Mathematics
---

The word "logarithm" has a way of making people nervous. It sounds like something you failed a test on in high school. But once you strip away the notation, it is just answering one question: **how many times can I divide this number in half before I get to 1?**

Throughout this post I will write `log(n)` to mean log base 2, which is the standard convention in computer science. When the base matters (like base 10 or base 3), I will write it out explicitly.

### The Definition

A **logarithm** is the inverse of exponentiation. The formal definition:

```
log_x(n) = y    if and only if    x^y = n
```

Since we default to base 2 in computer science:

```
log(n) = y    if and only if    2^y = n
```

In plain language: the log of N is the power to which you need to raise 2 to get N.

A few examples:

```
log(1)   = 0     because  2^0  = 1
log(4)   = 2     because  2^2  = 4
log(256) = 8     because  2^8  = 256
```

The reason base 2 is the natural choice comes from the binary nature of computing: every fundamental decision a computer makes is a binary choice (yes or no, 0 or 1, left or right). Algorithms like binary search split the input in half at each step, and that halving maps directly to powers of 2.

### Why Logarithmic Growth Is So Powerful

Notice what happens when N doubles: the log of N increases by just one.

```
log(16)  = 4     (2^3 * 2 = 2^4)
log(32)  = 5     (2^4 * 2 = 2^5)
```

If we tie this back to complexity analysis, an algorithm with a time complexity of O(log n) is incredibly efficient. When the input doubles, the number of elementary operations increases only by one.

To put real numbers on it: a linear time algorithm with an input of size 1,000 takes roughly 1,000 operations to complete. A logarithmic time algorithm with the same input takes roughly 10 operations, since 2<sup>10</sup> = 1,024.

### Why the Base Does Not Matter in Big O

It is worth noting that in Big O analysis, the base of the logarithm does not matter. This is because logarithms of different bases differ only by a constant factor. The **change of base formula** makes this explicit:

```
log_a(n) = log_b(n) / log_b(a)
```

For example, to convert from base 2 to base 3:

```
log_3(n) = log_2(n) / log_2(3)
```

Here, `1 / log_2(3)` is approximately 0.63, just a constant. Since Big O drops constant factors, O(log<sub>2</sub>(n)), O(log<sub>3</sub>(n)), and O(log<sub>10</sub>(n)) are all equivalent and written simply as O(log n).

### Binary Search: O(log n) in Action

Binary search is the classic example of a logarithmic algorithm. It works on a **sorted** array and repeatedly halves the search space:

```java
public static int binarySearch(int[] sortedArray, int target) {
    int left = 0;
    int right = sortedArray.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (sortedArray[mid] == target) {
            return mid;
        } else if (sortedArray[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return -1;
}
```

At each step, we compare the target to the middle element and discard half of the remaining elements. If the array has 1,024 elements, after one comparison we are down to 512, then 256, then 128, and so on. After at most 10 comparisons (since log(1024) = 10) we have either found the target or determined it is not present. Doubling the array to 2,048 elements only adds a single extra comparison. That is the power of O(log n).

### Other O(log n) Examples

Binary search on an array is not the only place where logarithmic complexity appears. The same "halving the problem" idea shows up in several fundamental data structures and algorithms.

#### Balanced Binary Search Trees

These work on exactly the same principle as binary search. At each node you compare your target to the current value and go either left or right, eliminating roughly half of the remaining candidates. A balanced tree holding 1,000 nodes has a height of about 10 (since log(1000) is roughly 10), so finding any element takes at most 10 comparisons, regardless of which element you are looking for.

#### Binary Heaps

Binary heaps also rely on logarithmic height. When you insert a new element or remove the root from a binary heap, the operation "bubbles up" or "sifts down" through the levels of the tree. Because a binary heap is a complete binary tree, its height is O(log n). A heap with one million elements has a height of only about 20, so insertions and removals are extremely fast even at large scale.

#### Exponentiation by Squaring

This is a clever technique that computes x^n in O(log n) multiplications instead of the naive O(n). The key insight is that you can break the exponent down using its binary representation: x<sup>10</sup> = (x<sup>5</sup>)<sup>2</sup>, and x<sup>5</sup> = x * (x<sup>2</sup>)<sup>2</sup>. Here is a Java implementation:

```java
public static long power(long base, int exp) {
    long result = 1;
    while (exp > 0) {
        if (exp % 2 == 1) {
            result *= base;
        }
        base *= base;
        exp /= 2;
    }
    return result;
}
```

With this approach, computing 2<sup>1000</sup> takes only about 10 multiplications instead of 1,000. The loop runs once for each bit in the binary representation of the exponent, and log(1000) is roughly 10.

The common thread tying all of these together is straightforward: every O(log n) algorithm works by halving (or otherwise dividing) the problem space at each step. Whether you are navigating a tree, sifting through a heap, or squaring a base, the logarithm appears because you are repeatedly cutting the remaining work in half.

### O(n log n): The Sweet Spot for Sorting

Once you understand O(log n), the next complexity class to appreciate is O(n log n). The best way to see it in action is through **Merge Sort**.

Merge Sort works by recursively splitting the array in half until each piece has one element, then merging those pieces back together in sorted order. The splitting produces log n levels of recursion (halving each time), and at every level the merging step touches all n elements exactly once. Multiply those together and you get n * log n total work.

To put concrete numbers on it: sorting 1,000 elements with Merge Sort takes roughly 1,000 * 10 = 10,000 operations. Bubble Sort, by contrast, would take about 1,000 * 1,000 = 1,000,000 operations. That is a 100x difference, and the gap only grows as the input gets larger.

The O(n log n) bound is not just a nice property of Merge Sort. It is the proven **lower bound** for comparison-based sorting: no comparison sort can do better in the general case. This was established through decision tree arguments, since any algorithm that sorts by comparing elements must make at least n log n comparisons to distinguish between all n! possible orderings of the input.

Here is how the most common sorting algorithms compare against that bound:

| Algorithm | Best Case | Average Case | Worst Case | Stable? |
|---|---|---|---|---|
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n<sup>2</sup>) | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | No |
| Bubble Sort | O(n) | O(n<sup>2</sup>) | O(n<sup>2</sup>) | Yes |
| Insertion Sort | O(n) | O(n<sup>2</sup>) | O(n<sup>2</sup>) | Yes |

Notice that the three efficient general-purpose sorts (Merge Sort, Quick Sort, Heap Sort) all hit that O(n log n) ceiling. The quadratic sorts only achieve O(n) in their best case because they detect already-sorted input, not because they have a fundamentally faster algorithm.

> **Related posts**: [Big O Notation](/posts/big-o-notation/), [Trees](/posts/trees/), [Complexity Analysis](/posts/complexity-analysis/)
