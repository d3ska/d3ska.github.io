---
title: "CAP Theorem"
categories:
  - Blog
tags:
  - Databases
  - CAP
---

The CAP theorem is a concept in distributed computing that states that in any distributed system, you can only have two out of the following three guarantees: consistency, availability, and partition tolerance.
It's reminds me of the “Fast/Good/Cheap” rule.

**Consistency:** All clients accessing the system see the same data at the same time, regardless of which node they connect to. This requires data to be replicated or forwarded to all nodes before a write is considered successful.

**Availability:** The system always responds to requests, even if one or more nodes are down. This means all working nodes must return a valid response for any request.

**Partition tolerance:** The system continues to function even if there are communication breakdowns or lost connections between nodes in the distributed system. This means the system is designed to handle network partitions and ensure that all nodes can still communicate with each other.


![img]({{site.url}}/assets/blog_images/2023-02-19-cap-theorem/venn-diagram-cap.png)


**CP database:** This type of database prioritizes consistency and partition tolerance, sacrificing availability. When there is a partition between nodes, the non-consistent node must be shut down and made unavailable until the partition is resolved.

**AP database:** This type of database prioritizes availability and partition tolerance, sacrificing consistency. When a partition occurs, all nodes remain available, but nodes on the wrong side of the partition might return an older version of data than others. After the partition is resolved, the database resyncs nodes to repair any inconsistencies.

**CA database:** This type of database prioritizes consistency and availability across all nodes, but cannot provide fault tolerance in the presence of a partition between any two nodes.

<br> 

Used and recommended sources:
* [Hazelcast article](https://hazelcast.com/glossary/cap-theorem/)
* [IBM article](https://www.ibm.com/topics/cap-theorem)
* [Educative article](https://www.educative.io/blog/what-is-cap-theorem)