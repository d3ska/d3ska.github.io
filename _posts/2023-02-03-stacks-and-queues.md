---
title: "Stacks and Queues"
categories:
  - Data Structures
tags:
  - Data Structures
  - Complexity Analysis
  - Stacks
  - Queues
  - LIFO
  - FIFO
---

Stacks and queues are among the simplest data structures, but they appear everywhere: from the function call stack that powers every program to the message queues that connect distributed systems. Both restrict how elements are added and removed, and that constraint is exactly what makes them useful.

### Stacks

A **stack** follows the **Last In, First Out** (LIFO) principle: the most recently added element is the first one removed. Think of a stack of plates: you always take the top plate first.

The standard operations and their time complexities:

* **Push** an element onto the stack: O(1)
* **Pop** an element off the stack: O(1)
* **Peek** at the top element without removing it: O(1)
* Searching for an element: O(n)

#### Implementation in Java

The `java.util.Stack` class exists in the JDK, but it is a legacy class that extends `Vector` and synchronizes every operation, making it unnecessarily slow for single-threaded use. The recommended replacement is `ArrayDeque`, which implements the `Deque` interface and provides the same push/pop/peek operations without the overhead:

```java
Deque<Integer> stack = new ArrayDeque<>();

// Push elements onto the stack
stack.push(10);  // stack: [10]
stack.push(20);  // stack: [10, 20]
stack.push(30);  // stack: [10, 20, 30]

// Pop the top element (LIFO: returns 30)
int top = stack.pop();  // top = 30, stack: [10, 20]

// Peek at the top without removing it
int peeked = stack.peek();  // peeked = 20, stack: [10, 20]
```

Under the hood, `ArrayDeque` uses a resizable array. A stack can also be implemented with a [singly linked list](/posts/linked-lists/#single-linked-list), where push and pop operate on the head node in O(1).

#### Real-World Use Cases

The **function call stack** is probably the most well-known example. Every time a method calls another method, the current context (local variables, return address) is pushed onto the call stack and popped when the called method returns. This is also why unbounded recursion causes a `StackOverflowError`.

Other common uses:

* **Undo/redo** in text editors: each action is pushed onto an undo stack; undoing pops it and pushes it onto a redo stack
* **Expression evaluation** in compilers and calculators: operands are pushed onto a stack and popped when an operator is encountered
* **Balanced parentheses checking**: push opening brackets, pop when a closing bracket is found, and verify they match
* **Browser back button**: each visited page is pushed onto a history stack

### Queues

A **queue** follows the **First In, First Out** (FIFO) principle: elements are removed in the order they were added. Think of a line at a ticket counter: the first person in line is the first one served.

The standard operations and their time complexities:

* **Enqueue** an element at the back: O(1)
* **Dequeue** an element from the front: O(1)
* **Peek** at the front element without removing it: O(1)
* Searching for an element: O(n)

#### Implementation in Java

In Java, `Queue` is an interface. The most common general-purpose implementation is `LinkedList`, which implements both `Queue` and `Deque`. For most use cases, `ArrayDeque` is a better choice (faster due to array-based storage and no node allocation overhead), unless you specifically need the linked list's ability to remove elements from the middle efficiently.

```java
Queue<Integer> queue = new ArrayDeque<>();

// Enqueue elements
queue.add(10);  // queue: [10]
queue.add(20);  // queue: [10, 20]
queue.add(30);  // queue: [10, 20, 30]

// Dequeue the front element (FIFO: returns 10)
int front = queue.poll();  // front = 10, queue: [20, 30]

// Peek at the front without removing it
int peeked = queue.peek();  // peeked = 20, queue: [20, 30]
```

The `Queue` interface offers two sets of methods for the same operations: one set throws an exception on failure (`add`, `remove`, `element`) and the other returns a special value (`offer`, `poll`, `peek`). In most cases, the special-value methods are preferred because they let you handle empty queues without exception handling.

#### Real-World Use Cases

* **Task scheduling**: operating systems use queues to schedule processes and threads in the order they become ready
* **Breadth-first search (BFS)**: graph traversal relies on a queue to explore nodes level by level
* **Print spoolers**: documents are printed in the order they were submitted
* **Message queues** (RabbitMQ, Kafka, SQS): decouple producers and consumers in distributed systems, buffering messages until the consumer is ready

### Priority Queues

A **priority queue** is a variant where elements are dequeued not in insertion order but by priority. The element with the highest (or lowest) priority is always at the front.

In Java, `PriorityQueue` implements this using a binary heap:

```java
PriorityQueue<Integer> pq = new PriorityQueue<>(); // min-heap by default

pq.add(30);
pq.add(10);
pq.add(20);

int first = pq.poll();  // 10 (smallest element, not the first inserted)
int second = pq.poll(); // 20
```

The time complexities differ from a regular queue: enqueue and dequeue are O(log n) because the heap must be maintained, while peek remains O(1).

Priority queues are used in Dijkstra's shortest-path algorithm, job scheduling with priorities, and any scenario where "most urgent first" ordering is needed.

### Deque (Double-Ended Queue)

A **deque** (pronounced "deck") generalizes both stacks and queues by allowing insertion and removal at both the front and the back in O(1) time. You can use a deque as a stack (push/pop at one end) or as a queue (enqueue at one end, dequeue at the other), or both simultaneously.

In Java, `ArrayDeque` is the recommended general-purpose implementation of the `Deque` interface. It outperforms both `Stack` (which is synchronized) and `LinkedList` (which allocates a node object per element) for stack and queue use cases.

```java
Deque<String> deque = new ArrayDeque<>();

deque.addFirst("A");  // deque: [A]
deque.addLast("B");   // deque: [A, B]
deque.addFirst("C");  // deque: [C, A, B]

String front = deque.removeFirst();  // "C"
String back  = deque.removeLast();   // "B"
```

### Comparison

| | Stack | Queue | Priority Queue | Deque |
|---|---|---|---|---|
| Order | LIFO | FIFO | By priority | Both ends |
| Insert | O(1) | O(1) | O(log n) | O(1) |
| Remove | O(1) | O(1) | O(log n) | O(1) |
| Peek | O(1) | O(1) | O(1) | O(1) |
| Java class | `ArrayDeque` | `ArrayDeque` | `PriorityQueue` | `ArrayDeque` |

> **Related posts:** [Arrays](/posts/arrays/), [Linked Lists](/posts/linked-lists/)
