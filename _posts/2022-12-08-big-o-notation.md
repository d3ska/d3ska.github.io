---
title: "Big O Notation"
categories:
  - Data Structures
tags:
  - Data Structures
  - Big O Notation
  - Complexity Analysis
  - Algorithms
  - Time Complexity
---

The Big O Notation is a mathematical, asymptotic notation that describes time complexity and space complexity of algorithms/functions when the argument tends towards a particular value of infinity.<br>
For example, O(n) might be the same complexity of an algorithm that traverses through an array of length n, <br>similarly, O(n+m) might be the time complexity of an algorithm that traverses through an array of length n and through a string of length m.<br>
The following are examples of common complexities and their Big O notations, ordered from fastest to slowest

- Constant: O(1)
- Logarithmic: O(log(n))
- Linear: O(n)
- Log-linear: O(n log(n))
- Quadratic: O(n<sup>2</sup>)
- Factorial: O(n!)

![img]({{site.url}}/assets/blog_images/2022-08-01-big-o-notation/big-o-notation-complexity-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2022-08-01-big-o-notation/big-o-notation-complexity-dark.png){: .dark }

Sometimes there may be best/average/worst case scenarios depending on the input data, how they're structured and so on, so in other words it depends on the algorithm (quick sort for example) but Big O usually refers to the worst case scenario.

Examples of algorithms and their time complexity:

- fun(arr[]) => 1 + arr\[0]     We just return a static number and the first element of an array (assuming it is not empty), so the size of input data will not affect the complexity anyhow. The time complexity will be O(1), in other words constant.
- fun(arr []) => sum(arr)    Here we are summing the given array, so we have to traverse through the whole array. In other words, the more elements are in the given array, the more work our algorithm needs to perform, so it has linear complexity, in other words O(n).
- fun(arr []) => pair(arr)    Here we are pairing all elements, assume that it's done by nested for loops. Then guess what, it would require going through every element for each element. In the previous example it was O(n) to iterate over the given array. In this case it will be O(n<sup>2</sup>).

### Code Examples for Common Complexity Classes

**O(1) — Constant Time**

```
function getFirst(array):
    return array[0]
```
No matter how large the array is, accessing the first element always takes the same amount of time.

**O(log n) — Logarithmic Time**

```
function binarySearch(sortedArray, target):
    left = 0
    right = length(sortedArray) - 1
    while left <= right:
        mid = left + (right - left) / 2;
        if sortedArray[mid] == target:
            return mid
        else if sortedArray[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```
Each iteration cuts the search space in half, so doubling the input size only adds one extra step.

**O(n) — Linear Time**

```
function sumArray(array):
    total = 0
    for each element in array:
        total = total + element
    return total
```
We visit every element exactly once, so the time grows directly in proportion to the input size.

**O(n²) — Quadratic Time**

```
function printAllPairs(array):
    for i = 0 to length(array) - 1:
        for j = 0 to length(array) - 1:
            print(array[i], array[j])
```
For each element, we loop through every other element, so the number of operations grows as the square of the input size.

<br>

The constants in Big O Notation do not matter, and here is how to simplify it.


Let's assume that we have a function that does a bunch of elementary operations: it sums up a few numbers, declares some variables, creates an array with a few elements. It's going to be an algorithm that may run in O(25), but that would be written down to O(1). So you never want to say O(25) as it does not really make sense, and you should write it as O(1).
The second example might be an algorithm in which we iterate over a given array from left to right and from right to left.  That would be an O(2n) complexity, but you would drop the two because it's a constant, so it would be just O(n).
Also, an important thing to remember: let's assume that in our function we have a pairing algorithm and the one described just before. That is O(n<sup>2</sup> + 2n) but we would remove the two as it's a constant and n also as it becomes meaningless next to the n squared, so it would be O(n<sup>2</sup>).

Other example: O(n! + n<sup>3</sup> + log(n) + 3)   =>   O(n!)   as factorial would grow the fastest.
We were able to drop this in the above way as we were iterating on the same input data N.
Now assume that we do the same but for input we're going to get two different arrays of size M and N.

O(m<sup>2</sup> + 2n) can we transform it to O(m<sup>2</sup>)?
No! We can not! As those are different inputs, so we want to know the behaviour as it relates to these two different inputs, and you can imagine that there is actually a scenario where m<sup>2</sup> is tiny compared to N. <br>
Imagine M is equal to two and N is equal to a thousand, then m<sup>2</sup> would be smaller than N. That's why when you have two variables, you do always want to keep both of them.  So it has to remain O(m<sup>2</sup> + n) (we can still drop constants).

### Amortized Complexity

Some operations have a worst-case cost that is expensive but happens rarely enough that the **average cost per operation** over a sequence of operations is much lower. This is called **amortized complexity**. A classic example is appending to a dynamic array. Most appends are O(1) because there is room in the pre-allocated buffer. Occasionally, the array runs out of space and must allocate a new, larger buffer and copy all existing elements over — an O(n) operation. However, because the array doubles in capacity each time, that expensive copy happens less and less frequently. When you spread the cost of the occasional O(n) copy across all the cheap O(1) appends, each append costs O(1) **amortized**.

### Big O vs Big Theta vs Big Omega

Big O is the notation we use most, but it is worth knowing about its siblings:

- **Big O (O)** describes an **upper bound**. Saying an algorithm is O(n²) means the number of operations grows no faster than n² (up to a constant factor) for sufficiently large inputs. It is a ceiling, not an exact measurement.
- **Big Omega (Ω)** describes a **lower bound**. Saying an algorithm is Ω(n) means it will take at least n operations in the worst case.
- **Big Theta (Θ)** describes a **tight bound** — the algorithm is both O(f(n)) and Ω(f(n)). For instance, a simple loop over an array is Θ(n): it is at least linear and at most linear.

In everyday engineering and interview discussions, we almost always use Big O, and we typically mean it as a tight bound even though, formally, it only guarantees an upper bound.
