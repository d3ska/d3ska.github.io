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

#### Choosing the Right GC

For most web applications and microservices, **G1 GC** is the right default. It ships as the JVM default since Java 9, handles heaps from a few hundred megabytes to several gigabytes well, and balances throughput with reasonable pause times. Start here unless you have a specific reason not to.

Reach for **ZGC** only if your application has strict latency requirements (sub-millisecond pauses) or operates on very large heaps (tens of gigabytes or more). Trading applications, real-time bidding systems, and interactive services where tail latency matters are good candidates. **Shenandoah** is a viable alternative to ZGC with similar goals. It is available in most OpenJDK distributions and worth benchmarking against ZGC for your specific workload.

**Serial GC** is useful for small heaps (under ~100MB) and containerized applications with limited CPU. If your service runs in a container with a single vCPU, Serial GC may actually outperform the parallel collectors because there is no coordination overhead.

**Parallel GC** remains a good choice for batch processing jobs and offline data pipelines where throughput matters more than pause times.

### Reference Types

Java provides special reference types in `java.lang.ref` that give you more control over how the garbage collector treats objects:

* **SoftReference**: the GC will keep the referenced object alive as long as there is enough memory, but will clear it before throwing an `OutOfMemoryError`. This makes it useful for memory-sensitive caches where you want to hold on to data if possible but can afford to recompute it.
* **WeakReference**: the GC can reclaim the referenced object at the next collection cycle, regardless of memory pressure. `WeakHashMap` uses this internally to allow entries to be garbage collected when their keys are no longer strongly referenced elsewhere.
* **PhantomReference**: the referenced object is already finalized and about to be reclaimed. You cannot retrieve the object through a `PhantomReference`. It is used together with a `ReferenceQueue` to perform cleanup actions (releasing native resources, for example) after the object is collected. This is the modern alternative to overriding `finalize()`.

### Common Memory Leak Patterns

Java's garbage collector handles deallocation, but that does not mean memory leaks are impossible. They just look different. A "leak" in Java typically means objects that are still reachable (so the GC cannot collect them) but are no longer useful to the application. A few patterns come up repeatedly:

* **Static collections that grow unbounded**: a `static List` or `static Map` that accumulates entries over time without any eviction. Because the collection itself is reachable from a class (which is reachable from the classloader), nothing in it will ever be garbage collected.
* **Unclosed resources**: streams, database connections, and HTTP clients that are opened but never closed. These often hold native memory or OS handles outside the heap. Always use try-with-resources.
* **Non-static inner classes holding outer class references**: an inner class instance implicitly holds a reference to its enclosing object. If the inner class instance outlives the outer object (for example, it is stored in a long-lived collection), the outer object and everything it references cannot be collected. Making the inner class `static` breaks this chain.
* **ThreadLocal not cleaned up**: `ThreadLocal` values are tied to the thread's lifecycle. In application servers and thread pools where threads are reused, a `ThreadLocal` that is set but never removed can hold on to objects indefinitely.

### Diagnostic Tools

When something goes wrong with memory, you need visibility into what the JVM is actually doing. A few tools are worth knowing:

**jcmd** is a lightweight command-line tool bundled with the JDK. It can query a running JVM without adding significant overhead:

```bash
# Print heap usage summary (generation sizes, occupancy)
jcmd <pid> GC.heap_info

# Trigger a heap dump for offline analysis
jcmd <pid> GC.heap_dump /tmp/heapdump.hprof

# Print class histogram (top memory consumers)
jcmd <pid> GC.class_histogram
```

**VisualVM** provides a graphical view of heap usage, thread activity, and GC behavior over time. It can connect to a running JVM locally or remotely and is useful for identifying memory trends during development or load testing.

For analyzing heap dumps (the `.hprof` files produced by `jcmd` or triggered automatically on `OutOfMemoryError` with `-XX:+HeapDumpOnOutOfMemoryError`), **Eclipse MAT** (Memory Analyzer Tool) is the standard. It can parse multi-gigabyte dumps and pinpoint which objects are retaining the most memory through its "dominator tree" and "leak suspects" reports.

> **Related posts**: [Is Java Pass by Value or by Reference?](/posts/is-java-pass-by-value-or-by-reference/), [Concurrency and Parallelism](/posts/concurrency-and-parallelism/), [Memory](/posts/memory/)
