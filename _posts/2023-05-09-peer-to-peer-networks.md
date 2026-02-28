---
title: "Peer-To-Peer Networks"
categories:
  - System Design
tags:
  - Peer-To-Peer Networks
  - P2P
  - Distributed Systems
  - Networking
---

In a traditional client-server architecture, a central server owns the data and clients request it. If the server goes down or gets overloaded, everyone is affected. A **Peer-to-Peer** (P2P) network flips this model: every node (called a **peer**) acts as both a client and a server. Peers share resources directly with each other, distributing the workload so that no single machine is a bottleneck or a single point of failure.

### How P2P Networks Are Used

**File sharing and content distribution.** BitTorrent is the most well-known P2P protocol. When you download a file via BitTorrent, you do not pull it from a single server. Instead, you download different pieces from multiple peers simultaneously, and you upload pieces you already have to other peers. The more popular a file is, the faster it downloads, because more peers are seeding it. This is the opposite of client-server, where popular files overload the server.

**Blockchain and cryptocurrencies.** Bitcoin, Ethereum, and other blockchain networks are P2P systems. Every node maintains a copy of the ledger and validates new transactions. There is no central authority. Consensus protocols (Proof of Work, Proof of Stake) replace the trust that a central server would normally provide.

**Distributed storage.** The InterPlanetary File System (IPFS) uses P2P for decentralized file storage. Files are split into blocks, each identified by its content hash, and distributed across participating nodes. Retrieving a file means fetching blocks from whichever peers have them, providing redundancy and resilience.

**Messaging.** P2P enables communication without centralized servers. Briar routes messages through the Tor network without relying on any central infrastructure, and Tox provides direct peer-to-peer encrypted messaging. Note that while apps like Signal use end-to-end encryption, they still rely on centralized servers for message routing and are not true P2P systems.

### Structured vs Unstructured Networks

P2P networks fall into two categories based on how peers are organized.

**Structured networks** impose a specific topology using a **Distributed Hash Table** (DHT). Each piece of data maps to a specific node (or set of nodes) based on its key. When a peer wants to find data, it does not need to ask everyone. It can route the query through the DHT and reach the responsible node in O(log N) hops, where N is the number of peers. Kademlia (used by BitTorrent and Ethereum) and Chord are well-known DHT implementations.

The trade-off is maintenance overhead: when peers join or leave, the DHT must be updated. But the lookup efficiency makes structured networks the right choice for systems where data needs to be found reliably.

**Unstructured networks** have no predefined topology. Peers form connections randomly, and there is no mapping between data and nodes. To find something, a peer floods a query to its neighbors, who forward it to their neighbors, and so on. Early Gnutella worked this way. Unstructured networks are simple to build and resilient to churn (peers joining and leaving frequently), but searching for rare items is inefficient because there is no guarantee that a query will reach the peer holding the data.

### Gossip Protocol

The **Gossip Protocol** (also called epidemic protocol) is how many P2P systems propagate information without centralized coordination. It works like rumors spreading through a crowd:

1. A peer periodically selects one or more random neighbors.
2. It shares its current state or recent updates with them.
3. Those neighbors do the same in their next cycle, sharing the information with their own randomly chosen neighbors.

Because each round roughly doubles the number of informed nodes, information spreads exponentially, reaching the entire network in O(log N) rounds. This makes gossip protocols remarkably scalable: even if some peers fail or messages are lost, the redundancy of multiple independent paths ensures that information eventually propagates everywhere.

In practice, gossip protocols handle membership management (tracking which peers are alive), failure detection, and state dissemination. Apache Cassandra uses gossip internally for cluster state propagation, and many blockchain networks use gossip-based transaction broadcasting.

### Challenges

P2P networks come with a unique set of challenges:

* **NAT traversal.** Most peers sit behind Network Address Translation (NAT) devices, making it difficult for external peers to initiate connections. Techniques like UDP hole punching, STUN, and TURN servers are used to work around this.
* **Free-riding.** Some peers consume resources (downloading files) without contributing back (not uploading or seeding). This degrades network performance. Incentive mechanisms like BitTorrent's tit-for-tat strategy help: peers that upload more get better download speeds.
* **Sybil attacks.** A malicious actor creates a large number of fake identities to gain disproportionate influence over routing, voting, or reputation systems. Defenses include proof-of-work identity costs and trust-based reputation systems.
* **Churn.** Peers frequently join and leave, forcing the system to constantly update routing tables and redistribute data. High churn rates can lead to data unavailability and increased maintenance overhead. Replication (storing data on multiple peers) is the primary mitigation.

### Centralized Discovery

Even in P2P networks, peers need a way to find each other initially. Two common approaches:

* **Tracker servers.** A central server maintains a list of peers participating in a particular swarm or network. BitTorrent trackers are the classic example. The tracker is not involved in data transfer, only in helping peers discover each other. If the tracker goes down, existing connections continue, but new peers cannot join.
* **Bootstrap nodes.** A small set of well-known nodes that new peers contact to learn about other peers in the network. Once a peer has discovered enough neighbors, it no longer needs the bootstrap nodes. This is how most DHT-based systems handle initial peer discovery.

Neither approach contradicts the P2P model. The data transfer and processing remain distributed; only the initial discovery step involves a known endpoint.
