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
  
#### Stack

Is an array-like data structure whose elements follow **L**ast **I**n **F**irst **O**ut principle. (**LIFO**)
It's really easy to visualize because we faced which such structures on a daily basis, you can compare it to a plates in a sink, the last that was put in the sink is first that would be washed. 

The following are a stack's standard operations and their corresponding complexities:
* Pushing an element onto the stack: O(1)
* Popping an element onto the stack: O(1)
* Peeking at the element on the top of the stack: O(1)
* Searching for an element in the stack: O(n)

A stack is typically based on [**dynamic array**](https://matthewonsoftware.com/blog/arrays/) or with a [**single linked list**](https://matthewonsoftware.com/blog/linked-lists/#single-linked-list).


#### Queues

Is an array-like data structure whose elements follow **F**irst **I**n **F**irst **O**ut principle. (**FIFO**)

A queues in the other hand may be compared to a group of people standing in line to purchase a ticket - the first person to get in the line is the first one to purchase ticket and to get out of the queue.

The following are a queue's standard operations and their corresponding complexities:
* Enqueuing an element into the queue: O(1)
* Dequeueing an element out of the queue: O(1)
* Peeking at the element at the front of the queue: O(1)
* Searching for an element in the queue: O(n)

A queue is typically implemented with a [**doubly linked list**](https://matthewonsoftware.com/blog/linked-lists/#doubly-linked-list).

