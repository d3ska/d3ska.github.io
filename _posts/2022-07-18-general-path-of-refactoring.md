---
title: "How to approach refactoring"
categories:
  - Blog
tags:
  - Refactoring
  - Clean code
---

#### Introduce

Firstly, what does refactoring mean? The definition that I personally find most suitable is:
"It's changing the structure of code without changing observable behaviour." 

Also, what is the difference between refactoring and rewrite?
In fact, we can make some rewrites during refactoring, for example while we were refactoring a class we rewrite a function,
or during refactoring of a module, we may rewrite a class, so in other words, rewrite of some component is actually a refactor for the higher order component.

#### General path of refactoring

There are steps that we can keep in mind and follow, those advices are really generic but very helpful at the same time.
So as I mentioned before refactoring is a change of structure of code without changing an observable behaviour.
The key word here is OBSERVABLE BEHAVIOUR, so:

1. Ensure the proper security of the change by creating a test based on the observed observable behaviors at the appropriate level.
2. Introduce to the system a new concept.
3. Connect a new object and make sure that the observed behaviors remain unchanged.
4. Observe the achieved effect and make a decision about further actions

Those rules/steps make refactoring much easier, safer and even pleasant. 
Test on appropriate level means, that sometimes unit test will be enough, 
but in other case integration or E2E test can be necessary to cover and secure observable behaviour. 
With those we would approach refactoring feeling more safely and freely, because the reason why many people don't like to refactor is a fear of change. 
