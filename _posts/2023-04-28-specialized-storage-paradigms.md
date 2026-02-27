---
title: "Specialized Storage Paradigms"
categories:
  - System Design
tags:
  - Databases
  - Blob Store
  - Time Series
  - Graph Database
  - Spatial Database
  - Quadtree
---

In addition to widely known SQL and NoSQL databases, such as document or key-value stores, there are specialized storage
paradigms that are well-suited for specific use cases. We will discuss four of them below:

### Blob Storage

Blob storage is a common storage solution used in both small and large-scale systems. While not considered a database
per se, it allows users to store and retrieve data based on a blob's name. This is similar to a key-value store, but
with different performance guarantees. Blob storage may be slower than key-value stores, but it can handle much larger
values, such as megabytes or even gigabytes.

Common use cases for blob storage include storing large binaries, database snapshots, or static assets like images for
websites. Blob storage infrastructure is complex to maintain on-premises and is typically provided by large companies
like Google, Amazon, or Microsoft. In the context of system design interviews, you can assume the availability of GCS,
S3, or Azure Blob Storage, which are blob storage services hosted by Google, Amazon, and Microsoft, respectively.

### Time Series Database

A Time Series Database (TSDB) is a specialized database optimized for storing and analyzing time-indexed data -- data
points occurring at specific moments in time. TSDBs are built around the assumption that data arrives in chronological order and is predominantly appended rather than updated in place, which allows them to use highly efficient compression and storage strategies.

Common TSDBs include **InfluxDB**, **TimescaleDB** (a PostgreSQL extension), **Prometheus**, and **Graphite**. Typical use cases extend well beyond basic monitoring:

* **Infrastructure and application metrics** -- tracking CPU, memory, request latency, and error rates over time.
* **IoT sensor data** -- ingesting millions of data points per second from connected devices.
* **Financial data** -- storing tick-by-tick stock prices, trading volumes, and market indicators for time-based analysis.

TSDBs typically offer built-in support for downsampling, retention policies, and time-windowed aggregations, making them far more efficient for these workloads than a general-purpose relational database.

### Graph Database

Graph databases store data using a graph data model, allowing data entries to have explicitly defined relationships,
similar to nodes and edges in a graph. They leverage their graph structure to perform complex queries on deeply
connected data efficiently.

Graph databases are often preferred over relational databases when dealing with systems where data points naturally form
a graph with multiple levels of relationships, such as social networks.

However, graph databases are not the right choice for every scenario. For simple relational data with well-defined schemas and straightforward join patterns, a traditional SQL database will typically outperform a graph database and be easier to maintain. Graph databases can also struggle with high-write, bulk-ingest workloads, as maintaining index structures for edges and nodes adds overhead compared to row-oriented or column-oriented stores.

#### Cypher

Cypher is a graph query language originally developed for the Neo4j graph database but has since been standardized for
use with other graph databases, aiming to become the "SQL for graphs." Cypher queries are generally simpler than their
SQL counterparts. For example, a Cypher query to find data in Neo4j, a popular graph database:

```cypher
MATCH (some_node:SomeLabel)-[:SOME_RELATIONSHIP]->(some_other_node:SomeLabel {some_property:'value'})
```

### Spatial Database

Spatial databases are designed for storing and querying spatial data, such as map locations. They utilize spatial
indexes to perform spatial queries efficiently, like finding all locations near a specific region. Common indexing mechanisms include **quadtrees** (which recursively subdivide 2D space into four quadrants), **R-trees** (which group nearby objects into bounding rectangles, excelling at range queries and nearest-neighbor searches), and **geohashes** (which encode latitude/longitude into a single string, enabling efficient prefix-based proximity lookups). The choice of index depends on the query patterns -- quadtrees work well for uniformly distributed point data, R-trees excel with overlapping regions and polygons, and geohashes are simple to implement and integrate with key-value stores.

#### Quadtree

A quadtree is a tree data structure commonly used to index two-dimensional spatial data. Each node in a quadtree has
either zero child nodes (leaf nodes) or exactly four child nodes. Quadtree nodes typically contain spatial data, such as
map locations, with a maximum capacity of a specified number n.

As long as nodes are not at capacity, they remain leaf nodes; when they reach capacity, they are given four child nodes,
and their data entries are distributed among the children. Quadtrees are well-suited for storing spatial data, as they
can be represented as a grid filled with rectangles that are recursively subdivided into four sub-rectangles. Each
quadtree node corresponds to a rectangle, and each rectangle represents a spatial region.

![img]({{site.url}}/assets/blog_images/2023-04-28-specialized-storage-paradigms/quadtree-exmaple-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-04-28-specialized-storage-paradigms/quadtree-exmaple-dark.png){: .dark }
