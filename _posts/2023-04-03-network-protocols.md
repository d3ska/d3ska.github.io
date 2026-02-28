---
title: "Network Protocols"
categories:
  - System Design
tags:
  - Networking
  - Protocols
  - TCP
  - UDP
  - HTTP
  - IP
---

When two machines communicate over a network, they need to agree on the rules: how to find each other, how to structure the data, and what to do when something goes wrong. These agreed-upon rules are **network protocols**. Every request your browser makes, every API call your backend sends, and every message a chat app delivers depends on a stack of protocols working together.

The most important ones to understand for system design are IP, TCP, UDP, and HTTP. Each builds on the layer below it and solves a different set of problems.

### IP: Addressing and Routing

The **Internet Protocol (IP)** is the foundation of the stack. It handles two things: addressing (every machine gets an IP address) and routing (getting data from source to destination across intermediate networks).

An **IP address** identifies a machine on the network. IPv4 addresses look like `192.168.1.42`, where each of the four numbers ranges from 0 to 255. Two special ranges are worth knowing:

- `127.0.0.1` (also called **localhost**) always refers to the machine you are on.
- `192.168.x.y` addresses are reserved for private networks, such as a home Wi-Fi or office LAN.

A **port** is a number between 0 and 65,535 that identifies a specific application on a machine. Without ports, only one program per machine could listen for network traffic. Some ports are conventionally assigned: 22 for SSH, 53 for DNS, 80 for HTTP, 443 for HTTPS.

Data travels as **IP packets**, each consisting of a header (source address, destination address, metadata) and a payload (the actual data). A single packet can carry at most 65,535 bytes. Anything larger must be split across multiple packets.

IP alone makes no promises. Packets may arrive out of order, arrive duplicated, or not arrive at all. It is a best-effort delivery system. The protocols built on top of IP exist precisely to address these limitations.

### TCP: Reliable, Ordered Delivery

