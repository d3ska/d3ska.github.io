---
title: "Latency And Throughput"
categories:
  - System Design
tags:
  - Latency
  - Throughput
  - Performance
---

#### Latency and throughput are the two most important measures of the performance of a system. 

<br>

### Latency

Latency essentially measures the time it takes for data to travel through a system, specifically, the duration required for data to move from one point in the system to another.

When discussing latency, it can refer to various aspects within a system. For example, we might consider the latency of a network request. This would be the time it takes for a request to travel from a client to a server and then back from the server to the client.

Similarly, if you're reading data from memory or perhaps from a disk, the time it takes to access that data is also considered latency.

One crucial aspect of latency to understand is that different components in systems have varying latencies. There is a trade-off involved in the design of a system, as some elements will exhibit higher latencies, while others will have lower latencies.

For example, if you are: 
- **Reading 1 MB from RAM**: 250 μs (0.25 ms)
- **Reading 1 MB from SSD**: 1,000 μs (1 ms)
- **Transfer 1 MB over Network (1 Gbps)**: 10,000 μs (10 ms)
- **Reading 1 MB from HDD**: 20,000 μs (20 ms)
- **Inter-Continental Round Trip for packet**: 150,000 μs (150 ms) <br>

When measuring latency in production systems, looking at the average alone can be misleading. **Tail latency** -- the latency experienced by the slowest requests, typically measured at the 95th percentile (p95) or 99th percentile (p99) -- often reveals problems that averages hide. For example, a service might have a median latency of 5 ms but a p99 latency of 500 ms, meaning 1 in 100 requests takes 100 times longer than usual. Tail latency matters because at scale, nearly every user session will encounter at least one slow request. This is why SLA definitions frequently reference percentile-based latency targets (e.g., "p99 latency must remain below 200 ms") rather than averages.


<br>

### Throughput

In simpler terms, throughput refers to the amount of work a machine can perform within a specific time frame. Generally, when discussing throughput, we focus on the volume of data that can be transferred from one point in a system to another within a certain period.

Throughput is commonly measured in units like gigabytes per second, kilobytes per second, or megabytes per second. For example, a server's throughput can often be gauged in terms of requests per second (RPS or QPS).

You can think of throughput as a bottleneck, where only a certain number of bytes can be accommodated within a given time frame, such as seconds. This capacity limitation is essentially what throughput represents.

As a concrete example, consider a web server capable of handling 10,000 requests per second (10k RPS). That is its throughput. If traffic spikes to 15,000 RPS, the server becomes a bottleneck -- requests start queuing, latencies climb, and some requests may time out entirely. Increasing throughput (by scaling horizontally with more servers, for instance) relieves that bottleneck and allows the system to serve all incoming traffic.


<br>

### Summary

It is important to note that although latency and throughput are closely related and crucial for measuring a system's performance, they are not necessarily correlated. Knowing one tells you very little about the other, and optimizing for one does not automatically improve the other.

Consider two contrasting examples:
* **Batch processing system**: A nightly data pipeline might process terabytes of data, achieving extremely high throughput. Yet any individual record takes hours to flow through the entire pipeline, meaning latency is very high.
* **In-memory cache (e.g., Redis)**: A cache lookup returns results in microseconds, delivering exceptionally low latency. However, a single cache node has a finite number of connections and memory, so its throughput is limited compared to a distributed database cluster.

In essence, we cannot make assumptions about latency or throughput based solely on the other. They are independent dimensions of system performance that both deserve careful attention during design.

<br>

### Little's Law: Connecting Latency, Throughput, and Concurrency

While latency and throughput are independent measures, there is an elegant formula that ties them together with a third quantity -- concurrency. **Little's Law** states that **L = λ × W**, where **L** is the average number of concurrent requests in the system, **λ** (lambda) is throughput measured in requests per second, and **W** is the average latency in seconds per request. Despite its simplicity, this relationship is one of the most powerful tools in a system designer's toolkit.

Let's make it concrete. Imagine a server handling 100 requests per second (λ = 100 RPS) where each request takes 0.5 seconds to complete (W = 0.5s). Little's Law tells us that, on average, there are L = 100 × 0.5 = 50 requests being processed concurrently at any given moment. Now suppose you optimize your service and cut latency in half to 0.25 seconds. Concurrency drops to L = 100 × 0.25 = 25. You now need half as many threads or connections to sustain the exact same throughput -- freeing up resources and reducing memory pressure across the board.

