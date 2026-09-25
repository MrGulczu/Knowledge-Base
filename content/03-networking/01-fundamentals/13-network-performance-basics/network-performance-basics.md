# Network Performance Basics

## Overview

Network performance is not described by a single number.

A connection may have very high bandwidth but still feel slow because of:

- high latency,
- jitter,
- packet loss,
- congestion,
- buffering,
- a slow bottleneck somewhere in the path.

Understanding these metrics is important when troubleshooting networks because different applications are affected by different performance problems.

The most important concepts covered in this topic are:

```
Bandwidth
Throughput
Goodput
Latency
Round-Trip Time
Jitter
Packet Loss
Packets Per Second
Bottlenecks
Available Bandwidth
Bandwidth-Delay Product
Bufferbloat
```

---

## Bandwidth

Bandwidth describes the capacity of a network link.

For example:

```
100 Mbps
1 Gbps
10 Gbps
```

If an Ethernet interface negotiates:

```
1 Gbps
```

this means the link is capable of carrying traffic at approximately that line rate under suitable conditions.

It does not mean every application will always transfer data at 1 Gbps.

Conceptually:

```
Bandwidth
→ maximum capacity of the link
```

A link can have high bandwidth while the actual data transfer rate is much lower.

---

## Link Speed

The link speed is the negotiated or configured speed of a physical or logical network connection.

For example:

```
Ethernet Link:
1 Gbps
```

This does not guarantee:

```
Application Transfer:
1 Gbps
```

Actual performance can be reduced by:

- protocol overhead,
- congestion,
- packet loss,
- storage performance,
- CPU performance,
- remote-server limitations,
- wireless conditions,
- firewall or router processing,
- another slower link along the path.

---

## Throughput

Throughput describes the actual amount of traffic transferred successfully during a period of time.

For example:

```
Link Capacity:
1 Gbps

Measured Throughput:
620 Mbps
```

In this case:

```
Bandwidth
→ 1 Gbps

Throughput
→ 620 Mbps
```

Throughput is therefore normally lower than the theoretical maximum capacity of a link.

The exact meaning of throughput depends on where it is measured.

A measurement taken at one layer may include protocol headers that are not part of the application's useful data.

---

## Goodput

Goodput describes the rate of useful application data successfully delivered.

Conceptually:

```
Bandwidth
→ maximum link capacity

Throughput
→ actual traffic transferred

Goodput
→ useful application payload
```

For example, if a connection carries:

```
900 Mbps
```

of total traffic, the application may receive less than 900 Mbps of useful file data because part of the traffic consists of:

- Ethernet headers,
- IP headers,
- TCP or UDP headers,
- TLS overhead,
- acknowledgements,
- retransmissions,
- other protocol information.

Goodput is therefore normally lower than raw throughput.

---

## Mbps vs MB/s

Network speeds are normally expressed in:

```
bits per second
```

while file-transfer applications often display:

```
Bytes per second
```

The important relationship is:

```
1 Byte = 8 bits
```

Therefore:

```
800 Mbps ÷ 8
= 100 MB/s
```

So an:

```
800 Mbps
```

connection has a theoretical maximum transfer rate of approximately:

```
100 MB/s
```

before accounting for overhead and other performance limitations.

Capitalization matters:

```
b
→ bit

B
→ Byte
```

Therefore:

```
Mbps
→ megabits per second

MB/s
→ megabytes per second
```

---

## Latency

Latency describes how long it takes data to travel from one point to another.

A simple definition is:

> Latency is the delay experienced while data travels across a network path.

For example:

```
Link A
Bandwidth: 1 Gbps
Latency:   5 ms
```

and:

```
Link B
Bandwidth: 1 Gbps
Latency:   120 ms
```

Both links may have the same bandwidth, but Link B will feel much less responsive.

Latency is particularly important for interactive applications such as:

- online gaming,
- voice calls,
- video calls,
- remote desktop,
- SSH,
- interactive web applications.

High latency is often experienced by users as:

```
lag
```

---

## Round-Trip Time

Round-Trip Time, or RTT, is the time required for traffic to travel to a destination and for a response to return.

Conceptually:

```
Sender
   |
   | request
   v
Destination
   |
   | response
   v
Sender
```

The total time is:

```
RTT
```

Tools such as:

```
ping
```

normally report round-trip time rather than one-way latency.

