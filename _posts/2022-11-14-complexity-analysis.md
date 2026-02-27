---
title: "Complexity Analysis"
categories:
  - Data Structures
tags:
  - Complexity Analysis
  - Big O
  - Algorithms
  - Time Complexity
  - Space Complexity
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

### How to Calculate Time Complexity

Calculating time complexity boils down to counting the number of elementary operations an algorithm performs relative to its input size. Here are the most common patterns you will run into.

A **single loop** that iterates over n elements runs in O(n) time:

```java
int sum = 0;
for (int i = 0; i < n; i++) {
    sum += arr[i];
}
```

**Nested loops** where each loop runs up to n iterations give us O(n²):

```java
for (int i = 0; i < n; i++) {
    for (int j = i + 1; j < n; j++) {
        if (arr[i] == arr[j]) {
            System.out.println("Duplicate found");
        }
    }
}
```

**Sequential (non-nested) operations** are added together and then simplified. Two separate loops each running n times give O(n) + O(n) = O(2n), which simplifies to O(n) because we drop the constant factor:

```java
for (int i = 0; i < n; i++) {
    process(arr[i]);
}
for (int i = 0; i < n; i++) {
    log(arr[i]);
}
```

A **logarithmic** pattern appears when we halve the input on every step, giving O(log n):

```java
int x = n;
while (x > 1) {
    x = x / 2;
}
```

When simplifying, always **drop constants and lower-order terms**. An expression like O(2n + 100) simplifies to O(n). The constants do not matter at scale because Big O describes growth rate, not exact operation count.

Finally, watch out for **multiple independent inputs**. If a method accepts two arrays of different sizes, the complexity depends on both:

```java
void processBoth(int[] a, int[] b) {
    for (int val : a) {
        handle(val);
    }
    for (int val : b) {
        handle(val);
    }
}
```

This is O(n + m), not O(n), because `a` and `b` can have completely different lengths. Treating them as the same variable would misrepresent the actual work the algorithm does.

### Best, Average, and Worst Case

Not every input makes an algorithm work equally hard. Consider **linear search** — scanning an array for a target value:

- **Best case O(1):** the target is the very first element, so we return immediately.
- **Average case O(n/2) = O(n):** on average, the target sits somewhere around the middle of the array, so we check roughly half the elements. After dropping the constant, this is still O(n).
- **Worst case O(n):** the target is the last element or is not in the array at all, forcing us to inspect every single element.

The distinction becomes especially important with algorithms like **QuickSort**. Its average-case time complexity is O(n log n), but if we consistently pick a bad **pivot** (for example, always the smallest or largest element), the partitions become extremely unbalanced, and performance degrades to O(n²).

By convention, when we state the Big O of an algorithm we are referring to the **worst case** because it provides a hard upper-bound guarantee: the algorithm will never do worse than this. That said, some analyses — QuickSort being the classic example — emphasize the average case because the worst case is exceedingly rare when a good pivot selection strategy (such as random or median-of-three) is used.

### Common Complexity Classes at a Glance

The table below lists the most frequently encountered **complexity classes**, from fastest to slowest growth:

| Complexity | Name | Example |
|---|---|---|
| O(1) | Constant | Array access by index |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Single loop through array |
| O(n log n) | Linearithmic | Merge sort |
| O(n²) | Quadratic | Nested loops (bubble sort) |
| O(2^n) | Exponential | Recursive Fibonacci (naive) |
| O(n!) | Factorial | Generating all permutations |

A useful rule of thumb: modern hardware can handle roughly **10^8 simple operations per second**. That means an O(n²) algorithm becomes impractical once n exceeds about 10,000, while an O(n log n) algorithm comfortably handles inputs up to around n = 10^7.

For related topics, see [Logarithm](https://matthewonsoftware.com/blog/logarithm/) and [Big O Notation](https://matthewonsoftware.com/blog/big-o-notation/).
