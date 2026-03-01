---
title: "Rate Limiting"
categories:
  - System Design
tags:
  - Rate Limiting
  - API Security
  - DoS
  - DDoS
---

Imagine your API starts getting hammered. Maybe it is a misbehaving client retrying in a tight loop, a bot scraping every endpoint, or an actual attack. Without any controls in place, a single source of excessive traffic can exhaust your server resources, degrade performance for legitimate users, and even bring the whole system down.

**Rate limiting** is the technique of controlling how many requests a client can make to a system within a given time window. It is one of the most fundamental defense mechanisms in system design, serving multiple purposes at once: protecting against denial-of-service attacks, ensuring fair resource distribution among users, preventing abuse of expensive operations, and controlling costs when your system depends on third-party APIs that charge per request.

Rate limits can be applied at different granularities (per IP address, per user account, per API key, or even per geographic region) and can be structured in tiers. For example, you might allow 1 request per second, 5 per 10 seconds, and 10 per minute for the same endpoint.

When a client exceeds the configured rate limit, the server responds with an HTTP **429 Too Many Requests** status code. This signals to the client that it has sent too many requests in a given time window and should back off before retrying. Well-designed APIs also include rate-limiting headers in their responses to help clients self-regulate:

* **X-RateLimit-Limit**: The maximum number of requests allowed in the current window.
* **X-RateLimit-Remaining**: The number of requests still available before the limit is reached.
* **X-RateLimit-Reset**: The time (usually a Unix timestamp) when the rate limit window resets.
* **Retry-After**: Included in 429 responses, indicating how many seconds the client should wait before making another request.


### Rate Limiting as a Defense Against DoS and DDoS

One of the primary motivations for rate limiting is defending against denial-of-service attacks. In a **DoS attack**, a single attacker floods the system with excessive traffic, aiming to make it unavailable to legitimate users. Rate limiting can be effective here because all the malicious traffic originates from one source, making it straightforward to detect and throttle.

![img]({{site.url}}/assets/blog_images/2023-05-17-rate-limiting/dos-lighty-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-05-17-rate-limiting/dos-lighty-dark.png){: .dark }

A **DDoS attack** (Distributed Denial-of-Service) is a more complex variant where the traffic comes from thousands of different machines, often a botnet. This makes it significantly harder to mitigate with rate limiting alone, because each individual source may appear to be sending a reasonable number of requests. Per-IP rate limits help, but they are not sufficient on their own. Defending against DDoS typically requires a combination of rate limiting, traffic analysis, CDN-level protection (e.g., Cloudflare, AWS Shield), and IP reputation systems.

![img]({{site.url}}/assets/blog_images/2023-05-17-rate-limiting/ddos-lighty.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-05-17-rate-limiting/ddos-lighty-dark.png){: .dark }

The key takeaway is that rate limiting is a necessary layer of defense, but for distributed attacks, it needs to be part of a broader strategy.


### Rate Limiting Algorithms

Several algorithms are commonly used to implement rate limiting, each with different trade-offs:

