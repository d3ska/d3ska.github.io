---
title: "Logarithm"
categories:
  - Blog
tags:
  - Data Structures
  - Complexity Analysis
---

It’s mathematical concept which is very often used in Computer Science in context of algorithms complexity, it’s even sounds similar 😀

The equation:

log<sub>x</sub>(n) = y    iff     <sub>x</sub><sup>y</sup> = n

log<sub>2</sub>(n) = y    iff     <sub>2</sub><sup>y</sup> = n

<br>
By default, in cs we assume that based of algorithm is 2, so we don’t even write it.

log<sub>2</sub>(n) = y   →   log(n) = y    iff   <sub>2</sub><sup>y</sup> = n

<br>
So in other words what does it imply?
The Log of N is the power to which you need to put 2  to get N.

For example:

log(1) = 0      because    <sub>2</sub><sup>0</sup>

log(4) = 2    because    <sub>2</sub><sup>2</sup>

log(256) = 8     because    <sub>2</sub><sup>8</sup>

<br>
But noticed one thing, as  y  is doubled the Log of N increase just by one.

log(16) = 4      (<sub>2</sub><sup>3</sup> * 2 = <sub>2</sub><sup>4</sup>)
                
log(32) = 5      (<sub>2</sub><sup>4</sup>  * 2 = <sub>2</sub><sup>5</sup>)

<br>

What does that mean for us? If we get algorithm with complexity log(n) its incredible good!
When N doubles, log(n) increases only by one, if we tie this back to complexity analysis, when we have an algorithm with a time complexity of log(n), that is incredibly good! That means as the input doubles, as the input increases, the number of elementary operations that we’re performing in the algorithm increases only by one.

Imagine a linear time complexity algorithm with an input of size 1000, it might take roughly 1000 operations to complete, whereas a logarithmic time complexity algorithm with the same input data would take roughly 10 operations to complete, since <sub>2</sub><sup>10</sup> = 1000.

Example of algorithm with complexity O(log(n)) may be a binary search.