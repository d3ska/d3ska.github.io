---
title: "Availability"
categories:
  - System Design
tags:
  - Availability
---

#### What is availability?

Availability can be thought of in a couple of ways. One way to consider it is how resistant a system is to failures. For instance, what happens if a server in your system fails? What happens if your database fails? Will your system go down completely, or will it still be operational? This is often described as a system's fault tolerance.

Another way to think about availability is the percentage of time in a given period, like a month or a year, during which a system is operational and capable of satisfying its primary functions.

Availability is crucial to consider when evaluating a system. In today's world, most systems have an implied guarantee of availability.

Imagine a system supporting airplane software, which allows an airplane to function properly. If that system were to go down while an airplane is flying, it would be absolutely unacceptable. Similarly, consider stock or crypto exchange systems, where downtime could lead to customers losing trust and money.

Even with less critical examples like YouTube or Twitter, downtime would be detrimental, as hundreds of millions of people use these platforms daily.

Another example might be cloud providers, such as AWS, Azure, or GCP, also need to maintain high availability. If parts of their systems go down, it affects all the businesses and customers relying on their services. For example, in summer 2019, Google Cloud Platform experienced a significant outage that lasted for a few hours, affecting many businesses, including Vimeo.

In summary, availability is of great importance in system design and operations.

<br>

#### How to measure availability?

Availability is usually measured as the percentage of a system's uptime in a given year.

For instance, if a system is up and operational for half of an entire year, then we can say that the system has 50% availability, which is quite poor.

In the industry, most services or systems aim for high availability, so we often measure availability in terms of "nines" rather than exact percentages.

"Nines" are essentially percentages, but they specifically represent percentages with the number nine. For example, if a system has 99% availability, we can say that the system has two nines of availability because the number nine appears twice in this percentage. Similarly, 99.9% would be considered three nines of availability, and so on.

This terminology is a standard way that people discuss availability in the industry.

Below you can find a chart from Wikipedia that showcases a range of popular availability percentages, which can help illustrate the differences between various levels of system availability.

