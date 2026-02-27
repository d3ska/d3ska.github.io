---
title: "Linked Lists: Singly and Doubly Linked"
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

<br>

### Implementing Insertion and Deletion

Now that we know the complexity characteristics, let's look at actual Java implementations using the `Node` class we defined earlier. Each of these methods is a building block you will reach for constantly when working with linked lists.

**insertAtHead** creates a new node and makes it the new head. Because we already have a reference to the head, there is no traversal involved — we simply wire the new node's `next` pointer to the current head and return it. This runs in O(1) time.

```java
public static Node insertAtHead(Node head, int value) {
    Node newNode = new Node(value);
    newNode.next = head;
    return newNode;
}
```

**insertAtTail** appends a new node at the end. We have to walk the entire list to find the last node (the one whose `next` is `null`), so the time complexity is O(n).

```java
public static Node insertAtTail(Node head, int value) {
    Node newNode = new Node(value);
    if (head == null) return newNode;
    Node current = head;
    while (current.next != null) {
        current = current.next;
    }
    current.next = newNode;
    return head;
}
```

**deleteByValue** removes the first node that holds a given value. The special case is when the head itself is the target — we just return `head.next`. Otherwise, we traverse until `current.next` holds the value, then bypass it. This is O(n) because of the traversal.

```java
public static Node deleteByValue(Node head, int value) {
    if (head == null) return null;
    if (head.value == value) return head.next;
    Node current = head;
    while (current.next != null && current.next.value != value) {
        current = current.next;
    }
    if (current.next != null) {
        current.next = current.next.next;
    }
    return head;
}
```

**insertAfter** inserts a new node right after a given node reference. Since we already have the reference, no searching is required — we create a new node, point it to `node.next`, and update `node.next` to the new node. O(1) time.

```java
public static void insertAfter(Node node, int value) {
    if (node == null) return;
    Node newNode = new Node(value);
    newNode.next = node.next;
    node.next = newNode;
}
```

These four operations cover the vast majority of linked list manipulation you will encounter. Notice the pattern: any operation where you already have a reference to the target position is O(1), while anything that requires finding a position first is O(n).

<br>

### Circular Linked Lists

A **circular linked list** is a variation where the last node's `next` pointer points back to the head instead of `null`, forming a continuous loop. This structure is useful in scenarios like **round-robin scheduling**, where a process scheduler cycles through tasks in order and, upon reaching the end, wraps back to the beginning seamlessly.

Here is a method that inserts a new node at the end of a circular linked list:

```java
public static Node insertAtEndCircular(Node head, int value) {
    Node newNode = new Node(value);
    if (head == null) {
        newNode.next = newNode;
        return newNode;
    }
    Node current = head;
    while (current.next != head) {
        current = current.next;
    }
    current.next = newNode;
    newNode.next = head;
    return head;
}
```

And here is how you traverse and print every value in a circular linked list:

```java
public static void traverseCircular(Node head) {
    if (head == null) return;
    Node current = head;
    do {
        System.out.print(current.value + " ");
        current = current.next;
    } while (current != head);
    System.out.println();
}
```

The key difference from a regular linked list is the termination condition: instead of checking whether the next pointer is `null`, we check whether we have returned to the starting node. Getting this wrong is a classic source of infinite loops, so always be deliberate about your stopping condition when working with circular structures.

<br>

### Common Linked List Problems

These three problems show up in interviews over and over again. They are worth memorizing not just for their own sake, but because the techniques they use — two pointers, in-place reversal, and fast/slow traversal — are the building blocks for dozens of harder problems.

**Floyd's Cycle Detection** (also known as the tortoise and hare algorithm) determines whether a linked list contains a cycle. Two pointers start at the head: `slow` advances one step at a time, while `fast` advances two steps. If a cycle exists, the fast pointer will eventually lap the slow pointer and they will meet. If `fast` reaches `null`, there is no cycle.

```java
public static boolean hasCycle(Node head) {
    Node slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

This runs in O(n) time and O(1) space — no extra data structures needed.

**Reversing a linked list in place** is the second essential technique. We iterate through the list with three pointers: `prev`, `current`, and `next`. At each step we save the next node, reverse the current node's pointer to point backward, then advance all three pointers forward.

```java
public static Node reverse(Node head) {
    Node prev = null;
    Node current = head;
    while (current != null) {
        Node next = current.next;
        current.next = prev;
        prev = current;
        current = next;
    }
    return prev;
}
```

Again, O(n) time and O(1) space. The trick is keeping track of `prev` so you do not lose the already-reversed portion of the list.

**Finding the middle node** reuses the two-pointer technique from cycle detection. `slow` moves one step, `fast` moves two. When `fast` reaches the end of the list, `slow` is sitting right at the middle.

```java
public static Node findMiddle(Node head) {
    if (head == null) return null;
    Node slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

O(n) time, O(1) space — one pass through the list is all it takes.

These three problems form the core toolkit for linked list interview questions. Variations of them — detecting the start of a cycle, reversing in groups of k, finding the nth node from the end — appear in many more complex problems, and they all build on the same pointer manipulation ideas shown here.

For a comparison of linked lists with their closest alternative, see [Arrays](https://matthewonsoftware.com/blog/arrays/).
