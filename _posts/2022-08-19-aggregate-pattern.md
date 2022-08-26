---
title: "Aggregate Pattern"
categories:
  - Blog
tags:
  - Aggregate
  - Patterns
  - Domain Driven Design
  - DDD
---

The Aggregate is one of the main pattern in Domain Driven Design (DDD) introduced by Martin Fowler.

How you may already deduced, it's aggregating, but what?

It’s a cluster of domain objects that can be treated as a single unit. So its aggregating some part of business logic. <br>
Be aware that it's not aggregating it from our 'contains' perspective as people often perceive it, I mean if client has addresses, and
phone numbers or whatever, it does not mean that it should be stored in out aggregate.  
It should store necessary data that will make it possible to make a consistent and independent decision about some part of bossiness.
Like a black box that answer for some question without any external data. 


Let’s take a look on characteristic of that and some example:

- can make decisions and carry out a consistent change of a certain data set
- it's root and stores in its interior a graph of other objects
- has a clearly defined identity
- its operation can be specified with the injected policy code e.g. using a strategy pattern.
- it increases the readability of the code
- there is a unit of consistent change in the system 
- it can be tested in unit test



An example of that may be a basket with products, order with line item, some kind of reservation and so on.

What does an Aggregate looks like ?

![img]({{site.url}}/assets/blog_images/2022-08-19-aggregate-pattern/aggregate-pattern-01.png)


"The boundary defines the contents of the aggregate. It is also the barrier between the aggregate contents and the rest of the application. Nothing outside the boundary can keep a reference to anything inside the boundary."


