---
title: "Collections in Java"
categories:
  - Blog
tags:
  - Collections
  - Data structure
---

### Collection

Java has numerous inbuilt data structures known as Collection types. It is a collection of interfaces and classes which helps in storing and processing the data efficiently.
The Collection interface (java.util.Collection) and Map interface (java.util.Map) are the two main “root” interfaces of Java collection classes.

![img]({{site.url}}/assets/blog_images/2021-10-24-transactions-with-spring-and-jpa/java-collection-framework-hierarchy.png)

Apart from these there some utilities classes such as Collections, Arrays that can be used to perform different operations on collection types in Java.

#### ArrayList and Vector Advantages

* Provides fast iteration of elements using indexing.

* It provides memory coherence meaning the elements are stored in a sequential memory location.

* Fast random access of elements due to memory coherence.

* Provides an optional option to define the size during its creation.

#### ArrayList and Vector Disadvantages


* Addition or deletion of data from the middle is time-consuming as data needs to be shifted to update the list

* Due to memory coherence, for larger elements a list will need significant contiguous blocks of memory. Hence, memory wastage is found.

* Resizing of an arraylist when it reaches its capacity with it initial capacity which is 10 is a costlier process as the elements will be copied from old to new space with 50% more capacity.

<br>

#### Advantages of LinkedList

* Efficient Memory Utilization ,i.e no need to pre-allocate memory.

* Very efficient for fast removal and addition of elements.

#### Disadvantages of LinkedList

* Slow iteration of elements as compared to ArrayList.

* Extra memory overhead is required to keep track of the pointer to the next element of the list.

* Inefficient random access.

<br>

#### Advantages of HashSet

* Allows null value and contains only unique values.

* The class offers constant time performance for the basic operations (add, remove, contains and size).

#### Disadvantages of HashSet

* Doesn't maintain the insertion order.

<br>

#### Advantages of TreeSet

* Elements of set will be sorted (ascending, natural, or the one specified by you via its constructor)

* Guarantees log(n) time cost for the basic operations (add, remove and contains)


#### Disadvantages of TreeSet

* Slower than HashSet.

* Always avoid implementing compareTo and equals inconsistently. If you need to order objects in a way that is inconsistent with equals, use a Comparator.

<br>

#### Advantages of LinkedHashSet

* Maintains insertion order of elements.

* Gives the performance of order O(1) for insertion, removal and retrieval operations.

#### Disadvantages of LinkedHashSet

* Not memory efficient as LinkedHashSet maintains LinkedList along with HashMap to store its elements.

<br>
<br/> 

### Map 

It an object that maps keys to values. A map cannot contain duplicate keys. There are three main implementations of Map interfaces: HashMap, TreeMap, and LinkedHashMap.

#### Advantages of HashMap

* Allows insertion of key value pair.

* It is non synchronized.HashMap cannot be shared between multiple threads without proper synchronization.

* Has a fail-fast iterator. 

* Faster access of elements due to hashing technology. 


#### Disadvantages of HashMap

* Potential of collision when 2 distinct keys generate the same hashCode() value worse the performance of the hashMap.

* Occasionally requires resizing when the original size of HashMap buckets are full. Resizing takes O(n) time as the elements from the previous hashtable/HashMap are transferred to a new bigger HashMap.

<br/>

#### Advantages of LinkedHashMap

* Maintains insertion order.

* Faster iteration with LinkedHashMap.

#### Disadvantages of LinkedHashMap

* Slower than HashMap for adding and removing elements.

<br/>

#### Advantages of TreeMap

* Stores key-value pairs in a sorted ascending order(based on the key).

* Lets you define a custom sort order.

* The retrieval speed of an element out of a TreeMap is fast, even in a TreeMap with a large number of elements.







