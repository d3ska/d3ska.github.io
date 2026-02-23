---
title: "Latency And Throughput"
categories:
  - System Design
tags:
  - Latency and Throughput
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