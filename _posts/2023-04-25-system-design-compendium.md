---
title: "System Design Compendium"
categories:
  - System Design
tags:
  - System Design
  - Software Architecture
  - Distributed Systems
---

Whether you are preparing for a system design interview or building production infrastructure, the same core concepts keep showing up: how data moves, where it lives, and what happens when things fail. This compendium collects those concepts in one place so you can build a solid mental model of how large-scale systems work.

### Topics

* [Defining Software Architecture](/posts/defining-software-architecture/) - the principles, patterns, and trade-offs behind structuring maintainable, scalable software systems.
* [Network Protocols](/posts/network-protocols/) - how machines communicate over networks, covering TCP, UDP, HTTP, and other foundational protocols.
* [Latency and Throughput](/posts/latency-and-throughput/) - the two key performance metrics for any system, what they measure, and why optimizing one does not automatically improve the other.
* [Availability](/posts/availability/) - designing systems that stay operational under failure, including uptime guarantees, redundancy strategies, and SLAs.
* [CAP Theorem](/posts/cap-theorem/) - the fundamental trade-off between consistency, availability, and partition tolerance in distributed systems.
* [Caching](/posts/caching/) - storing frequently accessed data closer to the consumer to reduce latency and backend load.
* [Proxies](/posts/proxies/) - intermediary servers that sit between clients and backends, enabling load distribution, security, and caching.
* [Load Balancing](/posts/load-balancers/) - distributing incoming traffic across multiple servers to maximize throughput and minimize response times.
* [Hashing](/posts/hashing/) - mapping data to fixed-size values for efficient lookups, and how consistent hashing enables scalable distributed systems.
* [SQL vs NoSQL Databases](/posts/sql-vs-no-sql-databases/) - comparing relational and non-relational databases, their data models, and when to choose each.
* [Specialized Storage Paradigms](/posts/specialized-storage-paradigms/) - purpose-built storage solutions like blob stores, time-series databases, and graph databases for domain-specific workloads.
* [Replication and Sharding](/posts/replication-and-sharding/) - techniques for duplicating and partitioning data across machines to improve reliability and performance.
* [Leader Election](/posts/leader-election/) - how distributed systems choose a single node to coordinate work, ensuring consistency without conflicts.
* [Peer-To-Peer Networks](/posts/peer-to-peer-networks/) - decentralized architectures where nodes share resources directly without relying on a central server.
* [Polling and Streaming](/posts/polling-and-streaming/) - two approaches for clients to receive updates from servers, each with different latency and resource trade-offs.
* [Rate Limiting](/posts/rate-limiting/) - controlling the number of requests a client can make to protect systems from overload and abuse.
* [Microservices vs Monolith](/posts/microservices-vs-monolith/) - comparing two architectural styles for structuring applications, with trade-offs in complexity, deployment, and scaling.
* [Request-Response vs Publish-Subscribe](/posts/request-response-vs-publish-subscribe/) - two fundamental communication patterns for services, differing in coupling, scalability, and failure handling.

### Suggested Reading Order

If you are new to system design, working through these topics in a structured order will help each concept build on the last.

**1. Foundations.** Start here to understand the vocabulary and core metrics.
- Defining Software Architecture
- Network Protocols
- Latency and Throughput
- Availability

**2. Data and storage.** Learn how data is stored, accessed, and distributed.
- SQL vs NoSQL Databases
- Specialized Storage Paradigms
- Caching
- Hashing
- Replication and Sharding

**3. Infrastructure and traffic.** Understand how requests flow through a system.
- Proxies
- Load Balancing
- Rate Limiting
- Polling and Streaming

**4. Distributed systems.** Dig into the harder problems that come with scale.
- CAP Theorem
- Leader Election
- Peer-To-Peer Networks

**5. Architecture patterns.** Tie it all together with high-level design decisions.
- Microservices vs Monolith
- Request-Response vs Publish-Subscribe

You do not need to follow this order strictly. If you already have experience with networking and databases, jumping straight to the distributed systems or architecture sections works fine too.
