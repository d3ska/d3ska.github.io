---
title: "Sets in java"
categories:
  - Blog
tags:
  - Collections
  - Sets
---

### Set

A Set is a Collection that cannot contain duplicate elements. 
It models the mathematical set abstraction. 
The Set interface contains only methods inherited from Collection and adds the restriction that duplicate elements are prohibited. 
Set also adds a **stronger contract on the behavior of the equals and hashCode operations**, allowing Set instances to be compared meaningfully even if their implementation types differ. 
Two Set instances are equal if they contain the same elements.


![img]({{site.url}}/assets/blog_images/2021-11-06-sets-in-java/java-set-implementation.png)

#### Implementations

Since Set is an interface you need to instantiate a concrete implementation of the interface in order to use it. You can choose between the following Set implementations: 

* HashSet
* LinkedHashSet
* TreeSet
* EnumSet

Of these implementations, the HashSet is the most commonly used.

#### HashSet

Java HashSet class is used to create a collection that uses a hash table for storage. It inherits the AbstractSet class and implements mentioned Set interface. 
The important points about Java HashSet class are: HashSet stores the elements by using a mechanism called hashing.
It offers constant time performance for the basic operations (add, remove, contains and size), but doesn't maintain the insertion order.


#### LinkedHashSet
The LinkedHashSet is an ordered version of HashSet that maintains a doubly-linked List across all elements. When the iteration order is needed to be maintained this class is used. 
If we would like to iterate through elements in LinkedHashSet, the elements will be at the same order as they were inserted.

#### TreeSet 


#### EnumSet









