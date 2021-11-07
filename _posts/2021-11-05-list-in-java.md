---
title: "Lists in Java"
categories:
  - Blog
tags:
  - Collections
  - Lists
---

### List

The Java List interface, java.util.List, represents an ordered sequence of objects. The elements contained in a Java List can be inserted, accessed, iterated and removed according to the order in which they appear internally in the Java List. 
The Java List interface is a standard Java interface, and it is a subtype of the Java Collection interface, meaning List inherits from Collection.

#### Implementations

Since List is an interface you need to instantiate a concrete implementation of the interface in order to use it. You can choose between the following List implementations:

* java.util.ArrayList
* java.util.LinkedList
* java.util.Vector
* java.util.Stack

Of these implementations, the ArrayList is the most commonly used.

#### ArrayList

Java ArrayList class uses a dynamic array (increases its size by 50%) to storing the elements. 
It is like an array, but there is no size limit. We can add or remove any element anytime. 
ArrayList works at the index basis.
The ArrayList in Java can have the duplicate elements as well. 
**It is not synchronized.**
Its maintains the insertion order internally.
**ArrayList provides fast iteration of elements using indexing.**
It inherits the AbstractList class and implements List interface.

#### LinkedList

Implements the Collection interface.
It uses a doubly linked list internally to store the elements.
LinkedList can store the duplicate elements. 
Efficiently utilize memory ,i.e no need to pre-allocate memory, very efficient for fast removal and addition of elements, but has slower iteration of elements as compared to ArrayList.
It maintains the insertion order and **is not synchronized**. 
In LinkedList, **the manipulation is fast because no shifting is required.**


#### Vector 

It is like the dynamic array which can grow or shrink its size (increases its size by 100%).
Vector is synchronized, so it is recommended to use the Vector class in the thread-safe implementation only. If you don't need to use the thread-safe implementation, you should use the ArrayList, the ArrayList will perform better in such case.

It is similar to the ArrayList, but with tree differences: 

* Vector increases its size by 100%
* Vector is synchronized.
* Java Vector contains many legacy methods that are not the part of a collections framework.

#### Stack

The stack is a linear data structure that is used to store the collection of objects. **It is based on Last-In-First-Out (LIFO).**
Stack is a class that falls under the Collection framework that **extends the Vector class.** It also implements interfaces List, Collection, Iterable, Cloneable, Serializable. It represents the LIFO stack of objects. 








