---
title: "SQL vs NoSQL Databases"
categories:
  - System Design
tags:
  - Databases
  - SQL
  - NoSQL
  - Relational Databases
  - Document Stores
---

Choosing a database is one of the most consequential decisions in system design. The two broad families, SQL (relational) and NoSQL (non-relational), make fundamentally different trade-offs around data structure, consistency, and scalability. Understanding those trade-offs is more useful than memorizing feature lists.

### SQL Databases

SQL databases organize data into **tables** with predefined columns and types. Relationships between tables are expressed through foreign keys, and the SQL query language provides powerful joins, aggregations, and filtering. Well-known SQL databases include **PostgreSQL**, **MySQL**, **Oracle**, and **SQL Server**.

```sql
CREATE TABLE customers (
    id         BIGINT PRIMARY KEY,
    name       VARCHAR(100) NOT NULL,
    email      VARCHAR(255) UNIQUE
);

CREATE TABLE orders (
    id          BIGINT PRIMARY KEY,
    customer_id BIGINT REFERENCES customers(id),
    total       DECIMAL(10, 2),
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Find all orders for a given customer
SELECT c.name, o.id, o.total
FROM customers c
JOIN orders o ON o.customer_id = c.id
WHERE c.email = 'john@example.com';
```

The schema enforces structure: every row in `orders` must reference a valid `customers` row, and `email` must be unique. This rigidity is the source of both SQL's strengths and its limitations.

**Strengths:**

* **ACID transactions**: Atomicity, Consistency, Isolation, and Durability guarantee that data remains correct even under concurrent writes and failures.
* **Powerful query language**: Joins, subqueries, window functions, and aggregations let you ask complex questions about your data in a single query.
* **Schema enforcement**: The database validates data on write, catching errors early.

**Limitations:**

* **Rigid schema**: Altering a table with millions of rows (adding a column, changing a type) can be slow and require careful migration planning.
* **Horizontal scaling is harder**: SQL databases are designed to run on a single node. Scaling across multiple machines (sharding) is possible but adds significant operational complexity.

### NoSQL Databases

NoSQL databases do not enforce a fixed schema. They come in several distinct subtypes, each optimized for different access patterns:

**Document stores** (MongoDB, CouchDB) store data as flexible JSON-like documents. Each document can have a different structure, which makes them natural for content management, catalogs, and any domain where the data shape varies between records.

```javascript
// MongoDB: inserting a document
db.customers.insertOne({
    name: "John",
    email: "john@example.com",
    orders: [
        { total: 29.99, date: ISODate("2024-01-15") },
        { total: 49.50, date: ISODate("2024-02-20") }
    ]
});

// Querying nested data
db.customers.find(
    { "orders.total": { $gt: 30 } },
    { name: 1, "orders.total": 1 }
);
```

Notice how orders are embedded inside the customer document. There is no separate table and no join. This makes reads fast for this access pattern but makes it harder to query orders independently of customers.

**Key-value stores** (Redis, DynamoDB) offer simple get/put operations with extremely low latency, well-suited for caching and session management.

**Column-family stores** (Cassandra, HBase) organize data by columns rather than rows, excelling at write-heavy workloads and time-series data at scale.

**Graph databases** (Neo4j, Amazon Neptune) model data as nodes and edges, making them the natural choice for highly connected data like social networks and recommendation engines.

**Strengths:**

* **Flexible schema**: Documents or records can vary in structure, making it easy to evolve the data model without migrations.
* **Horizontal scalability**: Most NoSQL databases are designed from the ground up to distribute data across many nodes.
* **Optimized for specific access patterns**: Each subtype excels at the workload it was designed for.

**Limitations:**

* **Weaker consistency by default**: Many NoSQL databases favor availability and partition tolerance over strong consistency (see [CAP Theorem](/posts/cap-theorem/)). Some have added ACID support over time (MongoDB 4.0+ supports multi-document transactions, Google Spanner provides globally distributed strong consistency), but it is not the default.
* **Limited ad-hoc querying**: Without joins and a standardized query language, querying data across different "tables" (collections, key spaces) is often less convenient than in SQL.
* **Data duplication**: Denormalization (embedding related data together) improves read performance but means the same data may live in multiple places, making updates more complex.

### When to Choose Each

Rather than generic advice, here are concrete scenarios:

**Choose SQL when:**

* Your data is naturally relational (customers, orders, products with many-to-many relationships).
* You need ACID transactions (financial systems, inventory management, booking systems).
* You need complex ad-hoc queries with joins and aggregations (analytics, reporting).
* Your data model is well-understood and unlikely to change drastically.

**Choose NoSQL when:**

* Your data varies in structure (user-generated content, product catalogs with different attributes per category).
* You need to scale writes horizontally across many nodes (IoT telemetry, event logging).
* Your access patterns are simple and well-defined (cache lookups, session storage, time-series writes).
* You are building a system where availability and partition tolerance matter more than immediate consistency.

In practice, many systems use both. A common pattern is a SQL database as the source of truth for transactional data, with a NoSQL database (or cache) handling high-volume reads or specialized access patterns.

