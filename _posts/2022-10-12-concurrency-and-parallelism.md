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

Let's take a look at real-life example:

![img]({{site.url}}/assets/blog_images/2022-10-12-concurrency-and-parallelism/concurrency-vs-parallelism.jpg")


In the case of concurrency, there are many threads that are waiting for some CPU time, still, when the thread is processed by the CPU core, it doesn't mean that it will finish. I mean it may be consumed many times before it finished its job.
It's because the CPU is optimized and cores are quickly switching between threads.

We can visualize the CPU Unit as below:

![img]({{site.url}}/assets/blog_images/2022-10-12-concurrency-and-parallelism/cpu-visualization.jpg")


Parallelism, however, means that some processes can genuinely run simultaneously as they run on separate cores