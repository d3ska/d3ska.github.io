---
title: "De Morgan's Laws"
categories:
  - Software Design
tags:
  - De Morgan's Laws
  - Boolean Logic
  - Clean Code
---

Boolean conditions in code tend to accumulate negations. A method might check `!a && !b && !c`, and after staring at it for a moment you realize it means "none of these are true." **De Morgan's Laws** give you a systematic way to rewrite such expressions into equivalent forms that are often easier to read.

### The Two Laws

De Morgan's Laws define how negation distributes over `AND` and `OR`:

**First law:** the negation of a disjunction (OR) is the conjunction (AND) of the negations.

```
!(A || B)  ==  !A && !B
```

**Second law:** the negation of a conjunction (AND) is the disjunction (OR) of the negations.

```
!(A && B)  ==  !A || !B
```

These laws work in both directions. You can factor a negation out of individual terms into a single outer negation, or distribute a single outer negation across the terms. Both directions are useful depending on which form is more readable in context.

### Applying the Laws in Code

Consider a condition that checks whether none of the three flags are set:

```java
if (!a && !b && !c) {
    // none are true
}
```

We read this as "not A, and not B, and not C." It works, but the repeated negations add cognitive load. Applying De Morgan's first law in reverse, we can factor out the negation:

```java
if (!(a || b || c)) {
    // none are true
}
```

Now it reads as "none of A, B, or C are true," which communicates the same intent more directly.

The transformation works the other way too. If we see:

```java
if (!a || !b || !c) {
    // at least one is false
}
```

We can apply the second law in reverse:

```java
if (!(a && b && c)) {
    // not all are true (at least one is false)
}
```

The rewritten version immediately tells us "not all conditions are met."

### Nested Conditions

De Morgan's Laws apply just as well to nested boolean expressions. Consider negating a condition like this:

```java
if ((a && b) || (c && d)) {
    // at least one pair is true
}
```

To negate the entire expression, work from the outside in. First, apply the first law to flip the outer `OR` into an `AND`:

```java
!(a && b) && !(c && d)
```

Then apply the second law to each inner group:

```java
(!a || !b) && (!c || !d)
```

The final negated form:

```java
if ((!a || !b) && (!c || !d)) {
    // neither pair is fully true
}
```

Each step is mechanical. Flip the operator, negate the operands, and repeat for any inner groups.

### Practical Refactoring Example

Let's say I have a method that checks whether a user should be denied access:

```java
// Before: a chain of negations
if (!user.isActive() || !user.hasValidSubscription() || !user.isEmailVerified()) {
    denyAccess();
}
```

Applying the second law (`!A || !B || !C` equals `!(A && B && C)`):

```java
// After: one negation wrapping positive conditions
if (!(user.isActive() && user.hasValidSubscription() && user.isEmailVerified())) {
    denyAccess();
}
```

Now the intent is clear: deny access unless **all three** conditions are satisfied. We could even extract the inner expression into a method for maximum readability:

```java
private boolean isFullyAuthorized(User user) {
    return user.isActive() && user.hasValidSubscription() && user.isEmailVerified();
}

if (!isFullyAuthorized(user)) {
    denyAccess();
}
```

### A Note on Comparison Operators

When manually negating a compound condition, you also need to negate each comparison operator: `<` becomes `>=`, `>` becomes `<=`, `==` becomes `!=`, and vice versa. This is not De Morgan's Law itself, but it comes up naturally when applying it to conditions that involve comparisons.

For example, negating `(age >= 18 && score > 50)` gives `(age < 18 || score <= 50)`. De Morgan's Law handles the `AND`-to-`OR` transformation and the outer negation, while the individual operator flips follow from standard logical negation.

### Short-Circuit Evaluation

One thing to keep in mind: transforming between `&&` and `||` changes short-circuit behavior. `&&` stops evaluating as soon as it hits the first `false`, while `||` stops at the first `true`. If any of your conditions have side effects or are expensive to compute, the order and number of evaluations can differ between the original and transformed form. In practice this rarely causes issues because well-written conditions tend to be side-effect-free, but it is worth being aware of.

### When It Helps

De Morgan's Laws are not about making code shorter. They are about picking the form that best communicates intent. A chain of negated ORs like `!a || !b || !c` is easier to understand as `!(a && b && c)` because the latter reads as a single thought: "not all of these hold." Conversely, `!a && !b && !c` becomes `!(a || b || c)`: "none of these hold."

The transformation is mechanical, which means you can apply it confidently without worrying about introducing bugs. When you spot a conditional that's hard to parse because of scattered negations, try applying De Morgan's Laws and see if the result reads better.

Modern IDEs like IntelliJ IDEA and Eclipse can apply De Morgan's transformation automatically as a refactoring action. In IntelliJ, placing the cursor on a `&&` or `||` operator and pressing Alt+Enter will offer to negate the expression for you. It is a quick way to explore which form reads best without doing the rewrite by hand.
