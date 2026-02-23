---
title: "Concurrency, Threading and Parallelism"
categories:
  - Blog
tags:
  - Concurrency
  - Parallelism
---

These two concepts are often misunderstood or used interchangeably, yet they are completely different.

>"In programming, concurrency is the composition of independently executing processes, while parallelism is the simultaneous execution of (possibly related) computations. Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once."
>
>Source: blog.golang.org

Let's take a look at a visualization of a processor with four cores.

![img]({{site.url}}/assets/blog_images/2022-10-12-concurrency-and-parallelism/cpu-visualization.png)


#### Parallelism

Parallelism means that multiple processes can genuinely run simultaneously because they execute on separate cores. In other words,
the number of cores your processor has is the number of operations that can happen at the exact same time.

On the image above we have four cores. That means at any point in time we can perform at most four operations at the exact
same time, and these operations are really low-level computation operations.

<br>
But as you can imagine, our computers do more than four operations. It is possible thanks to threads and multiprocessing.

<br>
<br>

#### Threads

A thread is essentially one program -- a set of operations that need to be processed.
Every thread is assigned to one core, so each core will have a bunch of threads that it
executes, switching between which ones it performs operations on.

Let's take a look at an analogy for concurrency and parallelism.
<br>

![img]({{site.url}}/assets/blog_images/2022-10-12-concurrency-and-parallelism/concurrency-vs-parallelism.png)

<br>
<br>

Concurrency means that many threads are waiting to be processed by a CPU core. However, when a thread is being processed, it does not necessarily run to completion. A thread may run on the same core and be paused many times before it finishes its job. Because the thread may block on I/O, sleep, or simply not need CPU time at that moment, the processor core can execute another thread while the previous one waits.

This is what **concurrency** means: not running things in parallel at the same time, but interleaving the execution of multiple threads. A single CPU core switches between threads in its execution chain, giving the illusion that they all make progress simultaneously.

Here is a simplified example of how a thread scheduler might interleave two threads on a single core:

```
Time -->
Core 1: [Thread A: work] [Thread B: work] [Thread A: work] [Thread B: work] [Thread A: done] [Thread B: done]
             |                 |                 |
         A runs            A blocks,          B blocks,
                          B gets CPU         A gets CPU
```

In Java, creating and running threads looks like this:

```java
public class SimpleThreadExample {

    public static void main(String[] args) {
        Thread threadA = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                System.out.println("Thread A - step " + i);
                Thread.sleep(100); // simulates work, yields CPU
            }
        });

        Thread threadB = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                System.out.println("Thread B - step " + i);
                Thread.sleep(100);
            }
        });

        threadA.start();
        threadB.start();
    }
}
```

Even on a single core, the output of Thread A and Thread B will interleave because the scheduler switches between them whenever one sleeps or yields.

#### Common Concurrency Hazards

Working with multiple threads introduces several well-known hazards:

- **Race condition** -- Two or more threads access shared mutable state at the same time, and the outcome depends on the order of execution. For example, two threads incrementing the same counter without synchronization can lose updates.
- **Deadlock** -- Two or more threads each hold a resource the other needs, and neither can proceed. Thread A holds Lock 1 and waits for Lock 2, while Thread B holds Lock 2 and waits for Lock 1 -- both are stuck forever.
- **Starvation** -- A thread is perpetually denied access to a resource it needs because other higher-priority threads keep consuming it. The starved thread is technically runnable but never gets scheduled.

Understanding these hazards is essential for writing correct concurrent code. Solutions include proper locking strategies, lock ordering, using concurrent data structures, and preferring immutable state wherever possible.

#### Clock speed

There is also a thing called clock speed of a processor, e.g. 2.6 GHz.
Each core in the processor runs at a speed of 2.6 GHz, which means each core can perform 2.6 billion clock cycles per second.
