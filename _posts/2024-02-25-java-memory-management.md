---
title: "Java Memory Management"
categories:
  - Java
tags:
  - Memory Management
  - JVM
  - Java
  - Heap
  - Stack
  - Garbage Collection
---

Every Java application uses two primary memory areas: the **stack** and the **heap**. Understanding how they work and how objects move between them is essential for writing efficient code and diagnosing memory issues.

### Stack Memory

Stack memory stores local variables and method call frames during thread execution. It holds primitive values and object references (the references themselves, not the objects they point to, which live on the heap).

Stack memory follows a **Last-In-First-Out** (LIFO) order. When a method is called, a new frame is pushed onto the stack containing the method's local variables. When the method returns, the frame is popped and the memory is reclaimed immediately.

Key characteristics:

* Each thread has its own stack, so stack memory is inherently **thread-safe**.
* Allocation and deallocation are automatic and extremely fast (just moving a pointer).
* Variables exist only for the duration of the method that created them.
* If the stack runs out of space, the JVM throws `java.lang.StackOverflowError` (typically caused by unbounded recursion).
* Stack memory is much smaller than heap memory (default is usually 512KB to 1MB per thread).

### Heap Space

Heap space is used for dynamic memory allocation. All Java objects and their instance data live on the heap. Unlike stack memory, heap objects are not tied to a single method or thread. They persist until the garbage collector determines they are no longer reachable.

The heap is divided into **generations** to optimize garbage collection:

* **Young Generation**: where most new objects are allocated. It is subdivided into:
  * **Eden Space**: objects are first created here. Most objects are short-lived and get collected quickly.
  * **Survivor Spaces (S0 and S1)**: objects that survive a garbage collection in Eden are moved here. The JVM alternates between the two survivor spaces during each collection cycle.

* **Old Generation (Tenured)**: objects that survive multiple garbage collection cycles in the Young Generation are promoted here. Collections in the Old Generation ("major" or "full" GC) are less frequent but more time-consuming.

* **Metaspace** (replaced Permanent Generation in Java 8): stores class metadata, method definitions, and other JVM internals. Unlike the old PermGen, Metaspace uses native memory and grows dynamically, avoiding the `OutOfMemoryError: PermGen space` errors that plagued older Java versions.

Key characteristics:

* Heap memory is shared across all threads, so access to objects must be synchronized when multiple threads are involved.
* The garbage collector is responsible for freeing unreachable objects. Memory deallocation is not manual.
* If the heap runs out of space, the JVM throws `java.lang.OutOfMemoryError`.
* Heap access is slower than stack access due to the overhead of garbage collection and pointer indirection.

### Example: Stack and Heap in Action

```java
public class Person {
    private int id;
    private String name;

    public Person(int id, String name) {
        this.id = id;
        this.name = name;
    }
}

public class Main {
    public static void main(String[] args) {
        int id = 23;
        String name = "John";
        Person person = buildPerson(id, name);
    }

    static Person buildPerson(int id, String name) {
        return new Person(id, name);
    }
}
```

![img]({{site.url}}/assets/blog_images/2023-03-25-java-memory-management/java-heap-stack-management-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-03-25-java-memory-management/java-heap-stack-management-dark.png){: .dark }

When `main` calls `buildPerson`, a new stack frame is pushed with its own copies of `id` (a primitive, stored directly on the stack) and `name` (a reference, pointing to the `"John"` string on the heap). Inside `buildPerson`, the `Person` constructor creates another frame. The `new Person(...)` call allocates the `Person` object on the heap. All three stack frames hold references that point to the same `"John"` string in the string pool and the same `Person` object on the heap.

When `buildPerson` returns, its frame is popped. The `Person` object remains on the heap because it is still referenced by the `person` variable in `main`. Once `main` returns and no references to the `Person` object remain, it becomes eligible for garbage collection.

### JVM Memory Tuning Flags

The JVM provides several flags to control memory allocation:

* **`-Xms`**: sets the initial heap size (e.g., `-Xms256m`). Setting this equal to `-Xmx` avoids heap resizing at runtime.
* **`-Xmx`**: sets the maximum heap size (e.g., `-Xmx2g`). If the application exceeds this limit, an `OutOfMemoryError` is thrown.
* **`-Xss`**: sets the stack size per thread (e.g., `-Xss512k`). Increase this if your application uses deep recursion and encounters `StackOverflowError`.

### Garbage Collection Algorithms

The JVM ships with several garbage collection algorithms, each optimized for different workloads:

* **Serial GC** (`-XX:+UseSerialGC`): a single-threaded collector suited for small applications with low memory footprints.
* **Parallel GC** (`-XX:+UseParallelGC`): uses multiple threads for garbage collection in the Young Generation, optimizing throughput for multi-core systems.
* **G1 GC** (`-XX:+UseG1GC`): the default collector since Java 9. G1 divides the heap into equal-sized regions and collects the regions with the most garbage first, targeting both low pause times and high throughput. A solid general-purpose choice for most applications.
* **ZGC** (`-XX:+UseZGC`): a low-latency collector designed to keep pause times under a few milliseconds, regardless of heap size. ZGC can handle multi-terabyte heaps and became production-ready in Java 15. Ideal for latency-sensitive applications.
* **Shenandoah GC** (`-XX:+UseShenandoahGC`): another low-pause-time collector that performs concurrent compaction. Available in OpenJDK builds, it shares similar goals with ZGC but uses a different implementation approach.

Choosing the right GC depends on the application's requirements. Throughput-oriented workloads benefit from Parallel GC, while latency-critical services should consider G1, ZGC, or Shenandoah.
