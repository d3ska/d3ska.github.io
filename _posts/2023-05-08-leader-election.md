---
title: "Leader Election"
categories:
  - System Design
tags:
  - Leader Election
---

In most societies, citizens elect their leaders through a voting system. Similarly, in a distributed system, servers utilize a set of algorithms to select a master node. This process, known as "leader election" or "algorithmocracy," forms a critical part of many systems.

Let's consider the design of a subscription-based product system like Netflix or Amazon Prime, where users can opt for monthly or annual plans. A specific service within this system is tasked with periodically collating data from the database and sending requests to third-party systems to charge customers whose subscriptions are nearing expiration. As charging a client must not be duplicated, we need to ensure it only happens once. This is where leader election comes into play.

Leader election allows a group of servers or machines, each capable of performing the same task, to select a leader among themselves. This elected leader is solely responsible for executing the desired action, thereby preventing task duplication.

To illustrate, if we have four servers handling the business logic between the database and the third-party payment service, they will elect a leader amongst themselves. The leader server will handle the business logic while the other three servers (followers) stand ready to step in if the leader encounters issues. If the leader fails, a new leader is elected from the remaining servers to take over.


### Leader Election

Leader election is the process through which nodes in a cluster (e.g., servers in a server set) elect a "leader" among them. This leader takes charge of the primary operations of the service these nodes support. When implemented correctly, leader election ensures all nodes in the cluster are aware of the current leader and can elect a new one should the leader fail.


### Consensus Algorithm

A consensus algorithm is a sophisticated type of algorithm that allows multiple entities to agree on a single data value. This could be used for determining the “leader” among a group of machines. Two popular consensus algorithms are Paxos and Raft.


### Paxos & Raft

Paxos and Raft are consensus algorithms that, when properly implemented, enable synchronization of specific operations, even in a distributed setting. These algorithms are essential for ensuring that distributed systems maintain consistency and agree on certain values or decisions, such as electing a leader.

Notable examples of systems that implement these consensus algorithms include Apache ZooKeeper and etcd.

Apache ZooKeeper utilizes a variation of the Paxos algorithm known as Zab (ZooKeeper Atomic Broadcast), enabling it to maintain strong consistency across distributed deployments. It provides an infrastructure for cross-node synchronization and is widely used by high-scale distributed applications.

On the other hand, etcd is a distributed key-value store that uses the Raft consensus algorithm. It provides a reliable way to store data across a cluster of machines and is used extensively in the Kubernetes container orchestration system to store all cluster data.

Both systems illustrate the practical implementation of consensus algorithms like Paxos and Raft in real-world distributed systems.