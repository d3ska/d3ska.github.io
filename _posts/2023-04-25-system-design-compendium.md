---
title: "System Design Compendium"
categories:
  - System Design
tags:
  - System Design
  - Software Architecture
  - Distributed Systems
---

System design is a part of the system development process, focusing on the creation of a comprehensive blueprint for a system to satisfy specified requirements. It includes the specification of the system's architecture, modules, interfaces and data necessary to satisfy a set of requirements.

In simpler terms, system design is like constructing a roadmap for a complex mechanism. This roadmap delineates the system's components, how they interact, and work together to fulfill the system's objectives or requirements.

Effective system design should consider both the system's functional requirements—what the system should do—and its non-functional requirements, such as scalability, reliability, robustness, and security.

Here are the topics that merit your focused attention and comprehension:

* [Defining Software Architecture](https://matthewonsoftware.com/system%20design/defining-software-architecture/) -- the principles, patterns, and trade-offs behind structuring maintainable, scalable software systems.
* [Network Protocols](https://matthewonsoftware.com/system%20design/network-protocols/) -- how machines communicate over networks, covering TCP, UDP, HTTP, and other foundational protocols.
* [Latency And Throughput](https://matthewonsoftware.com/system%20design/latency-and-throughput/) -- the two key performance metrics for any system, what they measure, and why optimizing one does not automatically improve the other.
* [Availability](https://matthewonsoftware.com/system%20design/availability/) -- designing systems that stay operational under failure, including uptime guarantees, redundancy strategies, and SLAs.
* [CAP Theorem](https://matthewonsoftware.com/system%20design/cap-theorem/) -- the fundamental trade-off between consistency, availability, and partition tolerance in distributed systems.
* [Caching](https://matthewonsoftware.com/system%20design/caching/) -- storing frequently accessed data closer to the consumer to reduce latency and backend load.
* [Proxies](https://matthewonsoftware.com/system%20design/proxies/) -- intermediary servers that sit between clients and backends, enabling load distribution, security, and caching.
* [Load Balancing](https://matthewonsoftware.com/system%20design/load-balancers/) -- distributing incoming traffic across multiple servers to maximize throughput and minimize response times.
* [Hashing](https://matthewonsoftware.com/system%20design/hashing/) -- mapping data to fixed-size values for efficient lookups, and how consistent hashing enables scalable distributed systems.
* [SQL vs NoSQL Databases](https://matthewonsoftware.com/system%20design/sql-vs-no-sql-databases/) -- comparing relational and non-relational databases, their data models, and when to choose each.
* [Specialized Storage Paradigms](https://matthewonsoftware.com/system%20design/specialized-storage-paradigms/) -- purpose-built storage solutions like blob stores, time-series databases, and graph databases for domain-specific workloads.
* [Replication And Sharding](https://matthewonsoftware.com/system%20design/replication-and-sharding/) -- techniques for duplicating and partitioning data across machines to improve reliability and performance.
* [Leader Election](https://matthewonsoftware.com/system%20design/leader-election/) -- how distributed systems choose a single node to coordinate work, ensuring consistency without conflicts.
* [Peer-To-Peer Networks](https://matthewonsoftware.com/system%20design/peer-to-peer-networks/) -- decentralized architectures where nodes share resources directly without relying on a central server.
* [Polling And Streaming](https://matthewonsoftware.com/system%20design/polling-and-streaming/) -- two approaches for clients to receive updates from servers, each with different latency and resource trade-offs.
* [Rate Limiting](https://matthewonsoftware.com/system%20design/rate-limiting/) -- controlling the number of requests a client can make to protect systems from overload and abuse.
* [Microservices vs Monolith](https://matthewonsoftware.com/blog/microservices-vs-monolith/) -- comparing two architectural styles for structuring applications and their trade-offs in complexity, deployment, and scaling.
* [Request-Response vs Publish-Subscribe](https://matthewonsoftware.com/blog/request-response-vs-publish-subscribe/) -- two fundamental communication patterns for services, differing in coupling, scalability, and failure handling.

