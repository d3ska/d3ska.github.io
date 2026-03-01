---
title: "Request-Response vs Publish-Subscribe Architecture"
categories:
  - System Design
tags:
  - Request Response
  - Pub-Sub
  - Messaging
  - Software Architecture
  - Communication Patterns
---

### Request-Response Architecture

**Request-Response** is a foundational communication pattern primarily utilized in client-server architectures. In this model, a client sends a request to a server, which then processes the request and sends back an appropriate response. This interaction ensures a tight coupling between the client's request and the server's response, making the communication synchronous and bidirectional.

Key characteristics of the Request-Response model include:

1. **Direct Interaction:** The client actively waits for the server's response after sending a request. The server, upon receiving the request, is expected to provide a corresponding reply.
2. **Statelessness:** Typically found in HTTP-based systems, the server doesn't retain any memory of client interactions between individual requests, promoting scalability and simplicity. That said, request-response is not always stateless. Technologies like gRPC bidirectional streaming and WebSockets maintain persistent connections with state between calls. A gRPC stream, for example, keeps a long-lived HTTP/2 connection open where both sides can send messages at any time, which makes it closer to a session than a one-off request. The "stateless" label really applies to vanilla HTTP request-response, not the pattern as a whole.
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

The system managing these subscriptions and dispatching messages is commonly referred to as the "message broker". It ensures that messages published to a topic are received by all subscribers to that topic. Popular message broker technologies include Apache Kafka, RabbitMQ, AWS SNS/SQS, Google Cloud Pub/Sub, and Redis Pub/Sub. Each offers different trade-offs in terms of throughput, durability, ordering guarantees, and operational complexity.

1. **Dynamic Scalability:** New subscribers can be added dynamically without needing any changes in the publisher's configuration.
2. **Decoupled Architecture:** Publishers and subscribers are loosely coupled, meaning that they have minimal knowledge about each other, promoting flexibility in system evolution.
3. **Real-time Communication:** Ideal for scenarios that require real-time updates like stock market feeds or live sports scores.


### Pub/Sub Pros & Cons

**Pros:**

* Easily scales with multiple receivers or subscribers.
* Perfectly suited for microservices architecture.
* Loose coupling ([please refer to this post about coupling](/posts/what-is-coupling/))
* Operational even when some clients are offline.

**Cons:**

* Message delivery issues can lead to eventual consistency challenges.
* Introduces complexity, e.g., managing backpressure, brokers storing messages, re-reading of topics, etc.


### Kafka Producer and Consumer Example

To make the Pub/Sub pattern more concrete, here is what producing and consuming messages looks like with Apache Kafka's Java client. The producer publishes order events to a topic, and the consumer reads them.

#### Producer

```java
Properties props = new Properties();
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.ACKS_CONFIG, "all");

try (KafkaProducer<String, String> producer = new KafkaProducer<>(props)) {
    String orderId = "order-123";
    String orderJson = "{\"orderId\":\"order-123\",\"item\":\"laptop\",\"quantity\":1}";

    ProducerRecord<String, String> record =
        new ProducerRecord<>("orders", orderId, orderJson);

    producer.send(record, (metadata, exception) -> {
        if (exception != null) {
            log.error("Failed to send message", exception);
        } else {
            log.info("Sent to partition {} at offset {}",
                metadata.partition(), metadata.offset());
        }
    });
}
```

The key (`orderId`) determines which partition the message lands in. All messages with the same key go to the same partition, which is important for ordering guarantees (more on that below).

#### Consumer

```java
Properties props = new Properties();
props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
props.put(ConsumerConfig.GROUP_ID_CONFIG, "order-processing-group");
props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props)) {
    consumer.subscribe(List.of("orders"));

    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        for (ConsumerRecord<String, String> record : records) {
            log.info("Received order: key={}, value={}, partition={}, offset={}",
                record.key(), record.value(), record.partition(), record.offset());

            processOrder(record.value());
        }
    }
}
```

Notice the `GROUP_ID_CONFIG`. Consumers with the same group ID form a **consumer group**, and Kafka distributes the topic's partitions among them. If one consumer goes down, Kafka rebalances its partitions to the remaining members. This is how Pub/Sub systems achieve fault tolerance without the producer knowing anything about who is consuming.


### Message Ordering and Delivery Semantics

