---
title: "Stacks and Queues"
categories:
  - Blog
tags:
  - Data Structures
  - Complexity Analysis
  - Stacks
  - Queues
---

#### Stacks

A stack is an array-like data structure whose elements follow <br>**L**ast **I**n **F**irst **O**ut principle (**LIFO**).
It's really easy to visualize because we faced which such structures on a daily basis, you can compare it to plates in a sink, the last that was put in the sink is first that would be washed.

The following are a stack's standard operations and their corresponding complexities:
* Pushing an element onto the stack: O(1)
* Popping an element off the stack: O(1)
* Peeking at the element on the top of the stack: O(1)
* Searching for an element in the stack: O(n)

Here is a simple example of push and pop operations:

```java
Stack<Integer> stack = new Stack<>();

// Push elements onto the stack
stack.push(10);  // stack: [10]
stack.push(20);  // stack: [10, 20]
stack.push(30);  // stack: [10, 20, 30]

// Pop the top element (LIFO - returns 30)
int top = stack.pop();  // top = 30, stack: [10, 20]

// Peek at the top without removing it
int peeked = stack.peek();  // peeked = 20, stack: [10, 20]
```

A stack is typically based on [**dynamic array**](https://matthewonsoftware.com/blog/arrays/) or with a [**single linked list**](https://matthewonsoftware.com/blog/linked-lists/#single-linked-list).

**Real-world use cases:** The function call stack is probably the most well-known example -- every time a function calls another function, the current context is pushed onto the call stack and popped when the called function returns. Stacks also power undo functionality in text editors, expression evaluation in compilers and calculators, balanced-parentheses checking, and the back button in web browsers (each visited page is pushed onto a history stack).


#### Queues

A queue is an array-like data structure whose elements follow <br>**F**irst **I**n **F**irst **O**ut principle (**FIFO**).

A queue, on the other hand, may be compared to a group of people standing in line to purchase a ticket - the first person to get in the line is the first one to purchase a ticket and to get out of the queue.

The following are a queue's standard operations and their corresponding complexities:
* Enqueuing an element into the queue: O(1)
* Dequeueing an element out of the queue: O(1)
* Peeking at the element at the front of the queue: O(1)
* Searching for an element in the queue: O(n)

Here is a simple example of enqueue and dequeue operations:

```java
Queue<Integer> queue = new LinkedList<>();

// Enqueue elements
queue.add(10);  // queue: [10]
queue.add(20);  // queue: [10, 20]
queue.add(30);  // queue: [10, 20, 30]

// Dequeue the front element (FIFO - returns 10)
int front = queue.poll();  // front = 10, queue: [20, 30]

// Peek at the front without removing it
int peeked = queue.peek();  // peeked = 20, queue: [20, 30]
```

A queue is typically implemented with a [**doubly linked list**](https://matthewonsoftware.com/blog/linked-lists/#doubly-linked-list).

**Real-world use cases:** Queues appear everywhere in systems that need fair, ordered processing. Task schedulers use queues to run jobs in the order they were submitted. Breadth-first search (BFS) in graph algorithms relies on a queue to explore nodes level by level. Print spoolers queue up documents so they print in order. Message queues (like RabbitMQ or Kafka) decouple producers and consumers in distributed systems.

#### Deque (Double-Ended Queue)

A **deque** (pronounced "deck") generalizes both stacks and queues by allowing insertion and removal at both the front and the back in O(1) time. You can use a deque as a stack (push/pop at one end) or as a queue (enqueue at one end, dequeue at the other) -- or both simultaneously. In Java, `ArrayDeque` is the recommended general-purpose implementation, and in Python, `collections.deque` serves the same role.
