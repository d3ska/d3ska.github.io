---
title: "Leader Election"
categories:
  - System Design
tags:
  - Leader Election
  - Distributed Systems
  - Consensus
  - Fault Tolerance
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

### Consensus Algorithms

A **consensus algorithm** is a protocol that allows multiple nodes to agree on a single data value, even when some nodes fail or messages are delayed. This could be used for determining the "leader" among a group of machines, committing a distributed transaction, or replicating a log entry. The two most widely studied consensus algorithms are Paxos and Raft.


### Paxos & Raft

#### Paxos

**Paxos**, introduced by Leslie Lamport in 1989, was the first proven consensus algorithm for asynchronous systems. Understanding its core mechanism helps explain why leader election works the way it does.

Paxos operates in two phases. In the **Prepare phase**, a node that wants to propose a value (called a proposer) sends a "prepare" request with a unique, monotonically increasing proposal number to a majority of nodes (called acceptors). Each acceptor promises not to accept any proposal with a lower number and replies with the highest-numbered proposal it has already accepted, if any. In the **Accept phase**, once the proposer has received promises from a majority, it sends an "accept" request with its proposal number and the value it wants to commit. If a majority of acceptors accept, the value is chosen and consensus is reached.

The key insight is that requiring a majority at each phase guarantees that any two majorities overlap by at least one node. This overlap means conflicting values cannot both be accepted, which is what makes Paxos safe even under network partitions and node failures.

In practice, raw Paxos is notoriously hard to implement correctly. Most production systems use Multi-Paxos, which optimizes the common case by electing a stable leader that skips the Prepare phase for subsequent proposals. Apache ZooKeeper, for example, uses a variation called **Zab (ZooKeeper Atomic Broadcast)** that builds on these ideas to maintain strong consistency across distributed deployments.

#### Raft

**Raft**, published by Diego Ongaro and John Ousterhout in 2014, was designed explicitly to be easier to understand than Paxos while providing the same safety guarantees. It achieves this by decomposing consensus into three subproblems: leader election, log replication, and safety.

In Raft, every node is in one of three states: **follower**, **candidate**, or **leader**. All nodes start as followers. If a follower does not hear from a leader within a randomized election timeout (typically 150 to 300 milliseconds), it transitions to candidate, increments its **term** number (a logical clock), votes for itself, and sends vote requests to all other nodes. A node grants its vote to the first candidate it hears from in a given term, and the candidate that receives votes from a majority becomes the leader. The randomized timeout is what prevents most split votes, because nodes rarely time out at exactly the same moment.

Once elected, the leader sends periodic heartbeats to all followers to maintain its authority. If the leader crashes or becomes unreachable, followers will eventually time out and trigger a new election. Because each term can have at most one leader (a node only votes once per term, and winning requires a majority), Raft guarantees that two leaders cannot coexist in the same term.

**etcd**, the distributed key-value store that backs Kubernetes, is built on Raft. So is **HashiCorp Consul**, which provides service discovery, health checking, and a distributed key-value store for microservice architectures.

#### Paxos vs Raft

Both algorithms guarantee safety (no two different values can be chosen for the same decision), but they differ in approach. Paxos is more general and permits multiple concurrent proposers, which makes it flexible but harder to reason about. Raft constrains the design by funneling all decisions through a single leader, which simplifies implementation and debugging at the cost of a brief unavailability window during leader transitions. For most practical systems, this trade-off is well worth it, which is why Raft has become the dominant choice in modern infrastructure.


### The Split-Brain Problem

One of the most dangerous failure modes in leader election is the **split-brain scenario**. This occurs when a network partition divides the cluster into two or more groups, and each group independently elects its own leader -- believing the other group's leader has failed. With two nodes both acting as the leader simultaneously, conflicting writes can occur, leading to data corruption or duplicated operations (such as charging a customer twice in the subscription example above).

Consensus algorithms like Raft and Paxos guard against split-brain by requiring a **majority quorum** to elect a leader. If the cluster has five nodes and a network partition isolates two nodes from the other three, only the partition with three nodes (the majority) can elect a leader. The minority partition knows it cannot form a quorum and will not elect a competing leader. This is why clusters are typically sized with an odd number of nodes -- it ensures that at most one partition can hold a majority.

Despite these safeguards, split-brain remains a real operational concern. Misconfigured timeouts, asymmetric network failures, or bugs in quorum logic can still cause it. Monitoring for duplicate leaders and using fencing tokens (unique, monotonically increasing identifiers attached to each leader term) are common defensive measures to detect and mitigate split-brain in production.


### Failure Detection and Lease Mechanisms

Even with a correct consensus algorithm, the practical question remains: how does the cluster decide that the current leader has failed? Getting this wrong causes either unnecessary leader transitions (if you declare the leader dead too quickly) or prolonged unavailability (if you wait too long).

#### Heartbeats

The most common mechanism is a **heartbeat**. The leader periodically sends a small message to every follower (in Raft, these double as `AppendEntries` RPCs). If a follower has not received a heartbeat within a configured timeout interval, it assumes the leader is gone and starts a new election. Choosing the right timeout is a balancing act: too short and a brief network hiccup triggers a needless election, too long and the cluster sits idle waiting for a dead leader to come back.