When working with Pub/Sub systems, two important considerations are message ordering and delivery guarantees.

#### Partitions and Ordering

**Message ordering** refers to whether messages are delivered to consumers in the same order they were published. To understand how ordering works in practice, you need to understand what a **partition** is.

A partition is a subdivided, ordered log within a topic. When a topic is created in Kafka (or a similar broker), it is split into one or more partitions. Each partition is an independent, append-only sequence of messages. Producers write to a specific partition based on the message key (or round-robin if no key is provided), and each partition is consumed by exactly one consumer within a consumer group.

This matters for ordering because Kafka guarantees message order only within a single partition, not across partitions. If messages A, B, and C all have the same key, they land in the same partition and are delivered in order. But if A and B have different keys and go to different partitions, there is no guarantee about their relative delivery order.

Maintaining strict global ordering across all partitions is expensive and often unnecessary. Most systems settle for per-partition or per-key ordering. For example, if you are processing orders for a specific customer, using the customer ID as the partition key ensures that all events for that customer arrive in order, which is usually all you need.

#### Delivery Semantics

**Delivery semantics** describe how many times a message may be delivered to a consumer:

* **At-most-once**: Messages may be lost but are never delivered more than once. This is the fastest option but risks data loss. It suits scenarios where occasional missed messages are acceptable, such as metrics collection.
* **At-least-once**: Messages are guaranteed to be delivered but may arrive more than once. The consumer must handle duplicates, typically through idempotent processing. This is the most common choice in practice.
* **Exactly-once**: Each message is delivered and processed exactly one time. This is the strongest guarantee but the hardest to achieve. Some systems like Kafka support exactly-once semantics through transactional producers and idempotent consumers, though it comes with a performance cost.

Understanding these trade-offs is essential when designing a Pub/Sub system, as the right choice depends on whether your application can tolerate duplicates, lost messages, or the performance overhead of stronger guarantees.


### Backpressure

**Backpressure** is a mechanism for consumers to signal (directly or indirectly) that they cannot keep up with the incoming message rate. Without it, a fast producer can overwhelm a slow consumer, leading to out-of-memory errors, message loss, or cascading failures.

In a request-response system, backpressure is built-in: if the server is slow, the client waits. In Pub/Sub, the producer and consumer are decoupled, so there is no natural feedback loop. You have to design for it explicitly.

There are several common strategies:

* **Bounded queues**: Configure the broker or consumer buffer with a maximum size. When the buffer is full, the producer blocks or receives an error. Kafka's `max.block.ms` setting controls how long a producer will wait when its buffer is full before throwing an exception. This is the simplest form of backpressure and prevents unbounded memory growth.

* **Consumer groups**: Scale the number of consumers horizontally. If a single consumer cannot keep up with the throughput, adding more consumers to the group distributes the partition workload. Kafka automatically rebalances partitions across group members. This doesn't reduce the message rate, but it increases the processing capacity to match it.

* **Rate limiting at the producer**: Throttle the producer's send rate to match the consumer's processing capacity. This can be done application-side (e.g., a token bucket controlling how many messages per second are sent) or through broker-level quotas. Kafka supports producer quotas that limit the bytes-per-second a client can publish.

* **Pausing and resuming consumption**: Kafka's consumer API supports `pause()` and `resume()` on specific partitions. When a consumer detects that its internal processing queue is getting too deep, it can pause consumption from certain partitions, process the backlog, and then resume. This gives fine-grained control without affecting other partitions.

Which strategy fits depends on your system's tolerance for latency versus throughput. In my experience, bounded queues combined with consumer group scaling covers the vast majority of cases. Only reach for producer-side rate limiting or partition-level pause/resume when you have very specific latency requirements.


### Summary

Request-Response and Publish-Subscribe architectures each have their strengths, and neither is universally superior. The choice depends on the specific context and use case. While Request-Response is excellent for direct and immediate interactions, Publish-Subscribe is tailored for scalable, real-time, and decoupled communications. Often, systems can benefit from employing both approaches in tandem, harnessing the strengths of each to address varied needs.

> **Related posts**: [Polling and Streaming](/posts/polling-and-streaming/), [What is Coupling](/posts/what-is-coupling/), [Microservices vs Monolith](/posts/microservices-vs-monolith/)

