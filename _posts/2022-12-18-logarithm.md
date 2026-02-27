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