This has direct implications for **capacity planning**. If your database connection pool maxes out at 50 connections and each query takes 100 ms on average, your maximum throughput is 50 / 0.1 = 500 queries per second. Want more throughput? You either need to reduce query latency or increase the pool size. Little's Law makes the trade-off explicit.

What makes the law especially useful is its universality. It holds regardless of request distribution, arrival patterns, or service discipline (FIFO, priority queue, round-robin -- it does not matter). Whether you are sizing a thread pool, provisioning database connections, or estimating the number of worker processes for a task queue, Little's Law gives you a reliable starting point.

<br>

### How to Improve Latency

Reducing latency is about shortening the time each individual operation takes to complete. Here are seven practical strategies that come up again and again in system design.

1. **Caching** -- Store computed results or frequently accessed data in memory so the system can return responses without repeating expensive computation or database lookups. Even a short-lived cache can eliminate a significant percentage of redundant work.

2. **CDNs** -- Serve static assets like images, scripts, and stylesheets from edge locations geographically closer to your users. Shaving off network distance directly reduces round-trip time.

3. **Query optimization** -- Add appropriate indexes, eliminate N+1 query patterns, and fetch only the columns and rows you actually need. A single missing index can turn a 2 ms query into a 2-second full table scan.

4. **Reduce round trips** -- Batch multiple small requests into a single call, use connection pooling to avoid repeated handshakes, or leverage HTTP/2 multiplexing to send several requests over one connection simultaneously.

5. **Faster storage** -- Move hot data from HDD to SSD, or from SSD to in-memory stores like Redis or Memcached. As the latency numbers earlier in this post show, the difference between storage tiers is measured in orders of magnitude.

6. **Profiling** -- Use flame graphs and distributed tracing tools to identify the actual bottleneck before you start optimizing. It is surprisingly common to spend weeks optimizing a component that accounts for only a fraction of total latency.

7. **Async processing** -- Offload non-critical work such as sending emails, generating reports, or updating analytics to background queues. The user-facing request returns immediately while the heavy lifting happens out of band.

One final reminder: optimizing the average is not enough. Focus on **tail latency** at p95 and p99 to ensure a consistent user experience. A fast median means little if one in every hundred requests takes ten seconds.

<br>

### How to Improve Throughput

Increasing throughput is about enabling the system to handle more total work in a given period. Here are seven strategies that are foundational to scaling.

1. **Horizontal scaling** -- Add more server instances behind a load balancer to distribute incoming requests across a larger fleet. This is often the most straightforward way to increase capacity when a single machine is no longer sufficient.

2. **Connection pooling** -- Reuse database and HTTP connections instead of creating and tearing down new ones for every request. Connection setup is expensive, and pooling amortizes that cost across many operations.

3. **Concurrency and parallelism** -- Use multi-threading, async I/O, or event-driven architectures to handle multiple requests simultaneously within a single process. A server blocked on one synchronous call wastes time that could be spent serving other requests.

4. **Message queues** -- Decouple producers from consumers so that traffic spikes are absorbed by the queue rather than overwhelming downstream services. The queue acts as a buffer, letting consumers process work at a sustainable rate.

5. **Sharding** -- Distribute data across multiple database instances so that no single node becomes a bottleneck. By partitioning on a key like user ID or region, read and write load is spread evenly.

6. **Resource monitoring** -- Identify whether CPU, memory, disk, or network is the constraining factor using metrics and alerts, then scale or optimize that specific resource. Throwing hardware at the wrong bottleneck wastes money and solves nothing.

7. **Rate limiting** -- Paradoxically, rate limiting can improve overall throughput by preventing resource exhaustion and cascading failures during traffic spikes. By shedding excess load gracefully, the system continues to serve the majority of requests instead of collapsing under pressure.

For related topics, see [Caching](https://matthewonsoftware.com/system%20design/caching/), [Load Balancers](https://matthewonsoftware.com/system%20design/load-balancers/), [Polling And Streaming](https://matthewonsoftware.com/system%20design/polling-and-streaming/), [Replication And Sharding](https://matthewonsoftware.com/system%20design/replication-and-sharding/), and [Rate Limiting](https://matthewonsoftware.com/system%20design/rate-limiting/).