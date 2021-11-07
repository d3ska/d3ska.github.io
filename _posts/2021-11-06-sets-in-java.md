---
title: "Sets in java"
categories:
  - Blog
tags:
  - Collections
  - Sets
---

## Set

A Set is a Collection that cannot contain duplicate elements. 
It models the mathematical set abstraction. 
The Set interface contains only methods inherited from Collection and adds the restriction that duplicate elements are prohibited. 
Set also adds a **stronger contract on the behavior of the equals and hashCode operations**, allowing Set instances to be compared meaningfully even if their implementation types differ. 
Two Set instances are equal if they contain the same elements. By default, sets implementations are not synchronized.

<br/>

### Implementations

Since Set is an interface you need to instantiate a concrete implementation of the interface in order to use it. You can choose between the following Set implementations: 

* [HashSet](https://matthewonsoftware.com/blog/sets-in-java/#hashset)
* [LinkedHashSet](https://matthewonsoftware.com/blog/sets-in-java/#linkedhashset)
* [TreeSet](https://matthewonsoftware.com/blog/sets-in-java/#treeset)
* [EnumSet](https://matthewonsoftware.com/blog/sets-in-java/#enumset)

Of these implementations, the HashSet is the most commonly used.

![img]({{site.url}}/assets/blog_images/2021-11-06-sets-in-java/java-set-implementation.png)

<br/>

### HashSet

Java HashSet class is used to create a collection that uses a hash table for storage. It inherits the AbstractSet class and implements mentioned Set interface. 
The important points about Java HashSet class are:
* It stores unique elements and permits nulls
* It's backed by a HashMap

**Performance**

The performance of a HashSet is affected mainly by two parameters – its Initial Capacity and the Load Factor.
The expected time complexity of adding an element to a set is O(1) which can drop to O(n) in the worst case scenario (only one bucket present) – therefore, it's essential to maintain the right HashSet's capacity.

<br/>

### LinkedHashSet

The LinkedHashSet is an ordered version of HashSet that maintains a doubly-linked List across all elements. 
That's why it requires more storage than HashSet, because LinkedHashSet maintains linked lists internally.
When the iteration order is needed to be maintained this class is used. 
If we would like to iterate through elements in LinkedHashSet, the elements will be at the same order as they were inserted.

**Performance** 

The performance of LinkedHashSet is slower than HashSet. It is because of linked lists present in LinkedHashSet

<br/>

### TreeSet 

TreeSet is a sorted collection that extends the AbstractSet class and implements the NavigableSet interface.
However, we can customize the sorting of elements by using the [Comparable or Comparator](https://matthewonsoftware.com/blog/comparable-and-comparator-interfaces/) interface.
The TreeSet uses a self-balancing binary search tree, more specifically a Red-Black tree.

**Performance**

When compared to a HashSet the performance of a TreeSet is on the lower side. 
Operations like add, remove and search take O(log n) time while operations like printing n elements in sorted order require O(n) time.

<br/>

### EnumSet









