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

Your database handles 10,000 reads and 2,000 writes per second. One day traffic doubles. The single database server is at 95% CPU, queries are timing out, and you need a solution by Friday. You have two fundamental tools at your disposal: **replication** (copying data across multiple servers) and **sharding** (splitting data across multiple servers). They solve different problems, and understanding when to reach for each one, or both, is one of the most important skills in system design.

### Replication

**Replication** involves duplicating data from one database server to others. The most common setups are master-slave (one writer, many readers) and master-master (multiple writers). Replication serves two main purposes:

- **Redundancy and fault tolerance.** If the primary server goes down, a replica can take over. Your system stays available even during hardware failures or regional outages.
- **Reduced latency.** By placing replicas closer to clients geographically, you can cut read latency significantly. A user in Tokyo reading from a Tokyo replica is going to have a much better experience than reading from a server in Virginia.

Replication is especially powerful for read-heavy workloads. If 80% of your traffic is reads, you can spread that load across five replicas and immediately reduce the pressure on any single server.

#### Keeping Replicas Up-To-Date

There are two ways to keep replicas updated: **synchronous** and **asynchronous**. Both come with trade-offs that should be considered on a case-by-case basis.

**Synchronous replication** means the primary waits for all replicas to confirm a write before acknowledging it to the client. This gives you strong consistency (every read sees the latest write) but at the cost of higher latency and lower availability. If any replica is slow or unreachable, the whole write stalls.

**Asynchronous replication** means the primary acknowledges the write immediately and propagates it to replicas in the background. This gives you lower latency and higher availability, but a client reading from a replica might see stale data for a short window. This is called **eventual consistency**.

The tension between these two approaches is a direct manifestation of the [CAP Theorem](/posts/cap-theorem/). In a distributed system, you cannot have strong consistency and high availability at the same time when network partitions occur. You have to choose which one matters more for your use case.

### Sharding

Also known as **data partitioning**, sharding involves splitting a database into two or more pieces called **shards**. Each shard holds a subset of the total data and runs on its own server (or cluster of servers). This is typically done to increase the throughput and storage capacity of your database beyond what a single machine can handle.

Where replication helps with reads, sharding helps with writes. If your single database is overwhelmed by write volume, no number of read replicas will fix that. You need to distribute the writes themselves, and that means sharding.

#### Shard Key Selection

The most important decision in sharding is choosing your **shard key**, the field used to determine which shard a given piece of data lives on. A good shard key distributes data evenly and aligns with your query patterns. A bad one creates hot spots and forces expensive cross-shard queries.

Here are some concrete examples:

- **Good: `user_id` for a social network.** User IDs are typically evenly distributed (UUIDs or auto-incrementing integers), and most queries are user-scoped ("show me this user's posts," "load this user's profile"). Each query hits exactly one shard.
- **Bad: `created_date` for a time-series workload.** All recent writes hammer the same shard (the one holding "today's" range), while older shards sit idle. You get a massive hot spot on the most recent shard.
- **Bad: `country` for a global application.** A handful of countries (the US, India, Brazil) will dominate traffic, creating huge imbalances between shards.

The simplest way to route a key to a shard is a hash-based approach:

```java
public class ShardRouter {

    private final int numberOfShards;

    public ShardRouter(int numberOfShards) {
        this.numberOfShards = numberOfShards;
    }

    public int getShardIndex(String userId) {
        return Math.abs(userId.hashCode() % numberOfShards);
    }
}
```

This gives you uniform distribution, but it has a weakness: if you change `numberOfShards`, almost every key maps to a different shard, forcing a massive data migration.

#### Sharding Strategies

There are several well-established strategies for distributing data across shards:

| Strategy | How it works | Strengths | Weaknesses |
|---|---|---|---|
| Range-based | Contiguous key ranges per shard | Efficient range queries | Uneven distribution if data is skewed |
| Hash-based | Hash function assigns key to shard | Uniform distribution | Range queries require scatter-gather |
| Directory-based | Lookup table maps keys to shards | Maximum flexibility | Lookup table is SPOF and bottleneck |
| Geographic | Partition by client region | Low latency for local users | Cross-region queries are expensive |

There is also **functional sharding**, which splits data based on the type of data being stored (e.g., user data in one shard, payment data in another). This is simpler to reason about but does not help when a single data type outgrows a single server.

#### Resharding

