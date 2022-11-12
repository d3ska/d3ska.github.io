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
**Set also adds a stronger contract on the behavior of the [equals and hashCode](https://matthewonsoftware.com/blog/java-equals-and-hashCode-contract/) operations**, allowing Set instances to be compared meaningfully even if their implementation types differ. 
Two Set instances are equal if they contain the same elements. By default, sets implementations are not synchronized.


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


### LinkedHashSet

The LinkedHashSet is an ordered version of HashSet that maintains a doubly-linked List across all elements. 
That's why it requires more storage than HashSet, because LinkedHashSet maintains linked lists internally.
When the iteration order is needed to be maintained this class is used. 
If we would like to iterate through elements in LinkedHashSet, the elements will be at the same order as they were inserted.

**Performance** 

The performance of LinkedHashSet is slower than HashSet. It is because of linked lists present in LinkedHashSet

### TreeSet 

TreeSet is a sorted collection that extends the AbstractSet class and implements the NavigableSet interface.
However, we can customize the sorting of elements by using the [Comparable or Comparator](https://matthewonsoftware.com/blog/comparable-and-comparator-interfaces/) interface.
The TreeSet uses a self-balancing binary search tree, more specifically a Red-Black tree.

**Performance**

When compared to a HashSet the performance of a TreeSet is on the lower side. 
Operations like add, remove and search take <br/> O(log n) time while operations like printing n elements in sorted order require O(n) time.


### EnumSet

An EnumSet is a specialized Set collection to work with enum classes. It implements the Set interface and extends from AbstractSet:
When we want to use EnumSet we should take into consideration facts below:

* It can contain only enum values and all the values have to belong to the same enum
* It uses a fail-safe iterator that works on a copy, so it won't throw a ConcurrentModificationException if the collection is modified when iterating over it.
* It's not thread-safe, so we need to synchronize it externally if required
* The elements are stored following the order in which they are declared in the enum
* It doesn't allow adding null values, throwing a NullPointerException in an attempt to do so

**Implementations**

EnumSet is a public abstract class, but JDK provides 2 different implementations – are package-private and backed by a bit vector:

* RegularEnumSet -  uses a single long to represent the bit vector. So it can store up to 64 elements.
* JumboEnumSet -  uses an array of long elements as a bit vector. This lets this implementation store more than 64 elements.

It takes in account only size of the enum class, not elements that will be stored in the collection.

**Performance**

EnumSet that we've described above, all the methods in an EnumSet are implemented using arithmetic bitwise operations. 
These computations are very fast and therefore all the basic operations are executed in a constant time.




