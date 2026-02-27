---
title: "Replication And Sharding"
categories:
  - System Design
tags:
  - Databases
  - Replication
  - Sharding
  - Scalability
  - Distributed Systems
---

The performance of a system often depends on the efficiency of its database. By optimizing the database, the system's performance can be significantly improved. In this article, we will discuss how data redundancy and data partitioning techniques can be employed to enhance a system's fault tolerance, throughput, and overall reliability, with a focus on real-world scenarios and the role of load balancers.

### Replication

Replication involves duplicating data from one database server to others, using methods such as master-slave or master-master replication. This can be used to increase system redundancy and tolerate regional failures, for example. Replication can also be utilized to move data closer to clients, thus reducing the latency of accessing specific data.

#### Keeping replicas up-to-date

There are two ways to keep replicas updated: synchronous and asynchronous. Both come with pros and cons that should be considered on a case-by-case basis. Synchronous updates emphasize strong consistency but come at the cost of lower availability and higher latency. Asynchronous updates, on the other hand, offer higher availability and lower latency but may result in eventual consistency. For a deeper understanding of the trade-offs between these methods, refer to an article discussing the CAP Theorem.



### Sharding

Also known as data partitioning, sharding involves splitting a database into two or more pieces called shards. This is typically done to increase the throughput of your database.

#### Sharding Strategies

There are several well-established strategies for distributing data across shards:

* **Range-based sharding** -- partitions data by contiguous ranges of a key (e.g., users A-M on shard 1, N-Z on shard 2). This approach supports efficient range queries but can lead to uneven load if the data distribution is skewed.
* **Hash-based sharding** -- applies a hash function to a chosen key and assigns the result to a shard. This produces a more uniform distribution but makes range queries across shards expensive.
* **Directory-based sharding** -- maintains a lookup table that maps each key to its shard. This offers maximum flexibility (any key can go anywhere) at the cost of the lookup table becoming a single point of failure and a potential bottleneck.
* **Geographic sharding** -- partitions data based on a client's region, keeping data close to the users who access it most frequently.
* **Functional sharding** -- splits data based on the type of data being stored (e.g., user data is stored in one shard, payment data in another).


#### Hot Spot and Load Balancers

When distributing a workload across a set of servers, the workload might be spread unevenly. This can occur if your sharding key or hashing function is suboptimal or if your workload is naturally skewed. As a result, some servers will receive significantly more traffic than others, creating a "hot spot."

Load balancers can be used to distribute traffic across replicas or shards, helping to mitigate hot spots and ensure a more even distribution of requests. They can automatically direct traffic to the least-loaded server or use other algorithms to optimize resource utilization.



### Sharding in Practice

Sharding focuses on partitioning the database into smaller segments (shards) that can be distributed across multiple servers. This allows for horizontal scaling and increases the overall throughput of the system, enabling it to handle a higher volume of concurrent requests. Sharding can also help to reduce contention and resource utilization on individual servers.

Consider an e-commerce platform as an example: sharding can be employed to split the database based on product categories, user locations, or other factors. This partitioning approach enables the platform to handle a higher volume of concurrent requests, as each server only manages a subset of the data. This is particularly beneficial for write-heavy applications or scenarios where data growth requires efficient distribution and management.



### Replication vs Sharding: A Comparison

Large-scale web applications, e-commerce platforms, and social media networks often require both replication and sharding to maintain performance and reliability. While they serve different purposes, they complement each other to create a more robust and efficient system.

Here is a summary of the main differences:


* **Data Redundancy**: Replication creates multiple identical copies of the entire database, while sharding divides the database into smaller, unique partitions.

* **Fault Tolerance**: Replication offers higher fault tolerance, as each replica can act as a backup in case of a server failure. Sharding, on its own, does not provide this level of fault tolerance. However, combining sharding with replication within each shard can provide similar redundancy.

* **Read Throughput**: Replication improves read throughput by distributing read operations across multiple replicas, reducing the load on individual servers. Sharding does not directly improve read throughput, as each shard contains only a portion of the data.

* **Write Performance and Scalability**: Sharding enhances write performance and scalability by distributing the data and workload across multiple servers. Replication, on the other hand, may introduce performance overhead for write-heavy applications, as each write operation must be propagated to all replicas to maintain consistency.



### Conclusion

In practice, the decision between replication and sharding comes down to your system's dominant bottleneck:

* **Choose replication** when your primary concerns are fault tolerance, high availability, and read-heavy workloads. Replication is simpler to set up and reason about, and it keeps your operational complexity low.
* **Choose sharding** when you are hitting the storage or write-throughput limits of a single machine. Sharding unlocks horizontal scaling but introduces complexity around cross-shard queries, rebalancing, and data locality.
* **Combine both** when your system demands high availability *and* horizontal write scalability -- which is the norm for large-scale production systems. A common pattern is to shard the database for write distribution and then replicate each shard for redundancy, so that losing a single node does not cause data loss or downtime.

The key takeaway is that replication and sharding are not competing techniques. They address orthogonal concerns -- data redundancy versus data distribution -- and the most resilient architectures leverage both in tandem.