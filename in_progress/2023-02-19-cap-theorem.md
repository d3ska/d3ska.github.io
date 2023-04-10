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

<pic> https://www.youtube.com/watch?v=k01mFPqfESw&t=1029s 08:06

similarly it may happen that different users would see different state of the system.

<pic2> https://www.youtube.com/watch?v=k01mFPqfESw&t=1029s 08:30 

Keep in mind that not every system has to have linearizability/ strongest consistency, as it's quite expensive and entails certain sacrifices, and in reality
most of the systems don't need that level of consistency.


<br>


#### Availability 
In the context of a system, signifies that every functioning node reliably provides non-error responses. This ensures that the system consistently delivers valid responses from working nodes, even when some nodes are down. In the CAP theorem, a system that meets this criterion is considered available.

<br>

#### Partition tolerance 
Is the capability of a distributed system to continue functioning despite communication breakdowns, lost connections between nodes, or even data loss. Essentially, it ensures that the system remains operational when facing network partitions, disruptions between nodes, or data loss events.

<img> 12:00  https://www.youtube.com/watch?v=k01mFPqfESw&t=1029s





#### CP (Consistency and Partition tolerance)

TBC...