<table style="width:100%">
  <tr>
    <th><b>Availability %</b></th>
    <th><b>Downtime per year</b></th>
    <th><b>Downtime per month</b></th>
    <th><b>Downtime per week</b></th>
    <th><b>Downtime per day</b></th>
  </tr>
  <tr>
    <td>55.5555555% ("nine fives")</td>
    <td>162.33 days</td>
    <td>13.53 days</td>
    <td>74.92 hours</td>
    <td>10.67 hours</td>
  </tr>  
  <tr>
    <td><b>90% ("one nine")</b></td>
    <td><b>36.53 days</b></td>
    <td><b>73.05 hours</b></td>
    <td><b>16.80 hours</b></td>
    <td><b>2.40 hours</b></td>
  </tr>
  <tr>
    <td>95% ("one and half nines")</td>
    <td>18.26 days</td>
    <td>36.53 hours</td>
    <td>8.40 hours</td>
    <td>1.20 hours</td>
  </tr>
  <tr>
    <td><b>97%</b></td>
    <td><b>10.96 days</b></td>
    <td><b>21.92 hours</b></td>
    <td><b>5.04 hours</b></td>
    <td><b>43.20 minutes</b></td>
  </tr>
  <tr>
    <td>98%</td>
    <td>7.31 days</td>
    <td>14.61 hours</td>
    <td>3.36 hours</td>
    <td>28.80 minutes</td>
  </tr>
  <tr>
    <td><b>99% ("two nines")</b></td>
    <td><b>3.65 days</b></td>
    <td><b>7.31 hours</b></td>
    <td><b>1.68 hours</b></td>
    <td><b>14.40 minutes</b></td>
  </tr>
  <tr>
    <td>99.5% ("two and a half nines")</td>
    <td>1.83 days</td>
    <td>3.65 hours</td>
    <td>50.40 minutes</td>
    <td>7.20 minutes</td>
  </tr>
  <tr>
    <td>99.8%</td>
    <td>17.53 hours</td>
    <td>87.66 minutes</td>
    <td>20.16 minutes</td>
    <td>2.88 minutes</td>
  </tr>
  <tr>
    <td><b>99.9% ("three nines")</b></td>
    <td><b>8.77 hours</b></td>
    <td><b>43.83 minutes</b></td>
    <td><b>10.08 minutes</b></td>
    <td><b>1.44 minutes</b></td>
  </tr>
  <tr>
    <td>99.95% ("three and a half nines)</td>
    <td>4.38 hours</td>
    <td>21.92 minutes</td>
    <td>5.04 minutes</td>
    <td>43.20 seconds</td>
  </tr>
  <tr>
    <td><b>99.99% ("fours nines")</b></td>
    <td><b>52.60 minutes</b></td>
    <td><b>4.38 minutes</b></td>
    <td><b>1.01 minutes</b></td>
    <td><b>8.64 seconds</b></td>
  </tr>
  <tr>
    <td>99.995% ("four and a half nines")</td>
    <td>26.30 minutes</td>
    <td>2.19 minutes</td>
    <td>30.24 seconds</td>
    <td>4.32 seconds</td>
  </tr>
  <tr>
    <td><b>99.999% ("five nines")</b></td>
    <td><b>5.26 minutes</b></td>
    <td><b>26.30 seconds</b></td>
    <td><b>6.05 seconds</b></td>
    <td><b>864.00 milliseconds</b></td>
  </tr>
  <tr>
    <td><b>99.9999% ("six nines")</b></td>
    <td><b>31.56 seconds</b></td>
    <td><b>2.63 seconds</b></td>
    <td><b>604.80 milliseconds</b></td>
    <td><b>86.40 milliseconds</b></td>
  </tr>
  <tr>
    <td><b>99.99999% ("seven nines")</b></td>
    <td><b>3.16 seconds</b></td>
    <td><b>262.98 milliseconds</b></td>
    <td><b>60.48 milliseconds</b></td>
    <td><b>8.64 milliseconds</b></td>
  </tr>
  <tr>
    <td><b>99.999999% ("eight nines")</b></td>
    <td><b>315.58 milliseconds</b></td>
    <td><b>26.30 milliseconds</b></td>
    <td><b>6.05 milliseconds</b></td>
    <td><b>864.00 microseconds</b></td>
  </tr>
  <tr>
    <td><b>99.9999999% ("nine nines")</b></td>
    <td><b>31.56 milliseconds</b></td>
    <td><b>2.63 milliseconds</b></td>
    <td><b>604.80 microseconds</b></td>
    <td><b>86.40 microseconds</b></td>
  </tr>
</table>


As you can see, even though 99% availability seems impressive, being down for three and a half days or more per year is still quite problematic and might be considered unacceptable. For systems that involve life-and-death situations, such downtime is undoubtedly unacceptable. Even for services like Facebook or YouTube, which serve billions of users, that amount of downtime is too high.

Five nines of availability (99.999%) is often considered the gold standard for availability. If your system achieves this level of availability, it can be regarded as a highly available system.

<br>
A (Service Level Agreement) and SLO (Service Level Objective) are related concepts used to define and measure the quality of service provided by a service provider.

#### SLA (Service Level Agreement)
An SLA is a formal, legally binding contract between a service provider and a client that outlines the expected level of service, performance metrics, and responsibilities of both parties. It specifies measurable targets such as availability, response time, and throughput, as well as consequences for not meeting these targets, such as refunds or service credits.

#### SLO (Service Level Objective)
An SLO is a specific, measurable goal or target within an SLA that defines a particular aspect of the service quality. SLOs serve as benchmarks to evaluate the service provider's performance and ensure that the agreed-upon service levels are met. Examples of SLOs include system availability (e.g., 99.9% uptime), maximum response time for support requests, or error rates.






<br>
While availability is a critical consideration in system design, it's not always of utmost importance. Achieving five nines of availability (99.999%) isn't always necessary, as high availability comes with trade-offs. Ensuring a high level of availability can be challenging and resource-intensive.

When designing a system, it's essential to carefully evaluate whether your system requires high availability or if only specific components need it. This assessment helps allocate resources effectively and prioritize the most critical components for high availability while allowing other parts to function with lower levels of redundancy. Ultimately, balancing the need for high availability with the system's overall requirements and constraints is crucial for efficient and sustainable system design.

For example, consider a payment processing system like Stripe or Visa. In such a system, the transaction processing component is critical and requires high availability to ensure uninterrupted service for customers making payments. Downtime or failures in this component could result in lost revenue and negatively impact the user experience.

On the other hand, there may be secondary components like an analytics dashboard or reporting tools that, while important, do not need the same level of high availability. These components can tolerate occasional downtime without significantly impacting the overall system performance or user experience.

<br>


#### How to achieve HA system?

Conceptually, achieving a high availability (HA) system is straightforward. First and foremost, you need to ensure that your system doesn't have single points of failure (SPOFs). These are points that, if they fail, the entire system fails. Keep in mind that even team members with specialized knowledge can be considered SPOFs.

To eliminate SPOFs, you can introduce redundancy. Redundancy involves duplicating, triplicating, or even further multiplying certain parts of the system.

For instance, if you have a simple system where clients interact with a server and the server communicates with a database, the server is a single point of failure. If it gets overloaded or goes down for any reason, the entire system fails.

To enhance availability and eliminate the SPOF, you can add more servers and a load balancer between clients and servers to distribute the load across multiple servers.

However, the load balancer itself could become a single point of failure, so it should also be replicated and run on multiple servers.




#### Passive Redundancy
In passive redundancy, a primary system/component is backed up by a secondary (standby) system/component, which remains idle until the primary fails. Upon failure, the secondary takes over the operation. This approach is also called "hot standby" or "failover." Examples include redundant power supplies, database replication with a standby database, or a backup server that takes over when the primary server fails.

#### Active Redundancy 
In active redundancy, multiple systems/components work in parallel and share the workload, ensuring continued operation even if one or more components fail. Active redundancy may also provide load balancing and improved performance. Examples include redundant network connections, RAID configurations for data storage, or multiple load-balanced servers providing the same service.

It's also important to mention that you'll want to have a rigorous process in place to handle system failures, as they might require human intervention. For instance, if servers in your system crash, you'll need a person to bring them back online, and it's crucial to establish processes that ensure timely recovery. So, keeping that in mind is essential for maintaining high availability in your system



