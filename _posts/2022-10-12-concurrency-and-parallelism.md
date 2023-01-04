---
title: "Concurrency vs parallelism"
categories:
  - Blog
tags:
  - Concurrency
  - Parallelism
---

Those two concepts are often misunderstood or used interchangeably, where they are completely different concepts.

>“In programming, concurrency is the composition of independently executing processes, while parallelism is the simultaneous execution of (possibly related) computations. Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once.”
>
>Source: blog.golang.org

Let's take a look on visualization of processor with four cores.

![img]({{site.url}}/assets/blog_images/2022-10-12-concurrency-and-parallelism/cpu-visualization.jpg")


#### Parallelism

Means that some processes can genuinely run simultaneously as they run on separate cores, in other words
the amount of cores you have on your processor is the amount of things that can happen at the exact the same time.

So as on the image above we have four cores, that means at any point in time we can do at most four operations at the exact
same time and these operations are really low-level computation operations.

<br>

But as you can imagine, our computer do more than four operations... Its possible thanks to threads and multiprocessing.

<br>
<br>

#### Threads

Is essentially a one program, a set of operations that need to be processed.  
Every thread is going to be assigned to one core, so each of cores will have a bunch of threads that they're 
going to be executing and switching between which ones they perform operations on.

Let's take a look on analogy to concurrency and parallelism.
<br>

![img]({{site.url}}/assets/blog_images/2022-10-12-concurrency-and-parallelism/concurrency-vs-parallelism.jpg")

<br>
<br>

Concurrency means that many threads are waiting to be processed by the CPU core, however, when the thread is being processed by the core, it doesn't mean that it will run until it finishes.
I mean it may run on the same core and stop many times before it finished its job. Because the thread may hang, stop, or basically doesn't need to be executed at the current time, the processor core can execute another one when the previous one is waiting.
And it describes **concurrent** programming, not when things run in parallel at the same time but when we're doing things in different timing sequences, so we can have multiple threads running at the same time and our CPU core
that we're running their threads on is switching between these threads in its execution chain.


#### Clock speed

There is also thing called clock speed of processor, e.g. 2.6-Ghz.
It means that each core in processor can run with speed of 2.6-Ghz, which basically mean that each core  can run 2.6 billion instructions in a second.