For example:

```
Reply time:
20 ms
```

normally represents approximately:

```
Sender → Destination → Sender
```

---

## Jitter

Jitter is the variation in packet delay over time.

A stable connection may look like:

```
20 ms
21 ms
19 ms
20 ms
22 ms
```

This represents relatively low jitter.

An unstable connection may look like:

```
20 ms
85 ms
25 ms
140 ms
18 ms
```

This represents high jitter.

Conceptually:

```
Latency
→ how long delivery takes

Jitter
→ how much that delay changes
```

High jitter is especially problematic for real-time applications.

Possible symptoms include:

- audio gaps,
- robotic voice,
- video freezes,
- stuttering,
- inconsistent game response.

Real-time applications may use jitter buffers to compensate for small variations in arrival time.

However, very large variations cannot always be hidden effectively.

---

## Packet Loss

Packet loss describes packets that fail to reach their intended destination.

It is often expressed as a percentage.

For example:

```
Packets Sent:
1000

Packets Received:
990

Packets Lost:
10
```

Packet loss is:

```
1%
```

Packet loss can occur because of:

- congestion,
- overloaded queues,
- faulty cabling,
- wireless interference,
- failing hardware,
- packet-processing limits,
- firewall or policy drops,
- routing problems.

---

## Packet Loss and TCP

TCP provides reliable delivery.

If TCP detects missing data, it can retransmit it.

TCP uses mechanisms such as:

- sequence numbers,
- acknowledgements,
- retransmission timers,
- duplicate acknowledgements.

Conceptually:

```
Packet Lost
    ↓
Missing data detected
    ↓
Retransmission
```

Packet loss can reduce TCP performance significantly.

TCP may interpret loss as a sign of congestion and reduce its sending rate.

Therefore packet loss can cause:

```
Retransmissions
        ↓
Lower congestion window
        ↓
Lower throughput
        ↓
Longer transfer time
```

Even a relatively small amount of loss can noticeably affect TCP performance, especially on high-latency paths.

---

## Packet Loss and UDP

UDP does not provide built-in retransmission.

If a UDP datagram is lost:

```
Datagram 1
Datagram 2
Datagram 3
```

may arrive as:

```
Datagram 1
Datagram 3
```

UDP itself does not resend Datagram 2.

The application decides whether the loss should be ignored or recovered.

For real-time applications, retransmitting old data may not be useful.

Possible effects include:

- missing audio,
- video artifacts,
- dropped frames,
- temporary stutter,
- missing game updates.

Applications can implement their own reliability mechanisms above UDP.

For example, QUIC uses UDP as its underlying transport but implements features such as:

- reliable delivery,
- acknowledgements,
- retransmission,
- congestion control.

HTTP/3 uses QUIC.

---

## Packets Per Second

Packets Per Second, or PPS, measures how many individual packets a device processes each second.

Conceptually:

```
bps
→ amount of data transferred per second

PPS
→ number of packets processed per second
```

A device can sometimes reach its packet-processing limit before reaching its bandwidth limit.

For example:

```
Traffic A
→ fewer large packets

Traffic B
→ very large number of small packets
```

Both may use similar bandwidth, but Traffic B may require significantly more processing.

Every packet may require operations such as:

- header parsing,
- routing lookup,
- switching lookup,
- firewall rule processing,
- NAT processing,
- queue handling,
- checksum handling,
- connection tracking.

This means a router, firewall, server, or switch can become overloaded by very high PPS even when the raw bandwidth is below the link's maximum capacity.

---

## PPS and Denial-of-Service Attacks

High packet rates can be used in denial-of-service attacks.

An attacker may attempt to overwhelm:

- CPU resources,
- packet-processing capacity,
- connection tracking,
- firewall state tables,
- interface queues.

This can happen even when the attacker does not completely saturate the available bandwidth.

Therefore both:

```
Bandwidth usage
```

and:

```
Packet rate
```

are important when analyzing network load.

---

## Bottlenecks

End-to-end performance is limited by the most constrained part of the network path.

Consider:

```
PC
 |
 | 1 Gbps
 v
Switch
 |
 | 1 Gbps
 v
Router
 |
 | 100 Mbps
 v
Internet
```

The 100 Mbps link is the bottleneck.

The theoretical end-to-end throughput cannot exceed approximately:

```
100 Mbps
```

