---
title: "CAP Theorem"
categories:
  - Blog
tags:
  - Databases
  - CAP
---


#### What is CAP theorem?

That statement is quite old, as it was formally introduced in 2000 by Eric Brewer,  on conference named 'Principles of Distributed Computing' and that presentation was named 'Towards robust distributed systems'.
That's why it's alternative name is a Brewer theorem.
The presentation can be found [here](https://people.eecs.berkeley.edu/~brewer/cs262b-2004/PODC-keynote.pdf).

Generally it's statement sounds rather simple, namely every distributed system can guarantee two out of three following characteristics

Lets deep dive a bit in each of them.

<br>

#### Consistency 
In a system is not a binary concept; it's not simply a matter of a system being consistent or inconsistent. Instead, consistency exists along a spectrum ranging from more relaxed to stricter levels. The position of a system on this spectrum influences whether anomalies related to read and write operations may occur.

Thus, it's not just about strong and eventual consistency, as there are variations in between these two extremes.

In the context of the CAP theorem, the highest or strongest level of consistency is typically considered, which provides linearizability. This means that if operation B starts after operation A has ended, then operation B should perceive the system in a state that reflects the changes made by operation A or a newer state.

In simpler terms, a distributed system achieves linearizability when it behaves, from the end user's perspective, as if it were running on a single instance. This means there is no network involvement and the complexities arising from network communication between different services are eliminated. When a system behaves in this manner, it is said to exhibit linearizability.


An example may be replicated database, with the master slave/leader follower topology.

Basically it works in the following way, we may read data from each replica but we can make writes just to the leader, and then
based on the approach, leader in synchronized way waits until leaders update their state and i.e. return ACK, and then response to the client.

The second approach is that we write to the leader that response us immedieatly and leaders are updated in the 'background' in async way, fire-forget we may say.
Usually it would be updated in a moment, but as some obstacles may occur related to the network, user may see the stale data, in other words outdated.

![img]({{site.url}}/assets/blog_images/2023-02-19-cap-theorem/eventual-consitency.png)

similarly it may happen that different users would see different state of the system.

![img]({{site.url}}/assets/blog_images/2023-02-19-cap-theorem/eventual-consitency-2.png)


Keep in mind that not every system has to have linearizability/ the strongest level consistency, as it's quite expensive and entails certain sacrifices, and in reality
most of the systems don't need that level of consistency.


<br>


#### Availability 
In the context of a system, signifies that every functioning node reliably provides non-error responses. This ensures that the system consistently delivers valid responses from working nodes, even when some nodes are down. In the CAP theorem, a system that meets this criterion is considered available.

<br>

#### Partition tolerance 
Is the capability of a distributed system to continue functioning despite communication breakdowns, lost connections between nodes, or even data loss. Essentially, it ensures that the system remains operational when facing network partitions, disruptions between nodes, or data loss events.


<br>


#### CP (Consistency and Partition tolerance)

In the scenario where a request is sent to change a nickname in a system with synchronized replication, the leader node receives the request and updates its state. It then sends a synchronized request to the follower nodes to update their data. The follower nodes return an acknowledgment (ACK) to the leader node, and once the transaction is committed, the user receives a response indicating that the transaction was successful.

Regardless of which node receives the request, the returned data will be consistent due to the synchronized replication.


<br>

If partitioning occurs between the leader and follower nodes in this model, the follower will not receive the request sent by the leader to update its data. As a result, the leader node will not receive the acknowledgment from the follower, which is a part of the replication process. The leader node will eventually rollback the transaction and return a response indicating that the operation failed.

![img]({{site.url}}/assets/blog_images/2023-02-19-cap-theorem/partitioning-example.png)

From the perspective of the CAP theorem, this model guarantees consistency, as the transaction was not committed. When requesting data from either the leader or follower nodes, the data will be consistent and have the same nickname. The model also guarantees partitioning tolerance, as the system remains operational even if the connection between the leader and follower nodes is lost. However, the model does not guarantee availability, as even though the node is operational, it returns an error. Therefore, from a CAP perspective, this model guarantees CP (consistency and partition tolerance).e was operational, it returns error, so from CAP perspective that model guarantee **CP**.