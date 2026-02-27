---
title: "Polling And Streaming"
categories:
  - System Design
tags:
  - Polling And Streaming
---

In computing systems, the interaction between clients and servers is fundamental. This interaction often revolves around the client sending requests, and the server responding with the requested data. The frequency at which this data is updated can vary, and this largely depends on the system's requirements and purpose. Two fundamental concepts often used in this context are polling and streaming.

### Polling

![img]({{site.url}}/assets/blog_images/2023-05-11-polling-and-streaming/polling-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-05-11-polling-and-streaming/polling-dark.png){: .dark }

Polling is essentially the process of gathering data at regular intervals. For instance, a system may be set up to request temperature data every 'X' seconds. This method has its utility, but it is not without its limitations.

Consider an online chat platform such as Facebook or WhatsApp, where instant messaging is paramount. In such scenarios, receiving messages after a certain time interval is not ideal. You could theoretically decrease the polling interval to as low as 0.1 second to simulate real-time communication, but this approach leads to significant load on the servers.

For a single client, issuing 10 requests per second might seem manageable. However, when you scale this to thousands or potentially even millions of users, each issuing 10 requests per second, it results in a tremendous load on the server. That's precisely where the concept of streaming becomes relevant.


### Long Polling

Long polling is an improvement over standard polling that significantly reduces unnecessary network traffic. In regular polling, the client sends a request and the server responds immediately, even if there is no new data -- resulting in many empty responses. With long polling, the client sends a request and the server holds the connection open until new data becomes available or a predefined timeout is reached. Once the server responds (either with new data or a timeout), the client immediately issues a new request, and the cycle repeats.

This approach drastically reduces the number of round trips compared to frequent short polling while still providing near-real-time updates. Long polling was widely used before WebSockets became mainstream, and it remains a solid choice for systems where the infrastructure does not support persistent connections. However, long polling still has drawbacks: each held connection consumes server resources, and under high concurrency the number of open connections can become a bottleneck.


### Streaming

![img]({{site.url}}/assets/blog_images/2023-05-11-polling-and-streaming/streaming-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-05-11-polling-and-streaming/streaming-dark.png){: .dark }

In the context of networking, streaming generally refers to continuously receiving a feed of information from a server by maintaining an open connection between the client and the server.

Streaming is achieved via sockets, a fascinating subject worth learning more about. In simple terms, sockets are akin to files that your computer can read from and write to over an ongoing connection with another machine. This connection remains open until one of the machines terminates it.

#### WebSockets

WebSockets are the primary technology for full-duplex, bidirectional communication between a client and a server over a single, long-lived TCP connection. The connection starts as a standard HTTP request and is then upgraded to a WebSocket connection via a protocol handshake. Once established, both the client and the server can send messages to each other at any time without the overhead of HTTP headers on every message. This makes WebSockets ideal for use cases like chat applications, live dashboards, multiplayer games, and collaborative editing tools.

#### Server-Sent Events (SSE)

Server-Sent Events (SSE) provide a simpler alternative to WebSockets when communication only needs to flow in one direction -- from server to client. With SSE, the client opens a standard HTTP connection, and the server pushes updates through it as they become available. The browser natively supports SSE through the `EventSource` API, which also handles automatic reconnection if the connection drops. SSE works well for scenarios like live news feeds, stock tickers, and notification streams where the client does not need to send data back to the server over the same channel.


### When to Use What

Choosing the right approach depends on the nature of the communication your system requires:

* **Polling** works best for simple use cases with low-frequency updates where near-real-time delivery is not critical. For example, checking for new email every 30 seconds or refreshing a dashboard periodically. It is the easiest to implement and works with any HTTP infrastructure.

* **Long polling** suits scenarios with moderate real-time requirements where WebSocket support may not be available. It reduces wasted requests compared to regular polling and provides a reasonable approximation of push-based communication. It was the go-to technique for real-time web apps before WebSockets gained broad adoption.

* **Server-Sent Events (SSE)** are a great fit when you only need the server to push updates to the client. They are simpler to set up than WebSockets, work over standard HTTP, and benefit from built-in browser reconnection logic. A live score feed or a deployment log stream are good examples.

* **WebSockets** are the right choice when you need full-duplex, low-latency, bidirectional communication. Chat applications, real-time collaborative editors, and multiplayer games all benefit from the persistent, two-way channel that WebSockets provide. The trade-off is increased complexity in connection management, scaling (since each client holds an open connection), and handling reconnections gracefully.