even though the PC and switch are connected at 1 Gbps.

Conceptually:

```
End-to-End Throughput
→ limited by the bottleneck
```

Actual application throughput will normally be lower because of protocol overhead and current network conditions.

---

## Link Capacity vs Available Bandwidth

The slowest configured interface is not always the only limitation.

Consider a:

```
1 Gbps
```

link already carrying:

```
950 Mbps
```

of other traffic.

Only a relatively small amount of capacity may remain for a new connection.

This introduces an important distinction:

```
Link Capacity
→ theoretical maximum capacity

Available Bandwidth
→ capacity currently available for additional traffic
```

A higher-capacity link can therefore provide less usable bandwidth than a slower but idle link if it is already heavily utilized.

---

## Routing and Performance

Routing determines which path traffic takes through a network.

However, normal routing protocols do not generally select paths based directly on current real-time interface utilization.

They usually make decisions using metrics and policies.

Examples include:

- route cost,
- administrative preference,
- path length,
- routing-policy attributes.

More advanced technologies can distribute traffic according to additional conditions.

---

## Bandwidth vs Latency

Different applications care about different performance characteristics.

Consider two connections:

```
Connection A
Bandwidth: 1 Gbps
Latency:   150 ms
```

```
Connection B
Bandwidth: 100 Mbps
Latency:   5 ms
```

For a large backup transfer, Connection A may be preferable because higher throughput is more important.

For competitive online gaming, Connection B may be preferable because low latency matters more.

A useful generalization is:

```
Bulk Transfers
→ bandwidth and throughput are very important

Interactive Applications
→ latency and jitter are very important
```

---

## Application Performance Examples

### File Transfer

Important metrics:

```
Bandwidth
Throughput
Goodput
Packet Loss
```

Higher latency can also affect TCP performance, especially on long-distance links.

### Online Gaming

Important metrics:

```
Latency
Jitter
Packet Loss
```

Most games do not require extremely high bandwidth.

A low-latency 100 Mbps connection can provide a better gaming experience than a high-latency 1 Gbps connection.

### Voice and Video Calls

Important metrics:

```
Latency
Jitter
Packet Loss
Available Bandwidth
```

Consistent packet delivery is often more important than extremely high raw bandwidth.

### Backup and Large File Transfer

Important metrics:

```
Throughput
Goodput
Available Bandwidth
Packet Loss
```

Large transfers benefit significantly from higher sustained throughput.

---

## Bandwidth-Delay Product

Bandwidth-Delay Product, or BDP, describes how much data can be in flight across a path.

For TCP, it is commonly considered using the bandwidth and round-trip time.

A simplified formula is:

```
BDP = Bandwidth × RTT
```

For example:

```
Bandwidth:
1 Gbps

RTT:
100 ms
```

Convert the RTT:

```
100 ms = 0.1 seconds
```

Then:

```
1,000,000,000 bits/s × 0.1 s
= 100,000,000 bits
```

Convert to Bytes:

```
100,000,000 ÷ 8
= 12,500,000 Bytes
≈ 12.5 MB
```

So approximately:

```
12.5 MB
```

of data would need to be in flight to fully utilize the theoretical 1 Gbps path at 100 ms RTT.

---

## Data In Flight

Data in flight is data that has been sent but has not yet been acknowledged.

Conceptually:

```
Sender
   |
   |------ data travelling ------|
   |------ data travelling ------|
   |------ data travelling ------|
   v
Receiver

Acknowledgements have not yet returned
```

A high-bandwidth, high-latency path requires a larger amount of data in flight to fully use the available capacity.

This is one reason TCP window sizing can affect performance over long-distance or high-speed connections.

---

## Buffering and Queues

Network interfaces and devices use queues to temporarily store packets when traffic cannot immediately be transmitted.

For example:

```
Incoming Traffic
→ 1 Gbps

Outgoing Link
→ 500 Mbps
```

Packets may begin accumulating in a queue.

Conceptually:

```
Packets arrive
     ↓
Outgoing link busy
     ↓
Packets wait in queue
     ↓
Latency increases
```

Short queues can absorb temporary bursts of traffic.

However, excessive queueing can create performance problems.

---

## Bufferbloat

Bufferbloat occurs when excessively large or poorly managed queues allow packets to wait for too long.

It does not simply mean:

```
Buffer is full
→ packets are dropped
```

