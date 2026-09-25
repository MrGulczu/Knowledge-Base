# TCP and UDP

## Overview

TCP and UDP are two of the most important transport-layer protocols used in IP networks.

They both allow applications to exchange data between systems, but they use very different approaches.

The main difference is:

```
TCP
→ connection-oriented
→ reliable
→ ordered delivery
→ acknowledgements
→ retransmission
```

while:

```
UDP
→ connectionless
→ lightweight
→ no built-in delivery confirmation
→ no built-in retransmission
→ no built-in ordering
```

TCP is normally used when reliable and ordered delivery is important.

UDP is commonly used when low delay and low protocol overhead are more important than guaranteed delivery.

---

## TCP

TCP stands for:

```
Transmission Control Protocol
```

TCP is a connection-oriented transport protocol.

Before normal application data is exchanged, TCP establishes a connection between the two endpoints.

TCP provides several important functions:

- connection establishment,
- sequence numbering,
- acknowledgements,
- retransmission of lost data,
- ordered delivery,
- flow control,
- congestion control,
- checksum-based error detection,
- graceful connection termination.

These mechanisms make TCP suitable for applications that require reliable transport.

---

## TCP Three-Way Handshake

TCP normally establishes a connection using a three-way handshake.

The three messages are:

```
SYN
SYN-ACK
ACK
```

Conceptually:

```
Client                         Server
  |                               |
  | ---------- SYN ------------>  |
  |                               |
  | <------- SYN-ACK -----------  |
  |                               |
  | ---------- ACK ------------>  |
  |                               |
  |       Connection ready        |
```

### SYN

The client sends:

```
SYN
```

to request a TCP connection.

### SYN-ACK

The server responds with:

```
SYN-ACK
```

This acknowledges the client's request and sends the server's own synchronization information.

### ACK

The client responds with:

```
ACK
```

After this exchange, the TCP connection is established and normal data transfer can begin.

The handshake confirms that both sides can communicate and synchronizes the sequence-number spaces used by TCP.

---

## TCP Sequence Numbers

TCP uses sequence numbers to track the position of data inside the TCP byte stream.

TCP does not treat application data as individual messages.

Instead, it provides an ordered stream of bytes.

For example:

```
Segment 1
Sequence Number = 1000

Segment 2
Sequence Number = 1500

Segment 3
Sequence Number = 2000
```

The receiver uses sequence numbers to determine where received data belongs.

If segments arrive in the wrong order, TCP can reconstruct the correct byte stream.

For example:

```
Received:

Segment 1
Segment 3
Segment 2
```

TCP can reorder the data as:

```
Segment 1
Segment 2
Segment 3
```

Out-of-order data may be buffered temporarily while the receiver waits for missing data.

---

## TCP Acknowledgements

TCP uses acknowledgements to confirm received data.

An acknowledgement normally indicates the next byte the receiver expects.

For example:

```
Sender
   |
   | Seq = 1000
   v
Receiver
   |
   | ACK = 1500
   v
Sender
```

This tells the sender that the preceding data has been received and the receiver now expects data beginning at byte 1500.

Acknowledgements are an important part of TCP reliability.

---

## TCP Retransmission

If TCP determines that expected data was not successfully delivered, it can retransmit that data.

Retransmission can occur after:

- a retransmission timeout,
- repeated acknowledgements indicating missing data,
- other loss-detection mechanisms.

Conceptually:

```
Sender
   |
   | Segment 1
   | Segment 2
   | Segment 3
   v
Receiver
```

If:

```
Segment 2
```

is lost, the sender can retransmit the missing data.

This is very different from UDP, where the transport protocol itself does not retransmit lost datagrams.

---

## TCP Error Detection

TCP includes a checksum used to detect corruption.

If a segment is received with invalid data according to the checksum, the segment is discarded.

Because the expected data is then missing, the sender can later retransmit it.

Conceptually:

