---
title: "DeMorgan’s Laws"
categories:
  - Software Design
tags:
  - DeMorgan's Laws
  - Boolean Logic
  - Clean Code
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


**Additionally, when negating compound conditions, each comparison operator is also negated:**

* < becomes >=

* \> becomes <=

* == becomes !=

* <= becomes >

* \>= becomes <

* != becomes ==


**Why it is useful during programming?**

It could help us and others to more easily read the if statements, and it's also useful to refactor some if's monsters (if you know what I mean).

**Examples**

* (!a && !b && !c), we will read it as, "not 'a', not 'b' and not 'c'".

after change: 

* !(a \|\| b \|\| c), by using the second law, we can simply read this as "at least one of these is required."

<br/>
<br/>

* (!a \|\| !b \|\| !c), after a while we see that all requirements must be met.

after change:

* !(a && b && c), now we see it immediately

<br/>

**Practical Refactoring Example**

Let's say I have a method that checks whether a user should be denied access:

```java
// Before: a chain of negations -- hard to parse at a glance
if (!user.isActive() || !user.hasValidSubscription() || !user.isEmailVerified()) {
    denyAccess();
}
```

Applying De Morgan's first law (`!A || !B || !C` equals `!(A && B && C)`), we can rewrite it as:

```java
// After: one negation wrapping positive conditions -- reads naturally
if (!(user.isActive() && user.hasValidSubscription() && user.isEmailVerified())) {
    denyAccess();
}
```

Now the intent is clear: deny access unless **all three** conditions are satisfied. We could even extract the inner expression into a helper method for maximum readability:

```java
private boolean isFullyAuthorized(User user) {
    return user.isActive() && user.hasValidSubscription() && user.isEmailVerified();
}

if (!isFullyAuthorized(user)) {
    denyAccess();
}
```

<br/>

**Summary**

De Morgan's Laws can help simplify your code to make it more readable. You can change a sequence of negated ands to something that reads as “at least one of these is required”, and a sequence of negated ors to something that reads as “all of these are required.”



