---
title: "Hash Tables"
categories:
  - Blog
tags:
  - Data Structures
  - Complexity Analysis
  - Hash Tables
---

A data structure that store data in key-value pairs and provides fast insertion, deletion and searching.

Under the hood it use a dynamic array of linked lists to efficiently store key/value pairs.

When searching for some value or inserting a key/value pair, a hash function firstly map our key to an integer value which gonna be our index in the underlying dynamic array.

Then the value associated with the key is added to the linked list stored at the index in the dynamic array, and the reference to the key is stored with the value as well, so in case if the collision occur and more then one element will be stored in the same linked list,  it would know which value is searched.

How ever hash tables rely on very powerful and highly optimized  hash functions to minimize the number of collisions that occur when two or more keys hashed to to the same index.

Visualization of how HashTable may look under the hood:

[<br>
0: (key1, value1) ⇒ null<br>
1: (key2, value2) ⇒ (key3, value3)  ⇒ (key4, value4)<br>
2: (key3, value3) ⇒ null<br>
3: (key4, value4) ⇒ (key5, value5)<br>
4: null <br>
5: (key6, value6) ⇒ null<br>
]

In the hash table above, the keys key2, key3, key4 collided by all being map to linked list under index 1 and the key4 and key5 collided by both being hashed to index 5.

Time complexity, as mentioned on the beginning they are fast and provide basics operations in following complexity:

- Inserting a pair: O(1) average, O(n) worst case
- Removing a pair: O(1) average, O(n) worst case
- Searching a pair: O(1) average, O(n) worst case

The worst case, linear-time complexity occurs when a lot of collision happen leading to long linked list, but it happen extremely rarely as hash function nowadays are really and constant-time complexity is all but guaranteed.