---
title: "Polling And Streaming"
categories:
  - System Design
tags:
  - Polling And Streaming
---

In computing systems, the interaction between clients and servers is fundamental. This interaction often revolves around the client sending requests, and the server responding with the requested data. The frequency at which this data is updated can vary, and this largely depends on the system's requirements and purpose. Two fundamental concepts often used in this context are polling and streaming.

### Polling

![img]({{site.url}}/assets/blog_images/2023-05-11-polling-and-streaming/client-server-polling.png)

Polling is essentially the process of gathering data at regular intervals. For instance, a system may be set up to request temperature data every 'X' seconds. This method has its utility, but it is not without its limitations.

Consider an online chat platform such as Facebook or WhatsApp, where instant messaging is paramount. In such scenarios, receiving messages after a certain time interval is not ideal. You could theoretically decrease the polling interval to as low as 0.1 second to simulate real-time communication, but this approach leads to significant load on the servers.

For a single client, issuing 10 requests per second might seem manageable. However, when you scale this to thousands or potentially even millions of users, each issuing 10 requests per second, it results in a tremendous load on the server. That's precisely where the concept of streaming becomes relevant.



### Streaming

![img]({{site.url}}/assets/blog_images/2023-05-11-polling-and-streaming/client-server-streaming.png)

In the context of networking, streaming generally refers to continuously receiving a feed of information from a server by maintaining an open connection between the client and the server.

Streaming is achieved via sockets, a fascinating subject worth learning more about. In simple terms, sockets are akin to files that your computer can read from and write to over an ongoing connection with another machine. This connection remains open until one of the machines terminates it.

In essence, polling and streaming are two different approaches to client-server communication. While polling is about periodic data requests, streaming involves maintaining an open connection for continuous data transfer. The choice between the two depends on the specific requirements and constraints of your system.