The defining problem is:

```
Excessive queueing
→ excessive latency
```

For example:

```
Connection idle:
Ping = 10 ms
```

During a large download:

```
Download = 900 Mbps
Ping = 200 ms
```

The connection still has high throughput, but interactive traffic becomes much less responsive.

Possible symptoms include:

- gaming lag,
- poor voice quality,
- slow remote desktop response,
- SSH delay,
- high jitter.

---

## Queue Management

Queue-management mechanisms can help prevent queues from becoming excessively large.

Examples include:

```
AQM
→ Active Queue Management

SQM
→ Smart Queue Management
```

These mechanisms can reduce queueing delay and improve responsiveness under load.

Detailed Quality of Service and queue-management design belongs in more advanced networking topics.

---

## Performance Is End-to-End

A fast local network does not guarantee fast application performance.

For example:

```
PC
→ 2.5 Gbps Ethernet
```

does not mean an Internet download will operate at:

```
2.5 Gbps
```

The entire path matters.

Possible limitations include:

- local interface speed,
- switch capacity,
- firewall performance,
- router performance,
- WAN bandwidth,
- ISP congestion,
- remote-server limits,
- long-distance latency,
- packet loss,
- application limits,
- disk or CPU performance.

Performance troubleshooting should therefore consider the complete path rather than only the local interface.

---

## Measuring Network Performance

Different tools measure different aspects of network performance.

### Ping

Useful for:

```
RTT
Packet Loss
Basic Reachability
```

Example:

```
ping <destination>
```

### Traceroute

Useful for examining the path toward a destination.

Linux:

```
traceroute <destination>
```

Windows:

```
tracert <destination>
```

Traceroute can help identify where latency changes significantly, but individual routers may treat traceroute responses differently from forwarded traffic.

### iperf3

`iperf3` is commonly used to measure network throughput between controlled endpoints.

Server:

```
iperf3 -s
```

Client:

```
iperf3 -c <server>
```

Testing between controlled endpoints is often more useful than an Internet speed test when diagnosing LAN or WAN performance.

---

## Troubleshooting Performance Problems

Useful questions include:

```
What is the negotiated link speed?

What throughput is actually achieved?

What is the application goodput?

Where is the bottleneck?

How much bandwidth is currently available?

What is the RTT?

Is latency stable?

Is jitter high?

Is packet loss present?

Does packet loss increase under load?

Does latency increase significantly during downloads?

Is the device reaching a PPS or CPU limit?

Are interface queues dropping packets?

Is the remote system the bottleneck?

Does the problem affect TCP, UDP, or both?
```

A performance problem should not automatically be described as:

```
The network is slow.
```

Instead, identify which metric is actually degraded.

For example:

```
Bandwidth limited
High latency
High jitter
Packet loss
High PPS load
Bufferbloat
Remote-server limitation
```

---

## Key Takeaways

Bandwidth describes the maximum capacity of a network link.

```
Bandwidth
→ theoretical link capacity
```

Throughput describes the actual rate of transferred traffic.

```
Throughput
→ actual traffic transferred
```

Goodput describes the useful application data delivered after protocol overhead and retransmissions are considered.

```
Goodput
→ useful application payload
```

Network speeds are usually expressed in bits per second, while file-transfer tools often display Bytes per second.

```
8 bits = 1 Byte
```

Therefore:

```
800 Mbps
≈ 100 MB/s theoretical maximum
```

Latency describes delivery delay.

RTT describes the time required for traffic to travel to the destination and back.

Jitter describes variation in latency.

Packet loss describes packets that fail to reach their destination.

TCP normally retransmits missing data, while UDP itself does not.

PPS describes the number of packets processed per second.

A device can reach its packet-processing limit before reaching its bandwidth limit.

The slowest or most constrained part of an end-to-end path acts as the bottleneck.

Link capacity and available bandwidth are not always the same.

Bandwidth-Delay Product describes how much data must be in flight to utilize a path efficiently.

Bufferbloat occurs when excessive queueing creates very high latency under load.

Different applications care about different metrics:

```
Large File Transfers
→ throughput and goodput

Gaming
→ latency, jitter, packet loss

Voice / Video
→ latency, jitter, packet loss

Network Devices
→ bandwidth and PPS
```

Network performance should therefore always be evaluated using multiple metrics rather than only the advertised link speed.