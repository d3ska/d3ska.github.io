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

It's a mathematical concept which is very often used in Computer Science in the context of algorithm complexity — it even sounds similar!

The equation:

log<sub>x</sub>(n) = y &nbsp;&nbsp; iff &nbsp;&nbsp; x<sup>y</sup> = n

log<sub>2</sub>(n) = y &nbsp;&nbsp; iff &nbsp;&nbsp; 2<sup>y</sup> = n

<br>
By default, in Computer Science we assume that the base of the logarithm is 2, so we don't even write it. The reason base-2 is the natural choice comes from the binary nature of computing: every fundamental decision a computer makes is a binary choice — yes or no, 0 or 1, left or right. Algorithms like binary search split the input in half at each step, and that halving maps directly to powers of 2.

log<sub>2</sub>(n) = y &nbsp;&nbsp; -> &nbsp;&nbsp; log(n) = y &nbsp;&nbsp; iff &nbsp;&nbsp; 2<sup>y</sup> = n

<br>
So in other words what does it imply?
The log of N is the power to which you need to raise 2 to get N.

For example:

log(1) = 0 &nbsp;&nbsp;&nbsp;&nbsp; because &nbsp;&nbsp; 2<sup>0</sup> = 1

log(4) = 2 &nbsp;&nbsp;&nbsp;&nbsp; because &nbsp;&nbsp; 2<sup>2</sup> = 4

log(256) = 8 &nbsp;&nbsp;&nbsp;&nbsp; because &nbsp;&nbsp; 2<sup>8</sup> = 256

<br>
But notice one thing: as N is doubled, the log of N increases by just one.

log(16) = 4 &nbsp;&nbsp;&nbsp;&nbsp; (2<sup>3</sup> * 2 = 2<sup>4</sup>)

log(32) = 5 &nbsp;&nbsp;&nbsp;&nbsp; (2<sup>4</sup> * 2 = 2<sup>5</sup>)

<br>

What does that mean for us? If we get an algorithm with complexity log(n), it's incredibly good!
When N doubles, log(n) increases only by one, if we tie this back to complexity analysis, when we have an algorithm with a time complexity of log(n), that is incredibly good! That means as the input doubles, as the input increases, the number of elementary operations that we're performing in the algorithm increases only by one.

Imagine a linear time complexity algorithm with an input of size 1000, it might take roughly 1000 operations to complete, whereas a logarithmic time complexity algorithm with the same input data would take roughly 10 operations to complete, since 2<sup>10</sup> = 1024.

It is also worth noting that in Big O analysis, the base of the logarithm does not matter. This is because logarithms of different bases differ only by a constant factor — for example, log<sub>3</sub>(n) = log<sub>2</sub>(n) / log<sub>2</sub>(3), and 1/log<sub>2</sub>(3) is just a constant. Since Big O drops constant factors, O(log<sub>2</sub>(n)), O(log<sub>3</sub>(n)), and O(log<sub>10</sub>(n)) are all equivalent and written simply as O(log(n)).

### Binary Search: O(log n) in Action

Binary search is the classic example of a logarithmic algorithm. It works on a **sorted** array and repeatedly halves the search space:

```
function binarySearch(sortedArray, target):
    left = 0
    right = length(sortedArray) - 1
    while left <= right:
        mid = (left + right) / 2
        if sortedArray[mid] == target:
            return mid
        else if sortedArray[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```

At each step, we compare the target to the middle element and discard half of the remaining elements. If the array has 1024 elements, after one comparison we are down to 512, then 256, then 128, and so on. After at most 10 comparisons (since log<sub>2</sub>(1024) = 10) we have either found the target or determined it is not present. Doubling the array to 2048 elements only adds a single extra comparison — that is the power of O(log n).

### Other O(log n) Examples

Binary search on an array is not the only place where logarithmic complexity appears. The same "halving the problem" idea shows up in several fundamental data structures and algorithms.

**Balanced Binary Search Trees** work on exactly the same principle as binary search. At each node you compare your target to the current value and go either left or right, eliminating roughly half of the remaining candidates. A balanced tree holding 1000 nodes has a height of about 10 (since log₂(1000) ≈ 10), so finding any element takes at most 10 comparisons — regardless of which element you are looking for.

**Binary Heaps** also rely on logarithmic height. When you insert a new element or remove the root from a binary heap, the operation "bubbles up" or "sifts down" through the levels of the tree. Because a binary heap is a complete binary tree, its height is O(log n). A heap with one million elements has a height of only about 20, so insertions and removals are extremely fast even at large scale.

**Exponentiation by squaring** is a clever technique that computes x^n in O(log n) multiplications instead of the naive O(n). The key insight is that you can break the exponent down using its binary representation: x^10 = (x^5)^2, and x^5 = x * (x^2)^2. Here is a Java implementation:

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

With this approach, computing 2^1000 takes only about 10 multiplications instead of 1000. The loop runs once for each bit in the binary representation of the exponent, and log₂(1000) ≈ 10.

The common thread tying all of these together is straightforward: every O(log n) algorithm works by halving (or otherwise dividing) the problem space at each step. Whether you are navigating a tree, sifting through a heap, or squaring a base, the logarithm appears because you are repeatedly cutting the remaining work in half.

### O(n log n): The Sweet Spot for Sorting

Once you understand O(log n), the next complexity class to appreciate is O(n log n) — and the best way to see it is through **Merge Sort**. Merge Sort works by recursively splitting the array in half until each piece has one element, then merging those pieces back together in sorted order. The splitting produces log n levels of recursion (halving each time), and at every level the merging step touches all n elements exactly once. Multiply those together and you get n × log n total work.

To put concrete numbers on it: sorting 1000 elements with Merge Sort takes roughly 1000 × 10 = 10,000 operations. Bubble Sort, by contrast, would take about 1000 × 1000 = 1,000,000 operations. That is a 100x difference — and the gap only grows as the input gets larger.

Here is how the most common sorting algorithms compare:

| Algorithm | Best Case | Average Case | Worst Case | Stable? |
|---|---|---|---|---|
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | No |
| Bubble Sort | O(n) | O(n²) | O(n²) | Yes |
| Insertion Sort | O(n) | O(n²) | O(n²) | Yes |

It is worth noting that O(n log n) is the proven **lower bound** for comparison-based sorting — no comparison sort can do better in the general case. This was established through decision tree arguments: any algorithm that sorts by comparing elements must make at least n log n comparisons to distinguish between all n! possible orderings of the input.

### Real-World Logarithmic Scales

Logarithms are not just an algorithm concept — they appear everywhere in the real world:

- **Decibels**: sound intensity is measured on a logarithmic scale. Every 10 dB increase represents a 10x increase in intensity, which is why the jump from 60 dB to 70 dB is far more dramatic than it sounds.
- **Richter scale**: each whole number increase represents roughly a 10x increase in measured amplitude and about 31.6x more energy released.
- **Moore's Law**: transistor counts double roughly every two years — exponential growth that looks linear on a logarithmic plot, which is exactly why researchers prefer log-scale charts for semiconductor trends.
- Data visualization more broadly benefits from log scales: they make exponential data readable by compressing large ranges into manageable axes.

The core intuition is always the same: logarithms compress exponential growth into manageable, linear steps. Whether you are analyzing an algorithm, measuring an earthquake, or plotting transistor counts, the logarithm is the tool that tames the exponential.

For related topics, see [Trees](https://matthewonsoftware.com/blog/trees/) and [Big O Notation](https://matthewonsoftware.com/blog/big-o-notation/).
