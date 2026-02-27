---
title: "Hash Tables"
categories:
  - Data Structures
tags:
  - Data Structures
  - Complexity Analysis
  - Hash Tables
  - HashMap
---

A data structure that stores data in key-value pairs and provides fast insertion, deletion and searching.

Under the hood it uses a dynamic array of linked lists to efficiently store key/value pairs. This approach to handling collisions is called **separate chaining** -- each array slot points to a linked list of entries that hashed to the same index. An alternative strategy is **open addressing**, where all entries are stored directly in the array itself. When a collision happens, the algorithm probes for the next available slot using techniques like linear probing (checking the next slot sequentially) or quadratic probing (checking slots at quadratically increasing offsets).

When searching for some value or inserting a key/value pair, a hash function first maps our key to an integer value which will be our index in the underlying dynamic array.

Then the value associated with the key is added to the linked list stored at the index in the dynamic array, and the reference to the key is stored with the value as well, so in case if a collision occurs and more than one element will be stored in the same linked list, it would know which value is searched.

However, hash tables rely on very powerful and highly optimized hash functions to minimize the number of collisions that occur when two or more keys hash to the same index.

Visualization of how HashTable may look under the hood:

[<br>
0: (key1, value1) => null<br>
1: (key2, value2) => (key3, value3) => (key4, value4)<br>
2: (key5, value5) => null<br>
3: (key6, value6) => (key7, value7)<br>
4: null <br>
5: (key8, value8) => null<br>
]

In the hash table above, the keys key2, key3, key4 collided by all being mapped to the linked list at index 1, and key6 and key7 collided by both being mapped to index 3.

**Load factor and resizing:** The load factor is the ratio of stored entries to the total number of slots in the array. As the load factor grows, collisions become more frequent and performance degrades. To maintain efficient operations, hash tables monitor this ratio and resize when it crosses a threshold (commonly 0.75). Resizing typically means allocating a new array of double the size, then rehashing every existing entry into the new array. This single resize operation is O(n), but because it happens infrequently, the amortized cost of insertion remains O(1).

**Keys must be hashable:** For a hash table to work correctly, keys must produce consistent hash values. In Java, this means keys must properly implement `hashCode()` and `equals()`. In Python, only immutable types (strings, numbers, tuples) can serve as dictionary keys. Using a mutable object as a key can lead to lost entries, because if the object changes after insertion its hash value may change too, making it impossible to locate.

**Real-world implementations:** Most languages provide a built-in hash table. Java offers `HashMap` (and the thread-safe `ConcurrentHashMap`), Python has `dict`, C++ provides `std::unordered_map`, and JavaScript uses plain objects and `Map`.

Time complexity, as mentioned at the beginning they are fast and provide basic operations in the following complexity:

- Inserting a pair: O(1) average, O(n) worst case
- Removing a pair: O(1) average, O(n) worst case
- Searching a pair: O(1) average, O(n) worst case

The worst case, linear-time complexity occurs when a lot of collisions happen leading to long linked lists, but it happens extremely rarely as hash functions nowadays are really well optimized and constant-time complexity is all but guaranteed.
