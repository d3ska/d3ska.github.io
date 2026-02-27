---
title: "Linked Lists"
categories:
  - Data Structures
tags:
  - Data Structures
  - Complexity Analysis
  - Linked Lists
---

A linked list is conceptually similar to an array, as both store ordered collections of elements.

Where a linked list differs from an array is how it's implemented, or rather how it's stored in memory.

When we store an array for example [3,0,6,7] we would need 16 back to back memory slots to store that array, assuming that it contains 32-bit integers, and it has to be free space back to back.

A linked list stores elements anywhere in memory and connects them using pointers. Each node occupies two memory slots back to back: one for the value and one for a pointer to the next node, which could be literally anywhere in the memory space. Note that the actual size of each node depends on the data type and the platform; for instance, on a 64-bit system a pointer alone takes 8 bytes. Because of this pointer overhead, linked lists have a higher per-element memory cost than arrays.

A simple linked list node in Java looks like this:

```java
class Node {
    int value;
    Node next;

    Node(int value) {
        this.value = value;
        this.next = null;
    }
}
```

Summarizing, array elements are stored back to back in memory, while linked list elements are not stored back to back and can be anywhere in memory.

**Space complexity** for a linked list is O(n), the same asymptotic class as an array. However, the constant factor is higher because each node carries an additional pointer (or two, in a doubly linked list). For very large collections of small values, this overhead can be significant.

**When to prefer a linked list over an array:** Linked lists excel when you need frequent insertions and deletions at known positions (head, tail, or a node you already have a reference to), since these operations are O(1) once you have the reference. Arrays, on the other hand, are the better choice when you need fast random access by index (O(1) vs O(n) for a linked list) or when memory locality matters for cache performance.

<br>

## Single Linked List

The following are a singly linked list's standard operations and their corresponding time complexities:

- Accessing the head: O(1)
- Accessing the tail: O(n)
- Accessing the middle: O(n)
- Inserting / Removing the head: O(1)
- Inserting / Removing the tail: O(n) to access + O(1)
- Inserting / Removing a middle node: O(n) to access + O(1)
- Searching for a value: O(n)

<br>

## Doubly Linked List

A doubly linked list maintains a reference to both the head and the tail of the list, and each node holds pointers to both the next and the previous node. This is why accessing the tail is O(1) -- the list always keeps a direct reference to it, so there is no need to traverse from the head.

- Accessing the head: O(1)
- Accessing the tail: O(1)
- Accessing the middle: O(n)
- Inserting / Removing the head: O(1)
- Inserting / Removing the tail: O(1)
- Inserting / Removing a middle node: O(n) to access + O(1)
- Searching for a value: O(n)
