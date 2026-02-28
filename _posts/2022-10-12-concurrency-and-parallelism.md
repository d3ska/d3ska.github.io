---
title: "Concurrency, Threading and Parallelism"
categories:
  - Concurrency
tags:
  - Concurrency
  - Parallelism
  - Multithreading
---

Concurrency and parallelism are often confused, but they describe fundamentally different things. Parallelism is about doing multiple things at the same time. Concurrency is about structuring a program to deal with multiple things at once, even if only one actually runs at any given moment.

>"Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once."
>
>Rob Pike, [Concurrency Is Not Parallelism](https://go.dev/blog/waza-talk)

### Processes and Threads

A **process** is a running program with its own isolated memory space. The operating system keeps processes separated from each other, so one process cannot directly read or write another's memory.

A **thread** is a unit of execution within a process. Multiple threads within the same process share the same memory space, which makes communication between them fast but also introduces the risk of data corruption when two threads modify the same data simultaneously.

Every Java application starts with a single thread (the main thread) and can create additional threads as needed.

### Parallelism

![img]({{site.url}}/assets/blog_images/2022-10-12-concurrency-and-parallelism/cpu-visualization-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2022-10-12-concurrency-and-parallelism/cpu-visualization-dark.png){: .dark }

**Parallelism** means that multiple operations genuinely run at the same time on separate CPU cores. A processor with four cores can execute four threads truly simultaneously. Each core runs at a certain clock speed (e.g., 2.6 GHz means 2.6 billion clock cycles per second), which determines how quickly it processes instructions.

Modern CPUs often support hyper-threading (SMT), which allows each physical core to handle two hardware threads. This can improve throughput for certain workloads, but it is not the same as having twice the cores. True parallelism is still limited by the number of physical cores.

### Concurrency

![img]({{site.url}}/assets/blog_images/2022-10-12-concurrency-and-parallelism/concurrency-vs-parallelism-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2022-10-12-concurrency-and-parallelism/concurrency-vs-parallelism-dark.png){: .dark }

**Concurrency** means structuring work so that multiple threads can make progress, even if they don't run simultaneously. On a single core, the operating system's scheduler switches between threads rapidly, a process called **context switching**. Each thread gets a time slice, and the switching happens so fast that it creates the illusion of simultaneous execution.

A thread might also voluntarily yield the CPU by sleeping, waiting for I/O, or blocking on a lock. When that happens, the scheduler picks another runnable thread to fill the gap.

Here is a simplified view of how a scheduler might interleave two threads on a single core:

```
Time   Core 1           Event
────   ───────────────   ──────────────────────────────
 t0    Thread A: work    A starts running
 t1    Thread B: work    A blocks on I/O, scheduler picks B
 t2    Thread A: work    B blocks on I/O, scheduler picks A
 t3    Thread B: work    A blocks again, scheduler picks B
 t4    Thread A: done    A gets CPU back, finishes
 t5    Thread B: done    B gets CPU back, finishes
```

Neither thread ran in parallel. They shared the same core. But because the scheduler interleaved their execution, both made progress. This is the essence of concurrency.

### Concurrency in Java

In Java, creating and running threads looks like this:

```java
public class ConcurrencyExample {

    public static void main(String[] args) {
        Thread threadA = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                System.out.println("Thread A - step " + i);
                sleep(100);
            }
        });

        Thread threadB = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                System.out.println("Thread B - step " + i);
                sleep(100);
            }
        });

        threadA.start();
        threadB.start();
    }

    private static void sleep(long millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

`Thread.sleep` throws a checked `InterruptedException`, so it must be wrapped in a try-catch. The helper method keeps the thread lambdas readable. Even on a single core, the output of Thread A and Thread B will interleave because the scheduler switches between them whenever one sleeps.

### Common Concurrency Hazards

Working with multiple threads introduces several well-known hazards:

- **Race condition.** Two or more threads access shared mutable state at the same time, and the outcome depends on the order of execution. For example, two threads incrementing the same counter without synchronization can lose updates.
- **Deadlock.** Two or more threads each hold a resource the other needs, and neither can proceed. Thread A holds Lock 1 and waits for Lock 2, while Thread B holds Lock 2 and waits for Lock 1. Both are stuck forever.
- **Starvation.** A thread is perpetually denied access to a resource it needs because other higher-priority threads keep consuming it. The starved thread is technically runnable but never gets scheduled.

Java provides several tools to address these hazards: the `synchronized` keyword and `volatile` fields for visibility and mutual exclusion, `java.util.concurrent` locks and data structures for more fine-grained control, and immutable objects that sidestep the problem entirely by eliminating shared mutable state.

> **Related post**: [Java Synchronized vs Volatile](/posts/java-synchronized-vs-volatile/)
