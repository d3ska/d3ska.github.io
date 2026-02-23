---
title: "Rate Limiting"
categories:
  - System Design
tags:
  - Rate Limiting
---

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
potentially causing disruptions or complete shutdowns. Rate limiting serves as both a security measure and an
essential component for maintaining the robustness, performance, and financial viability of a system.

When a client exceeds the configured rate limit, the server responds with an HTTP **429 Too Many Requests** status code. This signals to the client that it has sent too many requests in a given time window and should back off before retrying. Well-designed APIs also include rate-limiting headers in their responses to help clients self-regulate:

* **X-RateLimit-Limit**: The maximum number of requests allowed in the current window.
* **X-RateLimit-Remaining**: The number of requests still available before the limit is reached.
* **X-RateLimit-Reset**: The time (usually a Unix timestamp) when the rate limit window resets.
* **Retry-After**: Included in 429 responses, indicating how many seconds the client should wait before making another request.


### Rate Limiting Algorithms

Several algorithms are commonly used to implement rate limiting, each with different trade-offs:

* **Token Bucket**: A bucket holds a fixed number of tokens, and tokens are added at a constant rate. Each incoming request consumes one token. If the bucket is empty, the request is rejected. This algorithm allows short bursts of traffic (up to the bucket's capacity) while enforcing a sustained average rate. It is widely used in practice -- for example, in AWS and Stripe APIs.

* **Leaky Bucket**: Incoming requests are placed into a queue (the bucket) and processed at a fixed rate, like water leaking from a bucket at a constant drip. If the queue is full, new requests are dropped. This smooths out bursty traffic into a steady, predictable output rate.

* **Fixed Window Counter**: The timeline is divided into fixed-duration windows (e.g., one-minute intervals), and a counter tracks the number of requests in each window. Once the counter exceeds the limit, subsequent requests are rejected until the next window begins. The drawback is that a burst at the boundary of two windows can allow up to double the intended rate.

* **Sliding Window Log**: Each request's timestamp is logged. When a new request arrives, the system counts all logged requests within the past window duration. This eliminates the boundary burst problem of fixed windows but requires storing individual request timestamps, which can be memory-intensive at high volumes.

* **Sliding Window Counter**: A hybrid of the fixed window counter and sliding window log. It combines the counters from the current and previous windows, weighting the previous window's count by the overlap proportion. This provides a good approximation of a true sliding window with lower memory usage than the log approach.


### Where to Implement Rate Limiting

Rate limiting can be applied at different layers of the system architecture, depending on the requirements:

* **API Gateway level**: Managed gateways like AWS API Gateway, Kong, or Azure API Management offer built-in rate limiting as a configurable feature. This is often the simplest approach for teams using cloud-native architectures, as it requires no custom code and can be applied uniformly across all endpoints.

* **Reverse proxy / middleware level**: Tools like Nginx, HAProxy, or Envoy can enforce rate limits before requests even reach the application. This offloads the work from the application servers and provides a centralized enforcement point.

* **Application level**: Rate limiting logic embedded directly in the application code gives the most flexibility -- for example, applying different limits per user tier or per endpoint. Libraries and frameworks often provide middleware for this purpose. A shared data store like Redis is commonly used to track request counts across multiple application instances.

In practice, many systems combine multiple levels. A global rate limit at the API gateway protects the infrastructure, while application-level limits enforce fine-grained, business-specific rules.


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

