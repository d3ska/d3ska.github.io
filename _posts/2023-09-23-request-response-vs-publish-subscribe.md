---
title: "Request-Response vs Publish-Subscribe Architecture"
categories:
  - Blog
tags:
  - Request Response
  - Pub-Sub
  - Communication styles
---

### Request-Response Architecture
Request-Response is a foundational communication pattern primarily utilized in client-server architectures. In this model, a client sends a request to a server, which then processes the request and sends back an appropriate response. This interaction ensures a tight coupling between the client's request and the server's response, making the communication synchronous and bidirectional.

Key characteristics of the Request-Response model include:

1. **Direct Interaction:** The client actively waits for the server's response after sending a request. The server, upon receiving the request, is expected to provide a corresponding reply.
2. **Statelessness:** Typically found in HTTP-based systems, the server doesn't retain any memory of client interactions between individual requests, promoting scalability and simplicity.
3. **Point-to-point Communication:** The model revolves around direct communication between two entities, namely the client and the server, making it simple and straightforward.

### Request Response Pros & Cons

**Pros:**

* Elegant and Simple, well known approach.
* Stateless (HTTP).
* Scalable, especially horizontally (though 'scalability' is often an overloaded term).

**Cons:**

* Not suitable for multiple receivers.
* High Coupling.
* Both the client and the server have to be active and running simultaneously.
* Requires mechanisms like chaining, circuit breakers, and retries to ensure interconnected, highly coupled systems can communicate effectively.


### Publish Subscribe Architecture

Publish-Subscribe, or Pub/Sub, is an alternative message communication pattern where publishers send messages without specifying receivers. Instead, messages are categorized into topics. Subscribers, on the other hand, express interest in one or more topics and only receive messages that are of interest to them. Think of it like a magazine subscription model; publishers release editions without knowing who'll read them, and subscribers receive only editions of magazines they've subscribed to.

The system managing these subscriptions and dispatching messages is commonly referred to as the "message broker". It ensures that messages published to a topic are received by all subscribers to that topic.

1. **Dynamic Scalability:** New subscribers can be added dynamically without needing any changes in the publisher's configuration.
2. **Decoupled Architecture:** Publishers and subscribers are loosely coupled, meaning that they have minimal knowledge about each other, promoting flexibility in system evolution.
3. **Real-time Communication:** Ideal for scenarios that require real-time updates like stock market feeds or live sports scores.


### Pub/Sub Pros & Cons

**Pros:**

* Easily scales with multiple receivers or subscribers.
* Perfectly suited for microservices architecture.
* Loose coupling ([please refer to this post about coupling](https://matthewonsoftware.com/blog/what-is-coupling/))
* Operational even when some clients are offline.

**Cons:**

* Message delivery issues can lead to eventual consistency challenges.
* Introduces complexity, e.g., managing back pressure, brokers storing messages, re-reading of topics, etc.


### Summary

Request-Response and Publish-Subscribe architectures each have their strengths, and neither is universally superior. The choice depends on the specific context and use case. While Request-Response is excellent for direct and immediate interactions, Publish-Subscribe is tailored for scalable, real-time, and decoupled communications. Often, systems can benefit from employing both approaches in tandem, harnessing the strengths of each to address varied needs. 

