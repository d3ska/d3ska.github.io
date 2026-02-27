---
title: "Java Memory Management"
categories:
  - Blog
tags:
  - Memory models
  - JVM
---

### Java Memory Management

In this tutorial, we will delve into Java's memory models: Stack Memory and Heap Space. We will first discuss their main characteristics, followed by an explanation of how they are stored in RAM and their appropriate use cases. Finally, we will outline the key differences between the two memory models.

### Stack Memory in Java
Java's Stack Memory is utilized for storing local variables and method call frames during thread execution. It holds primitive values specific to a method and object references that originate from the method and point to objects residing in the heap.
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

* Access to this memory involves complex memory management techniques, including the Young Generation, Old or Tenured Generation, and Metaspace (Permanent Generation in Java 7 and earlier).
* When heap space is full, Java throws a java.lang.OutOfMemoryError.
* Access to heap memory is relatively slower compared to stack memory.
* Memory deallocation in heap space is not automatic, requiring the Garbage Collector to free up unused objects to maintain memory usage efficiency.
* Unlike stack memory, heap space is not thread-safe and must be protected through proper code synchronization.

<br>

#### Example

![img]({{site.url}}/assets/blog_images/2023-03-25-java-memory-management/java-heap-stack-management-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-03-25-java-memory-management/java-heap-stack-management-dark.png){: .dark }

### JVM Memory Tuning Flags

The JVM provides several flags to control memory allocation:

* **`-Xms`** -- Sets the initial heap size (e.g., `-Xms256m`). Setting this equal to `-Xmx` avoids heap resizing at runtime.
* **`-Xmx`** -- Sets the maximum heap size (e.g., `-Xmx2g`). If the application exceeds this limit, an `OutOfMemoryError` is thrown.
* **`-Xss`** -- Sets the stack size per thread (e.g., `-Xss512k`). Increase this if your application uses deep recursion and encounters `StackOverflowError`.

### Garbage Collection Algorithms

The JVM ships with several garbage collection algorithms, each optimized for different workloads:

* **Serial GC** (`-XX:+UseSerialGC`) -- A single-threaded collector suited for small applications with low memory footprints.
* **Parallel GC** (`-XX:+UseParallelGC`) -- Uses multiple threads for garbage collection in the Young Generation, optimizing throughput for multi-core systems.
* **G1 GC** (`-XX:+UseG1GC`) -- The default collector since Java 9. G1 divides the heap into equal-sized regions and collects the regions with the most garbage first, targeting both low pause times and high throughput. It is a solid general-purpose choice for most applications.
* **ZGC** (`-XX:+UseZGC`) -- A low-latency collector designed to keep pause times under a few milliseconds, regardless of heap size. ZGC can handle multi-terabyte heaps and became production-ready in Java 15. It is ideal for latency-sensitive applications.
* **Shenandoah GC** (`-XX:+UseShenandoahGC`) -- Another low-pause-time collector that performs concurrent compaction. Available in OpenJDK builds, it shares similar goals with ZGC but uses a different implementation approach.

Choosing the right GC algorithm depends on the application's requirements -- throughput-oriented workloads may benefit from Parallel GC, while latency-critical services should consider G1, ZGC, or Shenandoah.
