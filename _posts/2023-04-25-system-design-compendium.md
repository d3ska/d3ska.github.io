---
title: "System Design Compendium"
categories:
  - System Design
tags:
  - System Design
  - Software Architecture
  - Distributed Systems
---

System design is the process of defining a system's architecture, components, interfaces, and data flows to satisfy a set of requirements. It covers both functional requirements (what the system should do) and non-functional requirements such as scalability, reliability, and security.

Here are the topics that form the foundation of system design:

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
* [Microservices vs Monolith](/posts/microservices-vs-monolith/) - comparing two architectural styles for structuring applications and their trade-offs in complexity, deployment, and scaling.
* [Request-Response vs Publish-Subscribe](/posts/request-response-vs-publish-subscribe/) - two fundamental communication patterns for services, differing in coupling, scalability, and failure handling.
