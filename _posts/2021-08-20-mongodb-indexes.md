---
title: "MongoDB Indexes"
categories:
  - Blog
tags:
  - MongoDB
  - Indexes
  - efficiency
---

### Why use indexes?

### For faster queries.

Indexes are data structures, B-Tree to be precise (arrange the data in sorted order) at support the efficient execution of queries in MongoDB. They contain copies of parts of the data in documents to make queries more efficient.
Without indexes, MongoDB must perform a collection scan, i.e. scan every document in a collection, to select those documents that match the query statement. If an appropriate index exists for a query, MongoDB can use the index to limit the number of documents it must inspect.

To create an index in the Mongo Shell, use db.collection.createIndex().

**db.collection.createIndex( {key and index type specification}, {options} )**

The following example creates a single key descending index on the name field:

**db.collection.createIndex( { name: -1 } )**   

The db.collection.createIndex() method only creates an index if an index of the same specification does not already exist.


**Options for All Index Types according to documentation.**

 <table style="width:100%">
  <tr>
    <th>Parameter</th>
    <th>Type</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>background</td>
    <td>boolean</td>
    <td>Optional. Deprecated in MongoDB 4.2.
	Background builds do not block operations on the collection. The default value is false.
    Changed in version 4.2.
    All index builds use an optimized build process that holds the exclusive lock only at the beginning and end of the build process.
	The rest of the build process yields to interleaving read and write operations. MongoDB ignores the background option if specified.</td>
  </tr>
  <tr>
    <td>unique</td>
    <td>boolean</td>
    <td>Optional. Creates a unique index so that the collection will not accept insertion or update of documents where the index key value matches an existing value in the index.
	Specify true to create a unique index. The default value is false.
	The option is unavailable for hashed indexes.</td>
  </tr>
  <tr>
    <td>name</td>
    <td>string</td>
    <td>Optional. The name of the index. If unspecified, MongoDB generates an index name by concatenating the names of the indexed fields and the sort order.</td>
  </tr>
  <tr>
    <td>partialFilterExpression</td>
    <td>document</td>
    <td>Optional. If specified, the index only references documents that match the filter expression.</td>
  </tr>  
  <tr>
    <td>sparse</td>
    <td>boolean</td>
    <td>Optional. If true, the index only references documents with the specified field. These indexes use less space but behave differently in some situations (particularly sorts). The default value is false.</td>
  </tr>
  <tr>
    <td>expireAfterSeconds</td>
    <td>integer</td>
    <td>Optional. Specifies a value, in seconds, as a TTL to control how long MongoDB retains documents in this collection.</td>
  </tr>
  <tr>
    <td>hidden</td>
    <td>boolean</td>
    <td>Optional. A flag that determines whether the index is hidden from the query planner. A hidden index is not evaluated as part of the query plan selection. Default is false.</td>
  </tr>
  <tr>
    <td>storageEngine</td>
    <td>document</td>
    <td>Optional. Allows users to configure the storage engine on a per-index basis when creating an index.</td>
  </tr>  
</table> 
