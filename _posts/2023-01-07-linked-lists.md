---
title: "Linked Lists"
categories:
  - Blog
tags:
  - Data Structures
  - Complexity Analysis
  - Linked Lists
---

Is very similar to array, at least conceptually, to an array.

Where linked lists differs from an array is how its implemented or rather how it's stored in memory.

When we store an array for example [3,0,6,7] we would need 16 back to back memory slots to store that array, assuming that it contains 32-bit integers, and it has to be free space back to back.

Linked lists  would store elements in the list anywhere in memory, and they’re going to connect the elements using pointers. Two memory slots back to back would be needed (for one node) where one store the value and the second one point to the next element (node), which could be literally anywhere in the memory space.

Summarizing, array elements are stored back to back in the memory, linked lis elements are not stored back to back, they can be anywhere in memory.

<br>

## Single Linked List

The following are a singly linked list’s standard operations and their corresponding time complexities:

- Accessing the head: O(1)
- Accessing the tail: O(n)
- Accessing the middle: O(n)
- Inserting / Removing the head: O(1)
- Inserting / Removing the tail: O(n) to access + O(1)
- Inserting / Removing a middle node: O(n) to access + O(1)
- Searching for a value: O(n)

<br>

## Doubly Linked List

- Accessing the head: O(1)
- Accessing the tail: O(1)
- Accessing the middle: O(n)
- Inserting / Removing the head: O(1)
- Inserting / Removing the tail: O(1)
- Inserting / Removing a middle node: O(n) to access + O(1)
- Searching for a value: O(n)