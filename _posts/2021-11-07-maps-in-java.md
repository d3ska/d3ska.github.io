---
title: "Maps in java"
categories:
  - Blog
tags:
  - Collections
  - Sets
---

## Map

Map is an interface that is part of Java's Collections Framework. Unlike lists and sets, maps do not implement the Collection interface.
It's an object that maps keys to values. A map cannot contain duplicate keys; each key can map to at most one value.
All of above maps are not synchronized. 

### Implementations

Since Map is an interface you need to instantiate a concrete implementation of the interface in order to use it. You can choose between the following Map implementations: 

* [HashMap](https://matthewonsoftware.com/blog/maps-in-java/#hashmap)
* [LinkedHashMap](https://matthewonsoftware.com/blog/maps-in-java/#linkedhashmap)
* [TreeMap](https://matthewonsoftware.com/blog/maps-in-java/#treemap)


Of these implementations, the HashMap is the most commonly used.

![img]({{site.url}}/assets/blog_images/2021-11-06-sets-in-java/java-set-implementation.png)

<br/>

### HashMap

Key reason of using HashMap is performance. Comparing to a list, if we want find element in list, the time complexity is O(n), and if list is sorted, it will be O(log n).
The advantage of a HashMap is that the time complexity to insert and retrieve a value is O(1) on average. It's also a very similar class very Hashtable. 


### LinkedHashMap

The LinkedHashMap class is very similar to HashMap in most aspects. However, the linked hash map is based on both hash table and linked list to enhance the functionality of hash map.
The insertion order in LinkedHashMap is always maintained, and it is not affected if a key is re-inserted into the map.

LinkedHashMap provides a special constructor which enables us to specify, among custom load factor (LF) and initial capacity, a different ordering mechanism/strategy called access-order:

```java
LinkedHashMap<Integer, String> map = new LinkedHashMap<>(22, .75f, true);
```

The first parameter is the initial capacity, the second one is the load factor, and the last one is ordering mode.
So, by passing in true, we turned on access-order, whereas the default was insertion-order.
**Order of elements in the key set is transformed as we perform access operations on the map.**

**Performance**

LinkedHashMap has similar capabilities for the basic operations like add, remove, contains as HashMap.
However, this constant-time performance of LinkedHashMap is likely to be a little worse than the constant-time of HashMap due to the added overhead of maintaining a doubly-linked list.
But on the other hand linear time performance during iteration is better than HashMap‘s linear time.
This is because, for LinkedHashMap, n in O(n) is only the number of entries in the map regardless of the capacity. Whereas, for HashMap, n is capacity, and the size summed up, O(size+capacity).

### TreeMap

TreeMap is implementation that keep its entities sorted in default order if there is any or given one.
Unlike a hash map and linked hash map, does not employ the hashing principle anywhere since it does not use an array to store its entries.

TreeMap implements NavigableMap interface and bases its internal working on the principles of red-black trees.
Its self-balancing binary search tree. Red-black data structures is based on nodes.

**Performance**

Mentioned above data structure guarantee that basic operations like search, get, put and remove take logarithmic time O(log n).



### Which map choose

* [HashMap](https://matthewonsoftware.com/blog/maps-in-java/#hashmap) - is good for general-purpose, very efficient but its doesn't insert entities in order nor sort them, so it's a little chaotic.

* [LinkedHashMap](https://matthewonsoftware.com/blog/maps-in-java/#linkedhashmap) - is has good attributes from HashMap, and store entities in insertion order. It performs better in operations where there is a lot of iterations, because it 
only number of entities is taken into account regardless of capacity.

* [TreeMap](https://matthewonsoftware.com/blog/maps-in-java/#treemap) - provides possibility of sorting keys by default or custom order. However, it offers worse general performance than the other two alternatives.

