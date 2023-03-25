---
title: "Java memory models"
categories:
  - Blog
tags:
  - Modularity
  - Cohesion
  - Software Architecture
---

### Java Memory Management 

In this tutorial, we will delve into Java's memory models: Stack Memory and Heap Space. We will first discuss their main characteristics, followed by an explanation of how they are stored in RAM and their appropriate use cases. Finally, we will outline the key differences between the two memory models.

### Stack Memory in Java
Java's Stack Memory is utilized for static memory allocation and thread execution. It holds primitive values specific to a method and object references that originate from the method and reside in the heap.
Stack Memory follows a Last-In-First-Out (LIFO) access order. When a new method is called, a new block is added to the stack's top, containing values specific to that method, such as primitive variables and object references.

Upon completion of the method execution, the corresponding stack frame is removed, control returns to the calling method, and space becomes available for subsequent methods.

#### 1.1. Key Features of Stack Memory
Additional features of stack memory include:

* Expansion and contraction occur as methods are called and returned, respectively.
* Variables within the stack exist only for the duration of the method that created them.
* Memory allocation and deallocation occur automatically when a method completes its execution.
* If this memory becomes full, Java will throw a java.lang.StackOverFlowError.
* Access to this memory is faster compared to heap memory.
* Stack Memory is thread-safe, as each thread operates within its own stack area that stores local variables and function call information. Each thread in Java has its own stack, and it is automatically managed by the JVM. The stack is used for short-lived data and supports fast memory allocation and deallocation.


### Heap Space in Java
Heap space is utilized for dynamic memory allocation of Java objects and JRE classes during runtime. New objects are created within the heap space, while their references are stored in stack memory.

These objects have global access, allowing them to be accessed from anywhere within the application.

Heap space can be divided into smaller sections, known as generations, which include:

* **Young Generation** - The Young Generation, also known as the "New Generation," is where most new objects are initially allocated. The Young Generation is further subdivided into three spaces:
  * Eden Space: When an object is created, it is first placed in the Eden Space. This space is meant for short-lived objects that will be quickly garbage collected.
  * Survivor Space (S0 and S1): When objects in the Eden Space survive a garbage collection, they are moved to one of the two Survivor Spaces (S0 or S1). The JVM uses these spaces to keep objects that have survived a few garbage collection cycles but are not yet considered long-lived objects.
  
* **Old Generation (Tenured Generation)** - Objects that have survived multiple garbage collection cycles in the Young Generation are promoted to the Old Generation. This space is for objects that have a longer life cycle and are not frequently garbage collected. A garbage collection event in the Old Generation is known as a "major" or "full" garbage collection and is generally more time-consuming than a garbage collection in the Young Generation.

* **Permanent Generation (removed in Java 8) / Metaspace (introduced in Java 8)** - This generation is used to store metadata related to the JVM itself, such as class definitions and method data. In Java 8, the Permanent Generation was replaced with the Metaspace, which uses native memory instead of the Java heap to store metadata. This change was implemented to avoid issues like the OutOfMemoryError: PermGen space error and to allow for better control over metadata memory usage.

#### 3.1. Key Features of Java Heap Memory
Additional features of heap space include:

* Access to this memory involves complex memory management techniques, including the Young Generation, Old or Tenured Generation, and Permanent Generation.
* When heap space is full, Java throws a java.lang.OutOfMemoryError.
* Access to heap memory is relatively slower compared to stack memory.
* Memory deallocation in heap space is not automatic, requiring the Garbage Collector to free up unused objects to maintain memory usage efficiency.
* Unlike stack memory, heap space is not thread-safe and must be protected through proper code synchronization.

<br>

#### Example

![img]({{site.url}}/assets/blog_images/2023-03-25-java-memory-managment/java-heap-stack-diagram.png)
