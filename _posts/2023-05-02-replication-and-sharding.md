---
title: "Replication And Sharding"
categories:
  - System design
tags:
  - Databases
---

The performance of a system often depends on the efficiency of its database. By optimizing the database, the system's performance can be significantly improved. In this article, we will discuss how data redundancy and data partitioning techniques can be employed to enhance a system's fault tolerance, throughput, and overall reliability, with a focus on real-world scenarios and the role of load balancers.

### Replication

Replication involves duplicating data from one database server to others, using methods such as master-slave or master-master replication. This can be used to increase system redundancy and tolerate regional failures, for example. Replication can also be utilized to move data closer to clients, thus reducing the latency of accessing specific data.

#### Keeping replicas up-to-date

There are two ways to keep replicas updated: synchronous and asynchronous. Both come with pros and cons that should be considered on a case-by-case basis. Synchronous updates emphasize strong consistency but come at the cost of lower availability and higher latency. Asynchronous updates, on the other hand, offer higher availability and lower latency but may result in eventual consistency. For a deeper understanding of the trade-offs between these methods, refer to an article discussing the CAP Theorem.



### Sharding

Also known as data partitioning, sharding involves splitting a database into two or more pieces called shards. This is typically done to increase the throughput of your database. Popular sharding strategies include:

* Sharding based on a client's region
* Sharding based on the type of data being stored (e.g., user data is stored in one shard, payment data in another)
* Sharding based on the hash of a column (only for structured data)


#### Hot Spot and Load Balancers

When distributing a workload across a set of servers, the workload might be spread unevenly. This can occur if your sharding key or hashing function is suboptimal or if your workload is naturally skewed. As a result, some servers will receive significantly more traffic than others, creating a "hot spot."

Load balancers can be used to distribute traffic across replicas or shards, helping to mitigate hot spots and ensure a more even distribution of requests. They can automatically direct traffic to the least-loaded server or use other algorithms to optimize resource utilization.



### Replicas and Shards in Real-World Scenarios

Large-scale web applications, e-commerce platforms, and social media networks often require both replication and sharding to maintain performance and reliability. Replication and sharding serve different purposes but can complement each other to create a more robust and efficient system.

#### Replication: Redundancy, Fault Tolerance, and Read Throughput

Replication focuses on creating redundant copies of data across multiple servers to ensure fault tolerance and improved read throughput. While replication is focused on creating redundant copies of data across multiple servers to ensure fault tolerance and improved read throughput, sharding partitions the database into smaller segments (shards) distributed across multiple servers to enable horizontal scaling and increased write performance.

Here is a summary of the main differences between replication and sharding:


* **Data Redundancy**: Replication creates multiple identical copies of the entire database, while sharding divides the database into smaller, unique partitions.

* **Fault Tolerance**: Replication offers higher fault tolerance, as each replica can act as a backup in case of a server failure. Sharding, on its own, does not provide this level of fault tolerance. However, combining sharding with replication within each shard can provide similar redundancy.

* **Read Throughput**: Replication improves read throughput by distributing read operations across multiple replicas, reducing the load on individual servers. Sharding does not directly improve read throughput, as each shard contains only a portion of the data.

* **Write Performance and Scalability**: Sharding enhances write performance and scalability by distributing the data and workload across multiple servers. Replication, on the other hand, may introduce performance overhead for write-heavy applications, as each write operation must be propagated to all replicas to maintain consistency.

In summary, replication and sharding serve different purposes in real-world scenarios but complement each other to create a more robust and efficient system. By implementing these techniques effectively, large-scale applications can achieve enhanced performance and reliability. Combining replication and sharding offers the best of both worlds, providing the fault tolerance and read throughput benefits of replication and the increased write performance, throughput, and scalability of sharding.


#### Sharding: Throughput, Scalability, and Write Performance

Sharding focuses on partitioning the database into smaller segments (shards) that can be distributed across multiple servers. This allows for horizontal scaling and increases the overall throughput of the system, enabling it to handle a higher volume of concurrent requests. Sharding can also help to reduce contention and resource utilization on individual servers.

Using the e-commerce platform example, sharding can be employed to split the database based on product categories, user locations, or other factors. This partitioning approach enables the platform to handle a higher volume of concurrent requests, as each server only manages a subset of the data. This is particularly beneficial for write-heavy applications or scenarios where data growth requires efficient distribution and management.



### Conclusion 

Replication and sharding can be combined to further enhance the performance and resilience of a system. For instance, a sharded system can implement replication within each shard, creating multiple copies of the partitioned data. This combination provides the fault tolerance and read throughput benefits offered by replication and the increased write performance, throughput, and scalability of sharding.

In summary, replicas and shards serve different purposes in real-world scenarios but complement each other to create a more robust and efficient system.
Replication ensures data redundancy, fault tolerance, and improved read throughput, while sharding enhances write performance, throughput, and distributes data more efficiently. By implementing these techniques effectively, large-scale applications can achieve enhanced performance and reliability.