The **Transmission Control Protocol (TCP)** adds reliability on top of IP. Before any data is exchanged, the client and server perform a [three-way handshake](https://developer.mozilla.org/en-US/docs/Glossary/TCP_handshake) to establish a connection. Once the connection is open, TCP guarantees that:

- **All data arrives** at the destination. Lost packets are detected and retransmitted.
- **Data arrives in order.** Each segment carries a sequence number, and the receiver reassembles them correctly.
- **Corrupted data is detected.** Each segment includes a checksum. If verification fails, the segment is discarded and retransmitted.

This makes TCP suitable for any communication where correctness matters: web pages, API calls, file transfers, database queries, and email.

The trade-off is overhead. The handshake adds latency to connection setup. Acknowledgments and retransmissions add latency during transfer. And because TCP enforces in-order delivery, a single lost packet stalls all packets behind it, a problem known as **head-of-line blocking**.

TCP is implemented in the operating system kernel and exposed to applications through **sockets**. Here is a minimal TCP echo server and client in Java:

```java
// Server: listens on port 8080 and echoes back whatever it receives
try (ServerSocket server = new ServerSocket(8080)) {
    try (Socket client = server.accept()) {
        BufferedReader in = new BufferedReader(
                new InputStreamReader(client.getInputStream()));
        PrintWriter out = new PrintWriter(client.getOutputStream(), true);

        String message = in.readLine();
        out.println("Echo: " + message);
    }
}
```

```java
// Client: connects, sends a message, reads the response
try (Socket socket = new Socket("localhost", 8080)) {
    PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
    BufferedReader in = new BufferedReader(
            new InputStreamReader(socket.getInputStream()));

    out.println("Hello, server");
    String response = in.readLine(); // "Echo: Hello, server"
}
```

The developer works with streams (readers and writers), not raw packets. TCP handles segmentation, ordering, retransmission, and reassembly behind the scenes.

### UDP: Fast, Unordered Delivery

The **User Datagram Protocol (UDP)** also sits on top of IP but takes the opposite approach from TCP. There is no connection, no handshake, no acknowledgment, and no guaranteed ordering. You send a **datagram** (a self-contained message) and hope it arrives.

This sounds reckless, but the lack of overhead makes UDP significantly faster than TCP. It is the right choice when speed matters more than perfect delivery:

- **Video streaming and calls.** A few dropped frames are imperceptible. Waiting for retransmissions would cause visible lag.
- **Online gaming.** Players need real-time position updates. A retransmitted packet from 200ms ago is useless because the player has already moved.
- **DNS lookups.** A single request-response exchange. If the response is lost, the client simply asks again.
- **Voice over IP.** Slight audio loss is preferable to the pauses that retransmission would introduce.

UDP uses the same addressing model as TCP (IP address plus port). Since there is no connection state to maintain, a single UDP socket can send datagrams to many different destinations and receive from many different sources.

### TCP vs UDP at a Glance

| | TCP | UDP |
|---|---|---|
| Connection | Three-way handshake | None |
| Delivery guarantee | Yes (retransmission) | No |
| Ordering guarantee | Yes (sequence numbers) | No |
| Error detection | Checksum + retransmit | Checksum only |
| Overhead | Higher | Lower |
| Latency | Higher | Lower |
| Typical use | Web, APIs, file transfer, email | Streaming, gaming, DNS, VoIP |

In short, TCP and UDP represent two fundamentally different trade-offs. TCP prioritizes correctness at the cost of overhead. UDP prioritizes speed at the cost of guaranteed delivery. Choosing between them depends entirely on whether your application can tolerate lost data.

### HTTP: The Application-Layer Protocol

TCP solves the transport problem, but it says nothing about the meaning of the data being sent. The **HyperText Transfer Protocol (HTTP)** adds structure on top of TCP through a request-response model. A client sends a request with a method, path, headers, and optional body. The server responds with a status code, headers, and optional body.

A request looks like this:

```
GET /api/orders/42 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGci...
```

A response looks like this:

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 127

{"id": 42, "status": "PLACED", "total": "59.98"}
```

This simple structure is the foundation of virtually every web application and REST API. Developers work with HTTP methods (`GET`, `POST`, `PUT`, `DELETE`), status codes (`200 OK`, `404 Not Found`, `500 Internal Server Error`), and headers, without thinking about TCP segments or IP packets underneath.

#### HTTP Versions

HTTP has evolved significantly, and the differences matter for system performance:

**HTTP/1.1** introduced persistent connections (keep-alive), allowing multiple requests to reuse a single TCP connection. However, requests on a connection are processed sequentially: request 1 must complete before request 2 can begin. This is **head-of-line blocking** at the application layer. Browsers work around it by opening multiple TCP connections per domain (typically six), but this wastes resources.

**HTTP/2** solved application-layer head-of-line blocking with **multiplexing**: multiple requests and responses are interleaved as binary frames over a single TCP connection. It also introduced **header compression** (HPACK), which significantly reduces overhead when the same headers are sent repeatedly, and **server push**, which allows the server to send resources proactively before the client requests them. In practice, HTTP/2 reduces page load times noticeably for sites that serve many small resources (scripts, stylesheets, images).

**HTTP/3** replaces TCP entirely with **QUIC**, a transport protocol built on UDP. QUIC provides its own reliability and congestion control while handling each stream independently. If a packet belonging to one stream is lost, other streams continue unaffected, which eliminates TCP-level head-of-line blocking. QUIC also establishes connections faster (often in a single round trip, compared to TCP's three-way handshake plus the TLS handshake), which makes a noticeable difference on mobile networks with high latency. Major platforms including Google, Cloudflare, and Meta have adopted HTTP/3 at scale, and it is the default for most modern browsers.

> **Related posts**: [Latency and Throughput](/posts/latency-and-throughput/), [Caching](/posts/caching/), [Load Balancers](/posts/load-balancers/)
