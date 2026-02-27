---
title: "Network Protocols"
categories:
  - System Design
tags:
  - Networking
  - Protocols
  - TCP
  - HTTP
  - IP
---

### What is a protocol?

Protocol can be defined as an agreed-upon set of rules that govern an interaction between two or more parties. In our daily lives, we use protocols when interacting with people we meet, such as greeting them with a friendly "Hello, how are you doing?" and shaking hands. Similarly, when machines interact with each other, they also follow specific network protocols.
These protocols ensure that communication between machines is standardized and efficient, just like how human protocols facilitate smooth interactions between individuals. For example, the Transmission Control Protocol (TCP) is a network protocol used for communication between machines on the internet. It provides a reliable, ordered, and error-checked delivery of data between applications running on different hosts.


### Prerequisites 
#### IP Address

An address given to each machine connected to the public internet. IPv4 addresses consist of four numbers separated by dots:
**a.b.c.d** where all four numbers are between 0 and 255. Special values include:
* **127.0.0.1**: Your own local machine. Also referred to as **localhost**.
* **192.168.x.y**: Your private network. For instance, your machine and all machines on your private wifi network will usually have the **192.168** prefix

#### Port
In order for multiple programs to listen for new network connections to the same machine without colliding, they pick a **port** to listen on. 
A port is an integer between 0 and 65,535 (2<sup>16</sup> ports total).

Typically, ports 0-1023 are reserved for system ports (also called well-known ports) and shouldn't be used by user-level processes. 
Certain ports have pre-defined uses, and although you usually won't be required to have them memorized, they can sometimes come in handy. 
Below are some examples:
* 22: Secure Shell
* 53: DNS lookup
* 80: HTTP
* 443: HTTPS

#### IP Packet
Sometimes more broadly referred to as just a (network) **packet**, an IP packet is effectively the smallest unit used to describe data
being sent over **IP**, aside from bytes. An IP packet consists of:
* an **IP header**, which contains the source and destination **IP addresses** as well as other information related to the network
* a **payload**, which is just the data being sent over the network


<br>

### IP

Stands for **Internet Protocol**. This network protocol outlines how almost all machine-to-machine communications should happen in the world. 
Other protocols like **TCP**, **UDP** and **HTTP** are built on top of IP. The modern internet effectively operates following the Internet Protocol. When a machine or client tries to interact with another machine or server, the data is sent in the form of what is known as an IP packet.


They have a maximum size limit of only 65,535 bytes, which may not be sufficient for transmitting large files or data. When sending data that exceeds this limit, the information is split into multiple IP packets.

However, if the only protocol used is the Internet Protocol, there is no guarantee that all the packets will be received or that they will be received in the correct order. Some packets may get lost, leading to incomplete data transmission. In addition, the order in which packets are received and interpreted may not be as intended.

<br>

### TCP 
The Transmission Control Protocol (TCP) is designed to address the issues mentioned earlier and is built on top of the Internet Protocol (IP). Its purpose is to send IP packets in an ordered, reliable, and error-checked manner. Note that "error-checked" means TCP detects corrupted or missing data and requests retransmission; it does not prevent errors from occurring during transmission. This means that the order in which packets are read by the destination machine is guaranteed, and if any packets fail to reach their destination or get corrupted during transmission, the sender is informed so that the packets can be resent without corruption.

TCP is used in almost all web applications, allowing for the transmission of arbitrarily long pieces of data between machines. It achieves this by creating a connection [handshake](https://developer.mozilla.org/en-US/docs/Glossary/TCP_handshake) between the sender and receiver, ensuring the reliability of data transmission.

TCP is typically implemented in the kernel and exposes sockets to applications. Applications can use these sockets to stream data through an open connection, ensuring the data is transmitted reliably and accurately.

So basically it's a more powerful and more functional wrapper around IP, around the Internet Protocol. But still what it lacks is a really robust framework
that developers, software engineers can use to really define meaningful and easy-to-use communication channels for clients and servers in the system.


<br>

### UDP

The User Datagram Protocol (UDP) is another protocol built on top of IP. Unlike TCP, UDP does not establish a connection before sending data and provides no guarantees regarding ordering, reliability, or delivery. There is no handshake, no acknowledgment of received packets, and no automatic retransmission of lost data.

This lack of overhead makes UDP significantly faster and more lightweight than TCP. Because it skips the connection setup and does not wait for acknowledgments, UDP achieves lower latency per packet.

UDP is well suited for use cases where speed matters more than perfect reliability:
* **Video streaming and video calls**: A few dropped frames are acceptable; waiting for retransmissions would cause noticeable lag.
* **Online gaming**: Players need real-time updates; outdated retransmitted packets are less useful than fresh data.
* **DNS lookups**: A single request-response exchange benefits from minimal overhead. If the response is lost, the client simply retries.
* **Voice over IP (VoIP)**: Similar to video, slight data loss is preferable to the delays introduced by retransmission.

UDP uses the same addressing model as TCP (IP address plus port number), and each UDP message is called a **datagram**. Since there is no connection state to maintain, a single UDP socket can send datagrams to many different destinations and receive datagrams from many different sources.

In short, TCP and UDP represent two fundamentally different trade-offs: TCP prioritizes reliability and correctness at the cost of additional latency and overhead, while UDP prioritizes speed and simplicity at the cost of guaranteed delivery.


<br>

### HTTP
The **H**yper**T**ext **T**ransfer **P**rotocol is very common network protocol implemented on top of TCP, it provides a higher level of abstraction above TCP and IP through the request and response paradigm. This enables machines to communicate with each other by following a set of rules that govern the exchange of requests and responses.

HTTP's request and response paradigm, along with its accompanying rules, makes it easy for developers to create robust and maintainable systems. This is why most of the modern-day systems rely on the HTTP protocol for communication.

With HTTP, developers can focus solely on HTTP requests and responses, without worrying about the underlying details of IP packets and TCP.

This abstraction makes development faster, more efficient, and less error-prone, as developers can focus solely on building the functionality that their application requires.

Requests typically have the following schema:

``` 
host: string (example: matthewonsoftware.com)
port: integer (example: 80 or 443)
method: string (example: GET, PUT, POST, DELETE, OPTIONS or PATCH)
headers: pair list (example: "Content-Type": "application/json")
body: opaque sequence of bytes
```

Responses typically have the following schema:

```
status code: integer (example: 200, 401)
headers: pair list (example: "Content-Length": 1238)
body: opaque sequence of bytes
```

#### HTTP Versions

HTTP has evolved significantly over the years, and understanding the key versions helps when reasoning about system performance:

* **HTTP/1.1** introduced persistent connections (keep-alive) so that multiple requests could share a single TCP connection. However, it still suffers from head-of-line blocking: each request on a connection must wait for the previous response before being sent.

* **HTTP/2** solved the head-of-line blocking problem at the application layer by introducing **multiplexing**, which allows multiple requests and responses to be interleaved over a single TCP connection simultaneously. It also added header compression (HPACK) and server push capabilities.

* **HTTP/3** takes this a step further by replacing TCP entirely with **QUIC**, a protocol built on top of UDP. QUIC provides its own reliability, ordering, and congestion control mechanisms while eliminating TCP-level head-of-line blocking. Because QUIC handles each stream independently, a lost packet in one stream does not stall other streams. HTTP/3 also offers faster connection establishment, often completing the handshake in a single round trip.