A useful rule of thumb is to set the election timeout to at least ten times the network round-trip time. If your average RTT is 2 ms, an election timeout of 150 to 300 ms is reasonable. In higher-latency environments (cross-region deployments), the timeout may need to be several seconds.

#### Leader Leases

A **leader lease** adds a time-bound guarantee on top of heartbeats. Instead of relying only on heartbeat absence, the leader obtains a lease (a time-limited lock) that explicitly grants it authority for a fixed duration (for example, 15 seconds). The leader must renew the lease before it expires, or it loses its leadership status. Followers will not elect a new leader until the current lease has expired.

Leases help prevent a subtle problem: a leader that has been network-partitioned away from the cluster may not realize it has lost leadership. Without a lease, it could continue processing requests and issuing writes while the rest of the cluster has already elected a new leader. With a lease, the old leader knows its authority has a hard expiration time and will stop serving once the lease cannot be renewed.

#### Fencing Tokens

Even with leases, there is a window of vulnerability. Suppose a leader's lease expires at time T, but due to a garbage collection pause or clock skew, the leader does not realize this until T+2 seconds. During that window, both the old leader and the newly elected leader may issue writes.

**Fencing tokens** close this gap. Every time a new leader is elected, the system issues a monotonically increasing token (essentially the term number or epoch). All write requests carry this token, and the storage layer rejects any write with a token lower than the highest it has already seen. If the old leader (with token 5) tries to write after the new leader (with token 6) has already written, the storage layer rejects the stale write. This is a simple but powerful defense-in-depth mechanism.


### Leader Election with Apache Curator

In practice, you rarely implement consensus from scratch. Tools like Apache ZooKeeper provide the building blocks, and client libraries like **Apache Curator** wrap them in a clean API. Below is a practical example of leader election using Curator's `LeaderSelector`, which handles ZooKeeper session management, connection loss, and re-election automatically.

```java
public class SubscriptionBillingLeader implements LeaderSelectorListenerAdapter, Closeable {

    private final String name;
    private final LeaderSelector leaderSelector;
    private final SubscriptionBillingService billingService;

    public SubscriptionBillingLeader(CuratorFramework client,
                                     String leaderPath,
                                     String name,
                                     SubscriptionBillingService billingService) {
        this.name = name;
        this.billingService = billingService;
        this.leaderSelector = new LeaderSelector(client, leaderPath, this);
        // automatically re-queue for leadership when the takeLeadership() method returns
        this.leaderSelector.autoRequeue();
    }

    public void start() {
        leaderSelector.start();
    }

    @Override
    public void close() throws IOException {
        leaderSelector.close();
    }

    @Override
    public void takeLeadership(CuratorFramework client) throws Exception {
        System.out.println(name + " is now the leader. Starting billing cycle.");

        try {
            while (true) {
                billingService.chargeExpiringSubscriptions();
                // run every 60 seconds while this node holds leadership
                Thread.sleep(60_000);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.out.println(name + " was interrupted, giving up leadership.");
        }

        // when this method returns, leadership is released and a new election begins
        System.out.println(name + " relinquishing leadership.");
    }
}
```

Each instance of `SubscriptionBillingLeader` represents one of the servers in the subscription billing cluster. When you start multiple instances pointing at the same ZooKeeper path (for example, `/billing/leader`), Curator ensures that exactly one of them holds leadership at any time. If the leader's ZooKeeper session expires (because the process crashed or lost network connectivity), ZooKeeper's ephemeral node is automatically deleted, and another participant wins the election within seconds.

```java
public class BillingApplication {

    public static void main(String[] args) throws Exception {
        CuratorFramework client = CuratorFrameworkFactory.newClient(
                "zookeeper1:2181,zookeeper2:2181,zookeeper3:2181",
                new ExponentialBackoffRetry(1000, 3)
        );
        client.start();

        SubscriptionBillingService billingService = new SubscriptionBillingService();
        String instanceName = "billing-node-" + ManagementFactory.getRuntimeMXBean().getName();

        try (SubscriptionBillingLeader leader =
                     new SubscriptionBillingLeader(client, "/billing/leader", instanceName, billingService)) {
            leader.start();

            // keep the process running
            Thread.currentThread().join();
        }
    }
}
```

Under the hood, Curator uses ZooKeeper's **ephemeral sequential nodes** to implement the election. Each participant creates an ephemeral node under the leader path, and the participant with the lowest sequence number becomes the leader. When that participant's session ends, its ephemeral node disappears, the next participant in sequence takes over, and the cycle continues. This is why ZooKeeper-based leader election is so reliable: it piggybacks on ZooKeeper's session guarantees and its consensus protocol (Zab) for the ephemeral node semantics.

> **Related posts**: [Replication And Sharding](/posts/replication-and-sharding/), [Availability](/posts/availability/), [CAP Theorem](/posts/cap-theorem/)