### NewSQL

**NewSQL** databases aim to bridge the gap between SQL and NoSQL. Databases like **CockroachDB**, **Google Spanner**, and **TiDB** provide the familiar SQL interface and ACID guarantees while supporting horizontal scalability traditionally associated with NoSQL systems. If your application demands both strong consistency and the ability to scale out across many nodes, NewSQL can be a compelling option.

### MongoDB Transactions vs Postgres Transactions

The NoSQL limitations section above mentions that MongoDB added multi-document transactions in version 4.0. This is true, but the details matter. MongoDB's **multi-document transactions** come with real constraints compared to what Postgres offers.

Postgres has decades of investment in its **MVCC (Multi-Version Concurrency Control)** implementation. Transactions are a first-class concept: they are cheap to start, handle complex isolation levels (Read Committed, Repeatable Read, Serializable), and work seamlessly with joins, constraints, and triggers. You rarely think twice about wrapping a set of operations in a transaction.

MongoDB's transaction support is more recent and carries **performance overhead**. Multi-document transactions in MongoDB acquire locks across documents and can increase write latency, especially in sharded clusters where a transaction spans multiple shards. MongoDB also recommends keeping transactions short (under 60 seconds by default) and avoiding them for the majority of operations, preferring single-document atomicity through schema design instead.

In practice, if your workload requires frequent multi-document transactions with strong isolation, Postgres (or another mature SQL database) will handle that more naturally. If your transactions are rare and most writes touch a single document, MongoDB's transaction support is a reasonable safety net rather than a core design pattern.

### Operational Complexity

Both SQL and NoSQL databases demand ongoing operational attention, but the nature of that work differs.

With SQL databases, you will spend time on **schema migrations** (adding columns, changing types, creating indexes on large tables), **connection pooling** (tools like PgBouncer become necessary at scale), and **index tuning** (analyzing slow queries, adding composite indexes, removing unused ones). The upside is that these problems are well-understood, with decades of tooling and documentation.

NoSQL databases shift the operational burden to different areas. **Partition key design** is critical: a poorly chosen partition key in DynamoDB or Cassandra can create hot partitions that throttle your throughput. **Capacity planning** requires thinking about read/write units or cluster sizing upfront rather than relying on a single vertically scaled node. And **eventual consistency** means your application code must handle scenarios where a read does not reflect the latest write, which adds complexity to your application layer rather than the database layer.

Neither side is "easier to operate." The work is just different, and the skills your team already has should factor into your decision.

### Cost of Managed Services

When using cloud-managed databases, the cost models diverge in ways that matter at scale.

**Managed SQL** services (AWS RDS, Google Cloud SQL, Azure Database) charge primarily by instance size and storage. You pick a machine, you pay for it whether it is idle or under full load. This makes costs predictable but means you pay for peak capacity even during off-hours. Read replicas and multi-AZ deployments add roughly linear cost.

**Managed NoSQL** services (DynamoDB, Cosmos DB, Cloud Firestore) often use a **pay-per-request** or **provisioned throughput** model. For simple, high-volume key-value lookups, this can be significantly cheaper at scale because you are paying for what you use. However, costs can spike unpredictably when access patterns change, and complex queries (scans, secondary indexes, cross-partition reads) can become expensive quickly.

A rough rule of thumb: NoSQL managed services tend to be cheaper for high-throughput, simple access patterns. SQL managed services tend to be more cost-effective when your queries are complex and varied, because the cost of running a join on a SQL instance is the same as running a simple SELECT, while the equivalent denormalized NoSQL queries might require multiple reads across different items or collections.

### What Happens When You Choose Wrong

Migrating from SQL to NoSQL (or the other direction) is one of the most painful projects a team can undertake. It is worth understanding the costs up front so you take the initial decision seriously.

**Schema and data transformation** is the first hurdle. Moving from normalized SQL tables to denormalized NoSQL documents (or vice versa) requires rethinking your data model entirely. You cannot simply export rows and import them as documents. Relationships that were expressed through foreign keys need to be flattened into embedded objects or duplicated across collections. Going the other direction, you need to extract structure from flexible documents and enforce constraints that may not have existed before.

**Query rewriting** follows. Every query in your application was written for one paradigm. SQL joins become multiple lookups in NoSQL. NoSQL single-document reads may need to be split across multiple SQL tables. ORMs and data access layers need to be rewritten or replaced. This is not a search-and-replace task.

**Downtime and data integrity risks** are the most dangerous part. A live migration typically requires a period of dual-writing to both old and new databases, followed by a cutover. During this window, you need to keep both systems in sync, handle failures in either system, and validate that no data was lost or corrupted. Even with careful planning, many teams experience data inconsistencies that take weeks to track down.

The lesson is not to avoid NoSQL or SQL, but to invest time in the initial choice. Prototyping your most important queries against both options before committing is far cheaper than migrating later.

> **Related posts**: [Replication And Sharding](/posts/replication-and-sharding/), [CAP Theorem](/posts/cap-theorem/), [Specialized Storage Paradigms](/posts/specialized-storage-paradigms/)
