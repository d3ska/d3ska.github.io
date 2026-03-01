---
title: "Availability"
categories:
  - System Design
tags:
  - Availability
  - Reliability
  - SLA
  - Redundancy
---

On June 2, 2019, Google Cloud Platform went down for four hours and twenty-five minutes. Snapchat, Shopify, Vimeo, Discord, and thousands of smaller services went with it. A single provider's outage cascaded into lost revenue, broken workflows, and frustrated users across the globe. That incident is a textbook reminder of why **availability**, the percentage of time a system is operational and able to serve its intended function, is one of the most important properties in system design.

Availability is really two things at once. It is a measure of **fault tolerance** (what happens when a server crashes, a disk dies, or a network partition splits your cluster?) and it is a contractual promise (how much uptime do you guarantee to your users?). Both perspectives matter, and they feed into each other.

Consider airplane flight control software. If that system goes down mid-flight, lives are at risk. Stock exchanges face a different kind of stakes: minutes of downtime erode trust and cost real money. Even consumer products like YouTube or Twitter serve hundreds of millions of daily users, so even brief outages make headlines. The required level of availability varies, but almost every system carries an implicit expectation that it will be there when users need it.

### Measuring Availability: The Nines

Availability is typically expressed as a percentage of uptime over a year. The industry shorthand uses **"nines"**: 99% is "two nines," 99.9% is "three nines," and so on. The table below shows the levels you will encounter most often.

| Availability | Common Name | Downtime per Year | Downtime per Month | Downtime per Day |
|---|---|---|---|---|
| 99% | Two nines | 3.65 days | 7.31 hours | 14.40 minutes |
| 99.9% | Three nines | 8.77 hours | 43.83 minutes | 1.44 minutes |
| 99.95% | Three and a half nines | 4.38 hours | 21.92 minutes | 43.20 seconds |
| 99.99% | Four nines | 52.60 minutes | 4.38 minutes | 8.64 seconds |
| 99.999% | Five nines | 5.26 minutes | 26.30 seconds | 864 milliseconds |

The jump from two nines to three nines looks small on paper (99% to 99.9%), but it means going from over three and a half days of annual downtime to under nine hours. Five nines (99.999%) allows roughly five minutes of downtime per year. That is the gold standard, but very few systems actually need it.

### What Cloud Providers Actually Promise

Major cloud providers publish SLAs for their core services, and the numbers are instructive:

| Provider / Service | SLA Target |
|---|---|
| AWS S3 | 99.99% |
| AWS EC2 (single instance) | 99.99% |
| Google Compute Engine | 99.99% |
| Azure VMs (single instance) | 99.95% |
| Azure VMs (availability set) | 99.99% |

When a provider fails to meet the published SLA, customers are typically entitled to **service credits**, partial refunds applied to future bills. AWS, for example, offers a 10% credit if S3 drops below 99.9% and a 25% credit below 99.0%. These credits are not automatic; you usually have to file a claim. But the point is that SLAs carry real financial consequences.

Notice that even the largest providers commit to four nines for most compute services, not five. That should calibrate your own expectations.

### The Cost of Each Additional Nine

There is an exponential relationship between availability and cost. Moving from 99.9% to 99.99% does not cost 0.09% more. In practice it can cost roughly **10x the infrastructure spend**, because each additional nine demands more redundancy, more sophisticated failover, cross-region replication, and more engineering time to test and maintain all of it.

Five nines of availability is achievable, but it requires investment in redundant infrastructure at every layer, automated failover that works without human intervention, and an on-call culture that can respond to incidents in seconds. For most applications, three nines or three and a half nines (99.9% to 99.95%) is the right target. Spending 10x your infrastructure budget to shave a few hours of annual downtime only makes sense when the cost of that downtime truly justifies it (financial trading, healthcare, aviation).

### SLAs, SLOs, and SLIs

These three terms are often confused, but they form a clear hierarchy.

A **Service Level Indicator (SLI)** is the raw metric you measure: request latency at the 99th percentile, error rate, or uptime percentage over a rolling window. SLIs are the data.

A **Service Level Objective (SLO)** is the internal target you set against an SLI. For example, "99.95% of requests will return a successful response within 200ms over any 30-day window." SLOs are your engineering goals.

A **Service Level Agreement (SLA)** is the external, often legally binding contract with your customers. It defines what happens when the SLO is not met: service credits, refunds, or contract termination. SLAs are typically set slightly below the SLO to provide a safety margin. If your internal SLO is 99.95%, your published SLA might promise 99.9%.

#### Error Budgets

An **error budget** is the inverse of the SLO. If your SLO is 99.95% availability over 30 days, you have a budget of 0.05% of that period (roughly 21 minutes and 55 seconds) for acceptable downtime. As long as you stay within budget, teams can ship features aggressively. When the budget is nearly exhausted, the focus shifts to reliability work. This concept, popularized by Google's SRE practice, turns availability from a vague aspiration into a concrete, trackable resource that balances feature velocity against stability.

### Eliminating Single Points of Failure

A **single point of failure (SPOF)** is any component whose failure brings down the entire system. Achieving high availability starts with identifying every SPOF and adding redundancy.

Consider a typical three-tier web application:

```
Clients -> Load Balancer -> App Servers -> Database
```

In the simplest deployment, each tier is a SPOF. Here is how to address each one:

