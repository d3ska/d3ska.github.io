---
title: "DeMorgan’s Laws"
categories:
  - Blog
tags:
  - DeMorgan's Laws
  - Logical conditions
---

### DeMorgan's Laws

De Morgan’s laws state that specific Boolean statements can be written in different ways to the same effect.

The rules can be expressed in English as:

* the negation of a disjunction is the conjunction of the negations
* the negation of a conjunction is the disjunction of the negations

or

* the complement of the union of two sets is the same as the intersection of their complements
* the complement of the intersection of two sets is the same as the union of their complements

or

not (A or B) = (not A) and (not B)

not (A and B) = (not A) or (not B)


**The negation modifies each conditional as shown below**

* < becomes >=

* \> becomes <=

* == becomes !=

* <= becomes >

* \>= becomes <

* != becomes ==


**Why it is useful during programming?**

It could help us and others to more easily read the if statements, and it's also useful to refactor some if's monsters (if u know what I mean).

**Examples**

* (!a && !b && !c), we will read it as, "not 'a', not 'b' and not 'c'".

after change: 

* !(a \|\| b \|\| c), by using the second method, we can simply read this as “at least one of these is required.”

<br/>
<br/>

* (!a \|\| !b \|\| !c), so after while we see that all requirements must be met

after change:

* !(a && b && c), know it is easier to read and understand

<br/>

**Summary**

De Morgan's Laws can help simplify your code to make it more readable. You can change a sequence of negated ands to something that reads as “at least one of these is required”, and a sequence of negated ors to something that reads as “all of these are required.”



