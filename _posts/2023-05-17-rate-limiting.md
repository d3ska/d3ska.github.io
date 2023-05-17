---
title: "Rate Limiting"
categories:
  - System Design
tags:
  - Rate Limiting
---

### Rate Limiting

Rate limiting is a pivotal technique employed to manage and control the number of incoming and outgoing requests
interacting with a system. Not only does it serve as a defensive shield against denial-of-service (DoS) attacks, but it
also facilitates a fair distribution of resources among users, ensuring a smooth and uninterrupted service for all. The
application of rate limiting can be diverse, extending to different levels such as IP-address, user-account, or even a
geographic region. Moreover, this strategy can be implemented in a tiered structure, for example, limiting a particular
network request to 1 per second, 5 per 10 seconds, and 10 per minute.

Beyond its preventative role against cyber threats, rate limiting also brings about other significant benefits. It helps
maintain system stability by preventing overuse or misuse of resources, thus reducing the risk of system failures. It
can also protect sensitive information from being exposed to brute force attacks. Additionally, rate limiting aids in
controlling costs associated with third-party APIs that charge based on the number of requests.

Without rate limiting, systems are left vulnerable to malicious actors who can flood them with superfluous requests,
potentially causing disruptions or complete shutdowns. Thus, rate limiting is not just a security measure, but an
essential component for maintaining the robustness, performance, and financial viability of a system.


### DoS Attack

Denial-of-service attack, abbreviated as DoS attack, is a malevolent endeavor where an attacker aims to render a system
unavailable to its users. This is usually achieved by overloading the system with excessive traffic. Although rate
limiting can prevent certain types of DoS attacks, others might be considerably more challenging to counter.

![img]({{site.url}}/assets/blog_images/2023-05-17-rate-limiting/dos.png)


### DDoS Attack

A distributed denial-of-service attack, or DDoS attack, is a more complex form of a DoS attack. In a DDoS attack, the
overwhelming traffic directed at the target system originates from numerous sources—potentially thousands of
machines—making it significantly more difficult to prevent or mitigate.

![img]({{site.url}}/assets/blog_images/2023-05-17-rate-limiting/ddos.png)


### Examples and Analogies

To visualize the concept of rate limiting, consider a popular restaurant with a maximum seating capacity. If more customers arrive than the restaurant can accommodate, the excess customers are asked to wait, ensuring the restaurant can effectively serve those already seated. In this scenario, the restaurant's seating capacity is akin to the rate limit, and the customers represent incoming requests to a system.

For DoS and DDoS attacks, imagine a highway filled with cars. In a DoS attack, it's as if a single truck has intentionally blocked all lanes, stopping the flow of traffic. In a DDoS attack, it's like hundreds of cars from different directions converging on the same highway, causing a massive traffic jam.


### Mitigation Strategies

For DoS and DDoS attacks, rate limiting is one of many defensive measures. Here are a few more strategies:

* **Anomaly detection:** Systems can be configured to identify unusual traffic patterns or request rates and respond accordingly.

* **IP blacklisting:** IPs known to be sources of malicious traffic can be blocked from making requests.

* **Redundancy:** Having multiple servers can help to ensure that if one server becomes overwhelmed, others can pick up the load.

* **DDoS mitigation services:** Third-party services specialize in detecting and neutralizing DDoS attacks before they reach your network.