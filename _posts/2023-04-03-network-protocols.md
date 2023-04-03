---
title: "Network Protocols"
categories:
  - Blog
tags:
  - Network
  - Protocols
---

### What a protocol is?

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
Certain ports have pre-defined uses, and altough you usually won't be required to have them memorized, they can sometimes come in handy. 
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
Other protocols like **TCP**, **UDP** and **HTTP** are build on top of IP. The modern internet effectively operates following Internet Protocol, it means that when the machine or client for instance, try to interact with another machine or server, and it sends data to the another machine or a server and it sends data to the other machine, that data is going to be sent in the form of whats known as a IP packet. 

You can think of an **IP Packet** as the fundamental unit of data that is sent from one machine to another. IP packets are the building blocks of communication between machines over the internet, and are made up of bytes. 
They really have two main sections are known as the **IP header and the data**. <br>

The header of an IP packet typically contains important information, such as the IP addresses of the source and destination, along with other fields that help route the packet. Given its importance, it is designed to be compact, ranging from 20 to 60 bytes in size.

In contrast, the data portion of an IP packet contains the actual information being transmitted between machines. However, IP packets have a maximum size limit of only 65,536 bytes, which may not be sufficient for transmitting large files or data. When sending data that exceeds this limit, the information is split into multiple IP packets.

However, if the only protocol used is the Internet Protocol, there is no guarantee that all the packets will be received or that they will be received in the correct order. Some packets may get lost, leading to incomplete data transmission. In addition, the order in which packets are received and interpreted may not be as intended.

<br>

### TCP 
The Transmission Control Protocol (TCP) is designed to address the issues mentioned earlier and is built on top of the Internet Protocol (IP). Its purpose is to send IP packets in an ordered, reliable, and error-free manner. This means that the order in which packets are read by the destination machine is guaranteed, and if any packets fail to reach their destination or get corrupted during transmission, the sender is informed so that the packets can be resent without corruption.

TCP is used in almost all web applications, allowing for the transmission of arbitrarily long pieces of data between machines. It achieves this by creating a connection [handshake](https://developer.mozilla.org/en-US/docs/Glossary/TCP_handshake) between the sender and receiver, ensuring the reliability of data transmission.

It is worth noting that TCP, being built on top of the IP protocol, includes a TCP header in addition to the IP header and data part. This TCP header contains essential information, such as source and destination port numbers, sequence and acknowledgement numbers, and other fields that enable TCP to perform its reliable data transmission functions.

TCP is typically implemented in the kernel and exposes sockets to applications. Applications can use these sockets to stream data through an open connection, ensuring the data is transmitted reliably and accurately.

So basically it's a more powerful and more functional wrapper around IP, around the Internet Protocol. But still what it lacks is a really robust framework
that developers, software engineers can use to really define meaningful and easy to use communication channels for clients and servers in the system. 


<br>

### HTTP
The **H**yper**T**ext **T**ransfer **P**rotocol is very common network protocol implemented on top of TCP, it provides a higher level of abstraction above TCP and IP through the request and response paradigm. This paradigm enables machines to communicate with each other by following a set of rules that govern the exchange of requests and responses.

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


