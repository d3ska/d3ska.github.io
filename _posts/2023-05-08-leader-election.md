---
title: "Leader Election"
categories:
  - System Design
tags:
  - Leader Election
---

In most societies, citizens elect their leaders through a voting system. Similarly, in a distributed system, servers utilize a set of algorithms to select a leader node. This process, known as "leader election," forms a critical part of many systems.

Let's consider the design of a subscription-based product system like Netflix or Amazon Prime, where users can opt for monthly or annual plans. A specific service within this system is tasked with periodically collating data from the database and sending requests to third-party systems to charge customers whose subscriptions are nearing expiration. As charging a client must not be duplicated, we need to ensure it only happens once. This is where leader election comes into play.

Leader election allows a group of servers or machines, each capable of performing the same task, to select a leader among themselves. This elected leader is solely responsible for executing the desired action, thereby preventing task duplication.

To illustrate, if we have four servers handling the business logic between the database and the third-party payment service, they will elect a leader amongst themselves. The leader server will handle the business logic while the other three servers (followers) stand ready to step in if the leader encounters issues. If the leader fails, a new leader is elected from the remaining servers to take over.


### Leader Election

Leader election is the process through which nodes in a cluster (e.g., servers in a server set) elect a "leader" among them. This leader takes charge of the primary operations of the service these nodes support. When implemented correctly, leader election ensures all nodes in the cluster are aware of the current leader and can elect a new one should the leader fail.


### Leader Election vs Consensus

It is important to distinguish between leader election and consensus, as the two concepts are related but distinct. **Leader election** is the process of selecting a single node from a cluster to serve a special role -- the elected leader coordinates work, and the remaining nodes act as followers. **Consensus**, on the other hand, is the broader problem of getting multiple nodes to agree on a single value (which could be a leader, a transaction commit decision, or any shared state). Leader election is effectively a specific application of consensus: the nodes reach agreement on *who* the leader is.

In practice, systems implement leader election *using* consensus algorithms, which is why the two are so closely intertwined.

### Consensus Algorithm

A consensus algorithm is a sophisticated type of algorithm that allows multiple entities to agree on a single data value. This could be used for determining the "leader" among a group of machines. Two popular consensus algorithms are Paxos and Raft.


### Paxos & Raft

Paxos and Raft are consensus algorithms that, when properly implemented, enable synchronization of specific operations, even in a distributed setting. These algorithms are essential for ensuring that distributed systems maintain consistency and agree on certain values or decisions, such as electing a leader.

Notable examples of systems that implement these consensus algorithms include Apache ZooKeeper, etcd, and HashiCorp Consul.

Apache ZooKeeper utilizes a variation of the Paxos algorithm known as Zab (ZooKeeper Atomic Broadcast), enabling it to maintain strong consistency across distributed deployments. It provides an infrastructure for cross-node synchronization and is widely used by high-scale distributed applications.

On the other hand, etcd is a distributed key-value store that uses the Raft consensus algorithm. It provides a reliable way to store data across a cluster of machines and is used extensively in the Kubernetes container orchestration system to store all cluster data.

Consul, developed by HashiCorp, also uses the Raft consensus algorithm and provides service discovery, health checking, and a distributed key-value store. It is a popular choice for leader election in microservice architectures.

All three systems illustrate the practical implementation of consensus algorithms like Paxos and Raft in real-world distributed systems.


### The Split-Brain Problem

One of the most dangerous failure modes in leader election is the **split-brain scenario**. This occurs when a network partition divides the cluster into two or more groups, and each group independently elects its own leader -- believing the other group's leader has failed. With two nodes both acting as the leader simultaneously, conflicting writes can occur, leading to data corruption or duplicated operations (such as charging a customer twice in the subscription example above).

Consensus algorithms like Raft and Paxos guard against split-brain by requiring a **majority quorum** to elect a leader. If the cluster has five nodes and a network partition isolates two nodes from the other three, only the partition with three nodes (the majority) can elect a leader. The minority partition knows it cannot form a quorum and will not elect a competing leader. This is why clusters are typically sized with an odd number of nodes -- it ensures that at most one partition can hold a majority.

Despite these safeguards, split-brain remains a real operational concern. Misconfigured timeouts, asymmetric network failures, or bugs in quorum logic can still cause it. Monitoring for duplicate leaders and using fencing tokens (unique, monotonically increasing identifiers attached to each leader term) are common defensive measures to detect and mitigate split-brain in production.