```
Checksum valid
→ accept segment

Checksum invalid
→ discard segment
→ expected acknowledgement does not occur
→ retransmission may follow
```

The checksum detects transmission errors, but it is not a security or cryptographic integrity mechanism.

---

## TCP Ordered Delivery

TCP guarantees ordered delivery of the byte stream to the application.

If data arrives out of order, the receiving TCP stack can buffer the later data until the missing portion arrives.

This means the application receives data in the correct sequence even if packets did not travel through the network in exactly that order.

---

## TCP Flow Control

TCP flow control prevents the sender from overwhelming the receiving host.

The receiver has a limited amount of memory available for buffering incoming data.

If the sender transmits too much data before the receiving application can process it, the receive buffer can become full.

TCP uses the:

```
Receive Window
```

often written as:

```
rwnd
```

to communicate how much additional data the receiver is prepared to accept.

Conceptually:

```
Large receive window
→ receiver has available buffer space
→ sender can have more data in flight

Small receive window
→ receiver has less available buffer space
→ sender must slow down
```

If the receive buffer becomes completely full, the receiver can advertise:

```
Window = 0
```

which tells the sender to stop sending normal application data until more buffer space becomes available.

---

## Flow Control vs Congestion Control

Flow control and congestion control solve different problems.

```
Flow Control
→ protects the receiver
```

```
Congestion Control
→ protects the network path
```

The receive window tells the sender how much data the destination can currently accept.

The congestion-control mechanisms estimate how much data the network path can handle.

---

## TCP Congestion Control

TCP adjusts its sending behavior based on observed network conditions.

Possible signs of congestion include:

- packet loss,
- retransmission timeouts,
- duplicate acknowledgements,
- Explicit Congestion Notification where supported.

TCP commonly uses a:

```
Congestion Window
```

written as:

```
cwnd
```

Conceptually:

```
Network operating normally
→ sending rate can increase

Congestion detected
→ congestion window reduced
→ sender slows down
```

Common congestion-control concepts include:

```
Slow Start
Congestion Avoidance
Fast Retransmit
Fast Recovery
```

The exact algorithm depends on the operating system and TCP implementation.

The important principle is that TCP adapts its transmission rate to network conditions.

---

## TCP Connection Termination

TCP connections are normally closed gracefully using:

```
FIN
ACK
```

Because TCP is full-duplex, each direction of communication can be closed independently.

A typical graceful shutdown is:

```
Host A → Host B
FIN

Host B → Host A
ACK

Host B → Host A
FIN

Host A → Host B
ACK
```

When one side sends:

```
FIN
```

it means that side has no more data to send.

The other side may still continue sending until it also closes its direction.

---

## TCP Reset

TCP also provides:

```
RST
```

or:

```
Reset
```

RST is used to terminate or reject a TCP connection immediately.

It may appear when:

- a connection is invalid,
- a connection is unexpectedly terminated,
- a host receives traffic for a closed TCP port,
- an application aborts a connection.

Unlike FIN, RST does not represent a graceful shutdown.

---

## UDP

### Overview

UDP stands for:

```
User Datagram Protocol
```

UDP is a connectionless transport protocol.

It does not establish a session using a three-way handshake before sending data.

A sender can simply transmit a UDP datagram to a destination.

Conceptually:

```
Sender
   |
   | UDP Datagram
   v
Receiver
```

There is no built-in requirement for the receiver to acknowledge it.

---

## UDP Characteristics

UDP does not provide built-in:

- connection establishment,
- acknowledgements,
- retransmission,
- sequence numbering,
- ordered delivery,
- flow control,
- congestion control.

This makes UDP simpler and introduces less transport-protocol overhead than TCP.

However, this also means that UDP itself does not guarantee that data will:

- arrive,
- arrive only once,
- arrive in order.

---

## UDP Packet Loss

If a UDP datagram is lost, UDP itself does not recover it.

For example:

```
Sender
   |
   | Datagram 1
   | Datagram 2
   | Datagram 3
   v
Receiver
```

If:

```
Datagram 2
```

is lost, the receiver may simply receive:

```
Datagram 1
Datagram 3
```

UDP continues without retransmitting Datagram 2.

If the application requires reliability, the application protocol must implement its own recovery mechanism.

---

## UDP and Application-Level Reliability

UDP itself is unreliable in the transport-protocol sense, but applications can build reliability above UDP.

An application protocol can add features such as:

- acknowledgements,
- sequence numbers,
- retransmission,
- congestion control.

This allows protocols to use UDP while still implementing more advanced delivery behavior.

A modern example is QUIC, which runs over UDP but implements its own reliable transport features.

HTTP/3 uses QUIC instead of TCP.

---

## Why UDP Is Used

UDP is useful when low delay is more important than retransmitting old data.

For example, during a live voice call, a packet containing a small portion of audio may be lost.

Retransmitting that packet later may not be useful because the conversation has already moved forward.

It can be better to accept a small gap and continue with the newest audio.

Common UDP use cases include:

```
Voice calls
Video calls
Live streaming
Online gaming
DNS
DHCP
```

Not every application in these categories always uses UDP, but UDP is commonly associated with workloads where low latency is important.

---

## TCP vs UDP and Performance

UDP is often described as "faster" than TCP.

A more precise description is:

> UDP has less built-in transport overhead and does not require connection establishment or acknowledgement-based reliability.

This can reduce latency.

However, UDP is not automatically faster for every application.

The actual performance depends on:

- application design,
- network quality,
- packet loss,
- congestion,
- implementation.

TCP may perform better for workloads where reliable bulk transfer is required.

---

## Ports

Both TCP and UDP use port numbers.

IP addresses identify hosts or interfaces, while ports identify application endpoints on those hosts.

For example:

```
Client:
192.168.1.10

Server:
203.0.113.20
```

The client may use an ephemeral source port:

```
52341
```

and connect to:

```
443
```

on the server.

The connection can be represented as:

```
192.168.1.10:52341
        →
203.0.113.20:443
```

Some common port numbers include:

```
22
→ SSH

53
→ DNS

80
→ HTTP

443
→ HTTPS
```

A service can technically be configured on another port, so a port number does not guarantee which application protocol is actually running.

---

## Source and Destination Ports

The destination port normally identifies the service the client wants to reach.

The source port is often selected temporarily by the client operating system.

For example:

```
Client Source:
192.168.1.10:52341

Server Destination:
203.0.113.20:443
```

Different source ports allow the same client to maintain multiple simultaneous connections.

For example:

```
192.168.1.10:52341 → 203.0.113.20:443
192.168.1.10:52342 → 203.0.113.20:443
192.168.1.10:52343 → 198.51.100.50:443
```

The next topic, Ports and Sockets, covers this in more detail.

---

## Sockets

A socket is a software endpoint used for network communication.

A socket is associated with information such as:

```
IP Address
Port Number
Transport Protocol
```

For example:

```
192.168.1.10:52341 / TCP
```

An established connection can be distinguished using information from both endpoints.

Conceptually:

```
Source IP
Source Port
Destination IP
Destination Port
Protocol
```

Sockets, listening ports, ephemeral ports, and connection tuples are covered in more detail in the dedicated Ports and Sockets topic.

---

## TCP and UDP Examples

### Software Download

A software update requires complete and correct data.

A missing or corrupted part of the file could make the download unusable.

TCP is therefore a natural choice because it provides:

```
Reliable delivery
Ordered delivery
Retransmission
Error detection
```

Conceptually:

```
Software Download
→ TCP
```

---

### Live Voice Call

A live voice call is highly sensitive to delay.

A packet that arrives too late may no longer be useful.

For this type of traffic, it is often better to continue with new audio than wait for retransmission of an old audio packet.

UDP is therefore commonly used.

Conceptually:

```
Live Voice Call
→ UDP
```

The application may still implement its own mechanisms for jitter handling, packet-loss concealment, encryption, or congestion management.

---

## TCP vs UDP Summary

A simplified comparison is:

```
TCP
→ connection-oriented
→ three-way handshake
→ sequence numbers
→ acknowledgements
→ retransmission
→ ordered delivery
→ flow control
→ congestion control
→ higher protocol overhead
```

```
UDP
→ connectionless
→ no handshake
→ no built-in sequence numbers
→ no built-in acknowledgements
→ no built-in retransmission
→ no guaranteed ordering
→ no built-in flow control
→ lower protocol overhead
```

---

## Security Considerations

TCP and UDP behave differently from a firewall and security perspective.

A TCP firewall can often track connection state because TCP has:

```
SYN
ACK
FIN
RST
```

and clear connection-state transitions.

UDP has no equivalent connection establishment or termination.

Firewalls therefore track UDP communication using observed traffic and timeout-based state rather than a TCP-style connection state machine.

Other important security considerations include:

- open TCP and UDP ports can expose network services,
- unnecessary services should not be reachable,
- port numbers alone do not prove which application is running,
- UDP can be easier to spoof because there is no handshake,
- some UDP services can be abused for reflection or amplification attacks,
- TCP connection state does not replace application-layer security,
- encryption such as TLS is separate from TCP reliability.

---

## Troubleshooting TCP

Useful questions include:

```
Can the destination IP be reached?

Is the destination TCP port listening?

Does the SYN leave the client?

Does the server return SYN-ACK?

Is the final ACK sent?

Are retransmissions visible?

Are connections being reset?

Is a firewall blocking the handshake?

Is packet loss causing repeated retransmission?

Is the receive window very small?

Is the application responding after the connection is established?
```

Useful commands include:

### Linux

```
ss -tan
ss -ltn
```

Test a TCP port:

```
nc -vz <host> <port>
```

Packet capture:

```
tcpdump tcp
```

---

### Windows

Show TCP connections:

```
netstat -ano
```

Test a TCP port:

```
Test-NetConnection <host> -Port <port>
```

---

## Troubleshooting UDP

Useful questions include:

```
Is the destination UDP port listening?

Is the application receiving the datagrams?

Is a firewall dropping UDP?

Is ICMP Port Unreachable being returned?

Is packet loss occurring?

Does the application provide its own retry mechanism?

Is NAT or firewall state timing out?
```

On Linux:

```
ss -uan
ss -lun
```

Packet capture:

```
tcpdump udp
```

Because UDP does not establish a connection, troubleshooting often requires packet capture or application-specific testing.

---

## Key Takeaways

TCP and UDP are transport-layer protocols.

TCP is connection-oriented.

It normally establishes a connection using:

```
SYN
SYN-ACK
ACK
```

TCP provides:

- sequence numbers,
- acknowledgements,
- retransmission,
- ordered delivery,
- checksum-based error detection,
- flow control,
- congestion control.

TCP is suitable when correct and complete delivery is important.

UDP is connectionless.

It does not provide built-in:

- acknowledgements,
- retransmission,
- sequence numbering,
- ordered delivery,
- flow control.

If a UDP datagram is lost, UDP itself does not resend it.

The application can add its own reliability mechanisms when required.

TCP flow control protects the receiver using the receive window:

```
rwnd
```

TCP congestion control protects the network path using mechanisms that include the congestion window:

```
cwnd
```

TCP connections are normally closed gracefully using:

```
FIN
ACK
```

while:

```
RST
```

can terminate a connection immediately.

Both TCP and UDP use port numbers.

IP addresses identify systems, while ports identify application endpoints.

For example:

```
Software Update
→ TCP

Live Voice Call
→ UDP
```

The choice between TCP and UDP depends on the application's requirements.

TCP prioritizes reliable and ordered delivery.

UDP prioritizes simplicity and low transport overhead.