What happens when shard 3 is full and you need shard 5? **Resharding** is one of the hardest operational problems in distributed systems. With a naive modulo-based approach (`hash % N`), changing `N` remaps nearly every key to a different shard. This means migrating most of your data, which is expensive, error-prone, and can cause downtime.

**Consistent hashing** is the standard solution to this problem. Instead of mapping keys to shard numbers directly, you place both shards and keys on a virtual ring (typically using a hash space of 0 to 2^32). Each key is assigned to the first shard found clockwise from its position on the ring. When you add a new shard, only the keys between the new shard and its immediate neighbor need to move. Everything else stays put.

In practice, each physical shard is mapped to multiple **virtual nodes** on the ring to ensure an even distribution. Systems like Amazon DynamoDB, Apache Cassandra, and Memcached all use variants of consistent hashing for exactly this reason.

#### Hot Spots

Even with a good shard key and a solid hashing strategy, hot spots can still emerge. A **hot spot** occurs when one shard receives disproportionately more traffic than others. Common causes include:

- **Celebrity accounts.** A single user with millions of followers generates enormous read traffic on whichever shard holds their data.
- **Viral content.** A single post or piece of content suddenly attracts millions of requests.
- **Temporal patterns.** Flash sales, breaking news, or scheduled events concentrate traffic on specific data.

Concrete solutions to hot spots:

- **Splitting hot shards.** If shard 2 is consistently overloaded, split it into two smaller shards and redistribute its data.
- **Adding read replicas for hot shards.** If the hot spot is read-heavy (celebrity profile views), replicate that specific shard and spread reads across its replicas.
- **Rate limiting.** Protect the hot shard by throttling excessive requests and serving cached responses instead.
- **Application-level caching.** Cache hot data (celebrity profiles, viral posts) in a layer like Redis or Memcached so most requests never reach the shard at all.

**Load balancers** also play a role here. They can distribute traffic across replicas or shards, automatically directing requests to the least-loaded server. But load balancers are not a substitute for fixing the root cause. If your shard key is bad, a load balancer can only spread the pain, not eliminate it.

#### Real-World Examples

Twitter shards tweets by `user_id` because most queries are user-scoped ("show me this user's timeline"). This means loading a single user's tweets hits exactly one shard. Instagram shards by `photo_id` for similar reasons: the most common operation is loading a specific photo and its metadata.

Both companies combine sharding with replication. Each shard is replicated for redundancy, so losing a single node does not mean losing data or experiencing downtime.

### Replication vs Sharding: A Comparison

Large-scale web applications, e-commerce platforms, and social media networks often require both replication and sharding to maintain performance and reliability. While they serve different purposes, they complement each other to create a more robust and efficient system.

Here is a summary of the main differences:

- **Data redundancy.** Replication creates multiple identical copies of the entire database, while sharding divides the database into smaller, unique partitions.

- **Fault tolerance.** Replication offers higher fault tolerance, as each replica can act as a backup in case of a server failure. Sharding, on its own, does not provide this level of fault tolerance. However, combining sharding with replication within each shard can provide similar redundancy.

- **Read throughput.** Replication improves read throughput by distributing read operations across multiple replicas, reducing the load on individual servers. Sharding does not directly improve read throughput, as each shard contains only a portion of the data.

- **Write performance and scalability.** Sharding enhances write performance and scalability by distributing the data and workload across multiple servers. Replication, on the other hand, may introduce performance overhead for write-heavy applications, as each write operation must be propagated to all replicas to maintain consistency.

### Conclusion

In practice, the decision between replication and sharding comes down to your system's dominant bottleneck:

- **Choose replication** when your primary concerns are fault tolerance, high availability, and read-heavy workloads. Replication is simpler to set up and reason about, and it keeps your operational complexity low.
- **Choose sharding** when you are hitting the storage or write-throughput limits of a single machine. Sharding unlocks horizontal scaling but introduces complexity around cross-shard queries, rebalancing, and data locality.
- **Combine both** when your system demands high availability *and* horizontal write scalability, which is the norm for large-scale production systems. A common pattern is to shard the database for write distribution and then replicate each shard for redundancy, so that losing a single node does not cause data loss or downtime.

The key takeaway is that replication and sharding are not competing techniques. They address orthogonal concerns, data redundancy versus data distribution, and the most resilient architectures leverage both in tandem.

> **Related posts**: [CAP Theorem](/posts/cap-theorem/), [Availability](/posts/availability/), [Hashing](/posts/hashing/), [SQL vs NoSQL Databases](/posts/sql-vs-no-sql-databases/)
