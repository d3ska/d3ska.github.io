---
title: "Latency And Throughput"
categories:
  - Blog
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
- Reading 1 MB from RAM: 250 μs (0.25 ms)
- Reading 1 MB from SSD: 1,000 μs (1ms)
- Transfer 1 MB over Network (1 Gbps): 10,000 μs (10ms)
- Reading 1MB from HDD: 20,000 μs (20ms)
- Inter-Continental Round Trip: 150,000 μs (150ms) <br>
  for packet from CA->Netherlands->CA


<br>

### Throughput 

In simpler terms, throughput refers to the amount of work a machine can perform within a specific time frame. Generally, when discussing throughput, we focus on the volume of data that can be transferred from one point in a system to another within a certain period.

Throughput is commonly measured in units like gigabytes per second, kilobytes per second, or megabytes per second. For example, a server's throughput can often be gauged in terms of requests per second (RPS or QPS).

You can think of throughput as a bottleneck, where only a certain number of bytes can be accommodated within a given time frame, such as seconds. This capacity limitation is essentially what throughput represents.


<br>

### Summarization

It is important to note that although latency and throughput are closely related and crucial for measuring a system's performance, they are not necessarily correlated. For example, a system, or specific components within it, might have very low latency, meaning they support rapid data transfers. However, there might be parts of the system with low throughput, which could negate the benefits of those low-latency operations.

In essence, we cannot make assumptions about latency or throughput based solely on the other. They are not inherently correlated factors.