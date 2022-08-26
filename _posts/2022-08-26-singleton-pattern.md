---
title: "Singleton Pattern"
categories:
  - Blog
tags:
  - Aggregate
  - Patterns
  - Domain Driven Design
  - DDD
---

#### Singleton

Is a one of the most known design patterns. To be more particular, it's a creational design pattern, which ensures that a class would have just one instance.
So the all clients of that class would reuse that instance, in fact the only one, that's why it's called singleton. 

You may be wondering why would someone want to have single instance of some class.
Let's imagine some shared resource, like class with database connection, or some service class, or any other class which should 
have just a single instance available to all clients.


Please note that we would need some 