* **Token Bucket**: A bucket holds a fixed number of tokens, and tokens are added at a constant rate. Each incoming request consumes one token. If the bucket is empty, the request is rejected. This algorithm allows short bursts of traffic (up to the bucket's capacity) while enforcing a sustained average rate. It is widely used in practice -- for example, in AWS and Stripe APIs.

* **Leaky Bucket**: Incoming requests are placed into a queue (the bucket) and processed at a fixed rate, like water leaking from a bucket at a constant drip. If the queue is full, new requests are dropped. This smooths out bursty traffic into a steady, predictable output rate.

* **Fixed Window Counter**: The timeline is divided into fixed-duration windows (e.g., one-minute intervals), and a counter tracks the number of requests in each window. Once the counter exceeds the limit, subsequent requests are rejected until the next window begins. The drawback is that a burst at the boundary of two windows can allow up to double the intended rate.

* **Sliding Window Log**: Each request's timestamp is logged. When a new request arrives, the system counts all logged requests within the past window duration. This eliminates the boundary burst problem of fixed windows but requires storing individual request timestamps, which can be memory-intensive at high volumes.

* **Sliding Window Counter**: A hybrid of the fixed window counter and sliding window log. It combines the counters from the current and previous windows, weighting the previous window's count by the overlap proportion. This provides a good approximation of a true sliding window with lower memory usage than the log approach.

#### Choosing the Right Algorithm

There is no single best algorithm. The right choice depends on your traffic patterns and operational constraints:

| Algorithm | Burst Tolerance | Memory Usage | Accuracy | Complexity |
|---|---|---|---|---|
| Token Bucket | High (up to bucket size) | Low (counter + timestamp) | Good | Low |
| Leaky Bucket | None (smooths traffic) | Low (counter + queue) | Good | Low |
| Fixed Window Counter | Moderate (boundary burst issue) | Very low (one counter per window) | Approximate | Very low |
| Sliding Window Log | None | High (stores every timestamp) | Exact | Moderate |
| Sliding Window Counter | Moderate | Low (two counters) | Good approximation | Low |

If your system needs to tolerate occasional bursts (e.g., a user loading a dashboard that fires 10 API calls at once), the **token bucket** is usually the best fit. If you need a smooth, predictable output rate (e.g., writing to a rate-limited downstream service), the **leaky bucket** is preferable. For simple use cases where near-exact accuracy is not critical, the **fixed window counter** is the easiest to implement. The **sliding window counter** offers a good middle ground: better accuracy than fixed windows with minimal additional complexity. The **sliding window log** provides exact accuracy but at the cost of higher memory usage, which may be impractical at scale.


### Token Bucket Implementation in Java

The token bucket is the most commonly used algorithm in practice, so it is worth understanding how it works in code. Here is a thread-safe implementation:

```java
public class TokenBucketRateLimiter {

    private final int maxTokens;
    private final double refillRatePerSecond;
    private double availableTokens;
    private long lastRefillTimestamp;

    public TokenBucketRateLimiter(int maxTokens, double refillRatePerSecond) {
        this.maxTokens = maxTokens;
        this.refillRatePerSecond = refillRatePerSecond;
        this.availableTokens = maxTokens;
        this.lastRefillTimestamp = System.nanoTime();
    }

    public synchronized boolean tryAcquire() {
        refill();
        if (availableTokens >= 1) {
            availableTokens -= 1;
            return true;
        }
        return false;
    }

    private void refill() {
        long now = System.nanoTime();
        double elapsedSeconds = (now - lastRefillTimestamp) / 1_000_000_000.0;
        double tokensToAdd = elapsedSeconds * refillRatePerSecond;
        availableTokens = Math.min(maxTokens, availableTokens + tokensToAdd);
        lastRefillTimestamp = now;
    }
}
```

Usage is straightforward. You create a limiter and call `tryAcquire()` before processing each request:

```java
// Allow 10 requests per second, with bursts up to 20
TokenBucketRateLimiter limiter = new TokenBucketRateLimiter(20, 10.0);

public void handleRequest(HttpServletRequest request, HttpServletResponse response) {
    if (!limiter.tryAcquire()) {
        response.setStatus(429);
        response.setHeader("Retry-After", "1");
        return;
    }
    // process the request normally
}
```

This implementation works well for a single application instance. For distributed systems, you would replace the in-memory state with a shared store like Redis, which I discuss in the next section.


### Distributed Rate Limiting

Rate limiting on a single server is relatively simple: you keep counters in memory and the problem is solved. Things get significantly more complicated when your application runs across multiple instances behind a load balancer. If each instance maintains its own counters, a client can effectively multiply its allowed rate by the number of instances.

The standard solution is to use a **centralized data store**, typically Redis, to maintain rate limit state. Redis is a natural fit because it is fast (sub-millisecond latency), supports atomic operations, and provides built-in key expiration via TTL.

A basic Redis-based token bucket might use a Lua script to atomically check and decrement the token count:

```lua
-- KEYS[1] = rate limit key (e.g., "rate:user:123")
-- ARGV[1] = max tokens
-- ARGV[2] = refill rate per second
-- ARGV[3] = current timestamp in milliseconds

local key = KEYS[1]
local max_tokens = tonumber(ARGV[1])
local refill_rate = tonumber(ARGV[2])
local now = tonumber(ARGV[3])

local bucket = redis.call('HMGET', key, 'tokens', 'last_refill')
local tokens = tonumber(bucket[1]) or max_tokens
local last_refill = tonumber(bucket[2]) or now

local elapsed = (now - last_refill) / 1000.0
local new_tokens = math.min(max_tokens, tokens + elapsed * refill_rate)

if new_tokens >= 1 then
    new_tokens = new_tokens - 1
    redis.call('HMSET', key, 'tokens', new_tokens, 'last_refill', now)
    redis.call('EXPIRE', key, math.ceil(max_tokens / refill_rate) + 1)
    return 1
else
    redis.call('HMSET', key, 'tokens', new_tokens, 'last_refill', now)
    redis.call('EXPIRE', key, math.ceil(max_tokens / refill_rate) + 1)
    return 0
end
```

Using a Lua script ensures that the read-check-update cycle is atomic within Redis, preventing race conditions between concurrent requests.

#### Challenges in Distributed Rate Limiting

Even with a centralized store, distributed rate limiting introduces several challenges:

* **Latency overhead**: Every rate limit check now involves a network round-trip to Redis. For latency-sensitive endpoints, this can be significant. Some systems mitigate this with local "pre-check" caches that allow obviously-within-limit requests to pass without hitting Redis, only consulting the central store when the local count approaches the threshold.

* **Race conditions**: Without atomic operations, two instances could simultaneously read the same counter value, both decide the request is allowed, and both decrement the counter, effectively allowing one extra request. Lua scripts in Redis (or Redis transactions with WATCH/MULTI) solve this, but you need to be deliberate about it.

* **Clock skew**: If your application instances have slightly different system clocks, timestamp-based algorithms (like sliding window log) can produce inconsistent results. Using the Redis server's time (`redis.call('TIME')`) instead of client-side timestamps avoids this problem.

* **Redis availability**: If Redis goes down, you need a fallback strategy. Common approaches include failing open (allowing all requests, sacrificing protection for availability), failing closed (rejecting all requests, sacrificing availability for protection), or falling back to local in-memory rate limiting with relaxed limits.

* **Multi-region deployments**: When your application spans multiple geographic regions, each with its own Redis instance, maintaining a globally consistent rate limit is difficult. Some systems accept per-region limits (effectively multiplying the global limit by the number of regions), while others use asynchronous replication with eventual consistency.


### Where to Implement Rate Limiting

Rate limiting can be applied at different layers of the system architecture, depending on the requirements:

* **API Gateway level**: Managed gateways like AWS API Gateway, Kong, or Azure API Management offer built-in rate limiting as a configurable feature. This is often the simplest approach for teams using cloud-native architectures, as it requires no custom code and can be applied uniformly across all endpoints.

* **Reverse proxy / middleware level**: Tools like Nginx, HAProxy, or Envoy can enforce rate limits before requests even reach the application. This offloads the work from the application servers and provides a centralized enforcement point.

* **Application level**: Rate limiting logic embedded directly in the application code gives the most flexibility -- for example, applying different limits per user tier or per endpoint. Libraries and frameworks often provide middleware for this purpose. A shared data store like Redis is commonly used to track request counts across multiple application instances.

In practice, many systems combine multiple levels. A global rate limit at the API gateway protects the infrastructure, while application-level limits enforce fine-grained, business-specific rules.

> **Related posts**: [Load Balancing](/posts/load-balancers/), [Caching](/posts/caching/), [Availability](/posts/availability/)