1. **Load balancer**: Deploy a pair of load balancers in an active/passive or active/active configuration. Use a virtual IP (VIP) or DNS-based failover so that if one load balancer dies, traffic routes to the other. Cloud providers handle this transparently with managed load balancers (AWS ALB, GCP Cloud Load Balancing).

2. **Application servers**: Run multiple instances behind the load balancer. If one crashes, the load balancer stops routing traffic to it. Kubernetes takes this further by automatically restarting failed pods and rescheduling them onto healthy nodes.

3. **Database**: Set up primary-replica replication. The primary handles writes, one or more replicas handle reads, and if the primary fails, a replica is promoted. Managed services like Amazon RDS Multi-AZ automate this promotion.

4. **The load balancer's load balancer problem**: You might worry that even the redundant pair of load balancers is a SPOF. In practice, DNS round-robin, anycast routing, or cloud-managed services handle this layer so you rarely need to build it yourself.

The key insight is that redundancy must be applied at every layer. A system with three redundant app servers but a single database is only as available as that database.

Worth noting: SPOFs are not always technical. A single team member who holds all the institutional knowledge about a critical system is also a SPOF. Document runbooks, share on-call responsibilities, and cross-train.

### Passive Redundancy

In **passive redundancy**, a primary component is backed up by a standby that takes over when the primary fails. There are three common standby strategies:

- **Cold standby**: The backup is offline and powered down. When the primary fails, the standby must be started, configured, and synchronized before serving traffic. Recovery time is the longest, often minutes or more. This is the cheapest option.
- **Warm standby**: The backup is running and periodically synchronized with the primary, but not actively serving traffic. On failure, it needs a brief promotion step (updating DNS, promoting a database replica). Recovery is faster, typically seconds to a few minutes.
- **Hot standby**: The backup is running, fully synchronized, and ready to take over instantly. Failover can happen automatically with near-zero downtime. This is the most expensive option but provides the fastest recovery.

Examples include redundant power supplies, database replication with a standby replica, or a backup server that takes over when the primary fails.

### Active Redundancy

In **active redundancy**, multiple components work in parallel and share the workload simultaneously. If one fails, the remaining components absorb its share of the traffic. Active redundancy can also improve performance and throughput under normal conditions, since all nodes contribute rather than sitting idle.

Examples include multiple load-balanced application servers, RAID configurations for data storage, and redundant network connections across multiple ISPs.

### High Availability Technologies

In practice, several widely adopted technologies help achieve high availability:

- **Load balancers** (Nginx, HAProxy, or cloud-native solutions like AWS ALB) distribute traffic across multiple backend servers and automatically route around unhealthy instances.
- **Database replication** (primary-replica setups in PostgreSQL, MySQL, or managed services like Amazon RDS Multi-AZ) ensures a copy of the data is always available for failover.
- **Container orchestration** platforms like Kubernetes maintain desired pod replica counts, automatically restarting failed containers and rescheduling them onto healthy nodes.

### Monitoring and Alerting

Redundancy alone does not guarantee high availability. Without proper **monitoring and alerting**, failures can go undetected and recovery processes may never trigger.

At a minimum, an HA system should include health checks for all critical components, dashboards that surface real-time system status, and automated alerts (via tools like Prometheus, Grafana, Datadog, or PagerDuty) that notify on-call engineers when thresholds are breached. The faster a team detects a failure, the faster it can respond, whether automatically or through manual intervention.

It is also important to have rigorous incident response processes in place. If servers crash, you need clear runbooks, defined escalation paths, and on-call rotations that ensure someone is always available to act. Availability is not just a technical property; it is an organizational one.

### Trade-offs

High availability is not free. Every additional nine comes with costs that must be weighed against the value of the uptime it provides.

- **Cost**: Higher availability requires redundant servers, storage, networking infrastructure, multi-region deployments, and the engineering hours to build and maintain all of it. As discussed above, each additional nine roughly multiplies infrastructure cost by 10x.

- **Complexity**: Implementing failover, health checking, leader election, and data replication adds architectural complexity. More moving parts means more potential failure modes and harder debugging.

- **Performance**: Replicating data across nodes or geographic regions adds latency. Synchronous replication, which is needed for strong consistency, is especially expensive in terms of response time.

- **Consistency**: This is where the **CAP theorem** becomes relevant. In a distributed system, a network partition forces a choice between availability and consistency. Most highly available systems choose availability and accept **eventual consistency**, meaning replicas may serve slightly stale data during failures. This is an acceptable trade-off for many applications (social media feeds, product catalogs), but not for others (bank account balances, inventory counts for the last item in stock). Understanding where your application falls on this spectrum is critical.

The right availability target depends on what your system does, who depends on it, and what downtime actually costs. A payment processing system like Stripe needs near-perfect uptime for its transaction pipeline, but its internal analytics dashboard can tolerate occasional blips. Design for the availability each component actually requires, not the highest number you can think of.

### References

* [AWS Service Level Agreements](https://aws.amazon.com/legal/service-level-agreements/)
* [Google Cloud Service Level Agreements](https://cloud.google.com/terms/sla)
* [Summary of the Amazon S3 Service Disruption in the Northern Virginia (US-EAST-1) Region](https://aws.amazon.com/message/41926/) (2017 outage post-mortem)

> **Related posts**: [Load Balancing](/posts/load-balancers/), [Replication And Sharding](/posts/replication-and-sharding/), [Network Protocols](/posts/network-protocols/)
