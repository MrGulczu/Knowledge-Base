# Fundamentals Summary

**Description:**  
A practical end-to-end walkthrough showing how the networking fundamentals covered in topics 01-13 work together when a workstation connects to a remotely published HTTPS service.

## Overview

The previous fundamentals topics describe individual networking concepts such as:

- network models,
- network types and topologies,
- Ethernet and MAC addressing,
- IPv4 and IPv6 addressing,
- subnetting,
- ARP and NDP,
- ICMP,
- TCP and UDP,
- ports and sockets,
- traffic types,
- MTU and fragmentation,
- network performance.

This topic connects those concepts into one practical example.

The goal is to answer:

> What actually happens when a workstation communicates with a remote server across the Internet?

The main example uses IPv4, TCP, NAT/PAT, and a remotely published HTTPS service.

## Example Environment

The example represents two separate private networks connected through the Internet.

```
                      CLIENT SITE

                      Workstation
                    192.168.10.25/24
                           |
                           |
                        Switch
                           |
                           |
                  LAN: 192.168.10.1/24
                 +---------------------+
                 | Edge Router / NAT   |
                 +---------------------+
                  WAN: 198.51.100.10
                           |
                           |
                    Internet / ISP
                           |
                           |
                  WAN: 203.0.113.10
                 +---------------------+
                 | Remote Edge Router  |
                 | Firewall / NAT      |
                 +---------------------+
                  LAN: 10.20.30.1/24
                           |
                           |
                        Switch
                           |
                           |
                    Remote Server
                    10.20.30.50/24
                      TCP / 443
```

```text
Published service:

203.0.113.10:443
        |
        | DNAT / Port Forward
        v
10.20.30.50:443
```
The public addresses in this example come from documentation-only IPv4 ranges:

```
198.51.100.0/24
203.0.113.0/24
```

They are used only as examples.

Related topics:

- [01 - Network Models](../01-network-models/network-models.md)
- [02 - Network Types and Topologies](../02-network-types-topologies/network-types-topologies.md)
- [04 - IP Addressing](../04-ip-addressing/ip-addressing.md)

## 1. The Application Wants to Connect

The workstation runs an application that wants to open an HTTPS connection to:

```
203.0.113.10:443
```

The operating system creates or uses a TCP socket and selects an ephemeral source port.

For example:

```
192.168.10.25:52341
        →
203.0.113.10:443
```

At this point:

```
Source IP:         192.168.10.25
Source Port:       52341

Destination IP:    203.0.113.10
Destination Port:  443

Protocol:          TCP
```

Related topics:

- [09 - TCP and UDP](../09-tcp-udp/tcp-udp.md)
- [10 - Ports and Sockets](../10-ports-sockets/ports-sockets.md)

## 2. The Workstation Checks Whether the Destination Is Local

The workstation knows its own address and prefix:

```
192.168.10.25/24
```

The local IPv4 network is therefore:

```
192.168.10.0/24
```

The destination is:

```
203.0.113.10
```

The workstation compares the destination with its own local network.

Because the destination does not belong to:

```
192.168.10.0/24
```

the workstation determines that the destination is remote.

Traffic must therefore be sent to the default gateway:

```
192.168.10.1
```

Related topics:

- [04 - IP Addressing](../04-ip-addressing/ip-addressing.md)
- [05 - IPv4 Subnetting](../05-ipv4-subnetting/ipv4-subnetting.md)

## 3. The Workstation Needs the Gateway MAC Address

The workstation knows the IP address of its next hop:

```
192.168.10.1
```

but Ethernet requires a destination MAC address.

The workstation first checks its ARP cache.

If the mapping is already present:

```
192.168.10.1
→ BB:BB:BB:BB:BB:BB
```

it can use it immediately.

If the gateway MAC address is unknown, the workstation sends an ARP Request.

Conceptually:

```
Who has 192.168.10.1?
Tell 192.168.10.25
```

The ARP Request is sent as an Ethernet broadcast.

The gateway replies with its MAC address, and the workstation stores the mapping temporarily in the ARP cache.

Related topics:

- [03 - Ethernet and MAC Addressing](../03-ethernet-mac/ethernet-mac.md)
- [07 - ARP and NDP](../07-arp-ndp/arp-ndp.md)
- [11 - Traffic Types](../11-traffic-types/traffic-types.md)

## 4. The First Ethernet Frame Is Created

The workstation now knows:

```
Final IP destination:
203.0.113.10

Local next hop:
192.168.10.1

Gateway MAC:
BB:BB:BB:BB:BB:BB
```

The initial TCP SYN is encapsulated into an IP packet and then into an Ethernet frame.

The first Ethernet frame contains:

```
Source MAC:
workstation MAC

Destination MAC:
default gateway MAC
```

The IP packet inside it contains:

```
Source IP:
192.168.10.25

Destination IP:
203.0.113.10
```

This demonstrates an important distinction:

```
MAC addresses
→ local-hop delivery

IP addresses
→ Layer 3 communication between networks
```

Related topics:

- [01 - Network Models](../01-network-models/network-models.md)
- [03 - Ethernet and MAC Addressing](../03-ethernet-mac/ethernet-mac.md)
- [04 - IP Addressing](../04-ip-addressing/ip-addressing.md)

## 5. The Switch Forwards the Frame

The switch examines the destination MAC address.

It checks its MAC address table and forwards the Ethernet frame toward the port where the gateway MAC was learned.

The switch does not need to route the packet toward:

```
203.0.113.10
```

The router will perform the Layer 3 forwarding decision.

Related topics:

- [01 - Network Models](../01-network-models/network-models.md)
- [03 - Ethernet and MAC Addressing](../03-ethernet-mac/ethernet-mac.md)

## 6. The Client Edge Router Receives the Frame

The gateway receives the Ethernet frame because the destination MAC belongs to its LAN interface.

The router removes the Layer 2 frame and examines the IPv4 packet.

Conceptually:

```
Ethernet Frame
        ↓
Remove Layer 2 information
        ↓
Inspect IPv4 packet
```

The router sees:

```
Destination IP:
203.0.113.10
```

It performs a routing-table lookup and selects the best matching route toward the destination.

The router also decreases the IPv4 TTL.

Because the IPv4 header changes, its header checksum must be recalculated.

Related topics:

- [01 - Network Models](../01-network-models/network-models.md)
- [04 - IP Addressing](../04-ip-addressing/ip-addressing.md)
- [08 - ICMP](../08-icmp/icmp.md)

## 7. Client-Side NAT / PAT

The workstation uses a private IPv4 address:

```
192.168.10.25
```

Private IPv4 addresses are not routed across the public Internet.

Before the packet leaves the client site, the edge router normally performs source NAT.

With PAT, both the source IP and possibly the source port are translated.

For example:

```
Before NAT:

192.168.10.25:52341
        →
203.0.113.10:443
```

may become:

```
After NAT/PAT:

198.51.100.10:62001
        →
203.0.113.10:443
```

The edge router keeps a translation entry such as:

```
Inside:
192.168.10.25:52341

Translated:
198.51.100.10:62001
```

This allows returning traffic to be translated back toward the original workstation.

This is an important exception to the simplified statement that IP addresses remain unchanged between routed hops.

Normal routing changes Layer 2 information hop by hop.

NAT can intentionally modify Layer 3 and Layer 4 information:

```
IP address
Port number
```

Related topics:

- [01 - Network Models](../01-network-models/network-models.md))
- [04 - IP Addressing](../04-ip-addressing/ip-addressing.md)
- [10 - Ports and Sockets](../10-ports-sockets/ports-sockets.md)

## 8. The Packet Crosses the Internet

After client-side NAT, the public Internet sees traffic approximately as:

```
198.51.100.10:62001
        →
203.0.113.10:443
```

Across routed hops, each router performs the same general process:

```
Frame received
        ↓
Layer 2 header removed
        ↓
Destination IP checked
        ↓
TTL reduced
        ↓
Routing lookup performed
        ↓
New Layer 2 frame created
        ↓
Packet forwarded
```

On Ethernet segments, the frame normally uses:

```
Source MAC
→ current device / outgoing interface

Destination MAC
→ next-hop device
```

The MAC addresses used near the client do not travel across the Internet to the remote site.

Each Layer 2 segment has its own local addressing.

Related topics:

- [01 - Network Models](../01-network-models/network-models.md)
- [03 - Ethernet and MAC Addressing](../03-ethernet-mac/ethernet-mac.md)
- [04 - IP Addressing](../04-ip-addressing/ip-addressing.md)

## 9. What Happens If TTL Reaches Zero?

Every IPv4 router decreases the TTL value.

If the value reaches:

```
0
```

before the packet reaches the destination, the router discards the packet.

It normally returns:

```
ICMP Time Exceeded
```

toward the source.

This mechanism is used by tools such as:

```
traceroute
```

Related topic:

- [08 - ICMP](../08-icmp/icmp.md)

## 10. The Packet Reaches the Remote Edge Router

The destination public address is:

```
203.0.113.10
```

This address belongs to the remote edge router.

The router receives traffic destined for:

```
203.0.113.10:443
```

and checks its firewall and NAT configuration.

The published service is configured using destination NAT or port forwarding:

```
203.0.113.10:443
        ↓
DNAT
        ↓
10.20.30.50:443
```

The router therefore changes the destination address from:

```
203.0.113.10
```

to:

```
10.20.30.50
```

The destination port remains:

```
443
```

in this example.

After translation, the packet is logically:

```
198.51.100.10:62001
        →
10.20.30.50:443
```

Related topics:

- [04 - IP Addressing](../04-ip-addressing/ip-addressing.md)
- [10 - Ports and Sockets](../10-ports-sockets/ports-sockets.md)

## 11. The Remote Router Needs the Server MAC Address

The remote edge router has an internal interface:

```
10.20.30.1/24
```

and the server is:

```
10.20.30.50/24
```

Because the server belongs to the directly connected network:

```
10.20.30.0/24
```

the router can deliver the packet directly on the LAN.

To create the Ethernet frame, it needs the server MAC address.

If the MAC address is not already in its ARP cache, the router sends an ARP Request for:

```
10.20.30.50
```

The server replies with its MAC address.

Related topics:

- [03 - Ethernet and MAC Addressing](../03-ethernet-mac/ethernet-mac.md)
- [05 - IPv4 Subnetting](../05-ipv4-subnetting/ipv4-subnetting.md)
- [07 - ARP and NDP](../07-arp-ndp/arp-ndp.md)

## 12. The Final Ethernet Frame Is Created

The remote router creates a new Ethernet frame.

Conceptually:

```
Source MAC:
remote router LAN interface

Destination MAC:
server MAC
```

The translated IP packet contains:

```
Source IP:
198.51.100.10

Destination IP:
10.20.30.50
```

and the TCP information includes:

```
Source Port:
62001

Destination Port:
443
```

The frame is forwarded through the remote LAN toward the server.

Related topics:

- [03 - Ethernet and MAC Addressing](../03-ethernet-mac/ethernet-mac.md)
- [04 - IP Addressing](../04-ip-addressing/ip-addressing.md)
- [10 - Ports and Sockets](../10-ports-sockets/ports-sockets.md)

## 13. The Server Receives and Decapsulates the Traffic

The server receives the Ethernet frame.

The networking stack processes the data through the layers:

```
Ethernet Frame
        ↓
IPv4 Packet
        ↓
TCP Segment
```

The server sees:

```
Destination IP:
10.20.30.50

Destination Port:
443

Protocol:
TCP
```

Host firewall rules and other security mechanisms may decide whether the traffic is permitted.

If permitted, the operating system checks for a matching listening socket.

The HTTPS service is listening on:

```
TCP/443
```

so the TCP SYN can be associated with that service.

Related topics:

- [01 - Network Models](../01-network-models/network-models.md)
- [09 - TCP and UDP](../09-tcp-udp/tcp-udp.md)
- [10 - Ports and Sockets](../10-ports-sockets/ports-sockets.md)

## 14. The TCP Three-Way Handshake

The first TCP packet is a SYN.

The connection is established using:

```
Client → Server
SYN

Server → Client
SYN-ACK

Client → Server
ACK
```

After the final ACK, the TCP connection is established.

During the SYN and SYN-ACK, both endpoints can advertise TCP options such as:

```
MSS
Window Scale
SACK Permitted
```

For a common Ethernet MTU of:

```
1500 bytes
```

a typical TCP MSS over IPv4 with standard headers is:

```
1500
- 20 bytes IPv4 header
- 20 bytes TCP header
= 1460 bytes
```

The TCP handshake does not directly exchange interface MTU values.

Related topics:

- [09 - TCP and UDP](../09-tcp-udp/tcp-udp.md)
- [10 - Ports and Sockets](../10-ports-sockets/ports-sockets.md)
- [12 - MTU and Fragmentation](../12-mtu-fragmentation/mtu-fragmentation.md)

## 15. Return Traffic Travels Back Through NAT

The server replies toward the source it sees:

```
198.51.100.10:62001
```

The return flow begins approximately as:

```
10.20.30.50:443
        →
198.51.100.10:62001
```

At the remote edge router, the previous DNAT state is reversed.

The source becomes:

```
203.0.113.10:443
```

so traffic leaving the remote site appears as:

```
203.0.113.10:443
        →
198.51.100.10:62001
```

When the response reaches the client edge router, the PAT state is used to translate the destination back to:

```
192.168.10.25:52341
```

The final flow delivered to the workstation becomes:

```
203.0.113.10:443
        →
192.168.10.25:52341
```

The workstation therefore sees communication with the public service:

```
203.0.113.10:443
```

even though the actual server is:

```
10.20.30.50:443
```

Related topics:

- [04 - IP Addressing](../04-ip-addressing/ip-addressing.md)
- [09 - TCP and UDP](../09-tcp-udp/tcp-udp.md)
- [10 - Ports and Sockets](../10-ports-sockets/ports-sockets.md)

## 16. Application Data Can Now Flow

Once TCP is established, application-layer communication can begin.

Because this example uses TCP/443, an HTTPS application would normally continue with TLS and HTTP processing.

Those protocols are outside the scope of this fundamentals section.

From the transport perspective, TCP provides mechanisms such as:

```
Reliable delivery
Ordered delivery
Sequence numbers
Acknowledgements
Retransmission
Flow control
Congestion control
```

Related topic:

- [09 - TCP and UDP](../09-tcp-udp/tcp-udp.md)

## 17. Large Application Data Is Segmented

An application can generate much more data than can fit inside one network packet.

TCP divides its byte stream into appropriately sized segments.

For example:

```
Application Data:
10,000 bytes
```

may be split into multiple TCP segments.

Conceptually:

```
Application Data
        ↓
TCP Segmentation
        ↓
TCP Segments
        ↓
IP Packets
        ↓
Layer 2 Frames
```

Related topics:

- [09 - TCP and UDP](../09-tcp-udp/tcp-udp.md)
- [12 - MTU and Fragmentation](../12-mtu-fragmentation/mtu-fragmentation.md)

## 18. MTU and IPv4 Fragmentation

Each Layer 3 interface has an MTU.

For normal Ethernet, a common value is:

```
1500 bytes
```

If an IPv4 packet is too large for an outgoing link and fragmentation is allowed:

```
Oversized IPv4 packet
        ↓
Router may fragment it
```

If the Don't Fragment flag prevents fragmentation:

```
Oversized IPv4 packet
        ↓
Packet dropped
        ↓
ICMP Fragmentation Needed
```

Path MTU Discovery allows endpoints to determine an appropriate packet size for the complete path.

Related topics:

- [08 - ICMP](../08-icmp/icmp.md)
- [12 - MTU and Fragmentation](../12-mtu-fragmentation/mtu-fragmentation.md)

## 19. Packet Loss and TCP Retransmission

Suppose some TCP data is lost along the path.

TCP can detect that data has not been successfully acknowledged.

Loss may be detected using mechanisms such as:

```
Duplicate acknowledgements
Retransmission timeout
```

The missing data can then be retransmitted.

Conceptually:

```
Segment 1 → received
Segment 2 → lost
Segment 3 → received
Segment 4 → received
```

The missing data associated with Segment 2 is retransmitted.

Packet loss can also reduce performance:

```
Packet Loss
        ↓
Retransmission
        ↓
Additional Delay
        ↓
Possible Congestion Response
        ↓
Lower Throughput
```

Related topics:

- [09 - TCP and UDP](../09-tcp-udp/tcp-udp.md)
- [13 - Network Performance Basics](../13-network-performance-basics/network-performance-basics.md)

## 20. Traffic Types Used in the Example

The normal HTTPS exchange is unicast:

```
One sender
        →
One specific destination
```

The ARP Requests used on the local networks are broadcasts:

```
One sender
        →
All devices in the local broadcast domain
```

TCP does not make traffic unicast by itself.

UDP traffic can also be unicast.

Traffic type and transport protocol describe different properties.

Related topic:

- [11 - Traffic Types](../11-traffic-types/traffic-types.md)

## 21. MAC Addresses Are Local to Each Segment

The workstation never needs to know the MAC address of the remote server.

The workstation only needs the MAC address of its own next hop:

```
192.168.10.1
```

The remote server likewise only needs Layer 2 information relevant to its own local network.

Conceptually:

```
Workstation
→ local gateway MAC

Router
→ next-hop Layer 2 address

Router
→ next-hop Layer 2 address

Remote router
→ server MAC
```

MAC addresses are therefore used for local Layer 2 delivery.

IP addresses are used for Layer 3 communication between networks.

Related topics:

- [03 - Ethernet and MAC Addressing](../03-ethernet-mac/ethernet-mac.md)
- [07 - ARP and NDP](../07-arp-ndp/arp-ndp.md)

## 22. Network Types Along the Path

The workstation may be connected through:

```
LAN
```

or:

```
WLAN
```

The traffic then leaves the local site through the edge router and crosses provider or Internet infrastructure.

A simplified view is:

```
Client LAN / WLAN
        ↓
Client Edge Router
        ↓
Provider / WAN / Internet
        ↓
Remote Edge Router
        ↓
Server LAN
```

A MAN specifically refers to a network covering a metropolitan area.

A WAN generally connects geographically separated networks or sites.

In everyday administration, WAN is also commonly used to describe the external or provider-facing side of an edge router.

Related topic:

- [02 - Network Types and Topologies](../02-network-types-topologies/network-types-topologies.md)

## 23. What Changes With IPv6?

The overall communication logic remains similar:

```
Application
        ↓
Transport Protocol
        ↓
Destination Checked
        ↓
Next Hop Selected
        ↓
Layer 2 Delivery
        ↓
Routing
        ↓
Destination
```

However, several mechanisms change.

### ARP Becomes NDP

IPv6 does not use ARP.

Neighbor Discovery Protocol performs neighbor discovery using ICMPv6.

Instead of an ARP broadcast, IPv6 normally uses Neighbor Solicitation with multicast.

### No IPv6 Broadcast

IPv6 does not use IP broadcast.

Multicast is used for many functions that relied on broadcast in IPv4.

### Addressing Changes

IPv4 example:

```
192.168.10.25
```

IPv6 uses 128-bit hexadecimal addresses.

### TTL Becomes Hop Limit

IPv4:

```
TTL
```

IPv6:

```
Hop Limit
```

Both prevent packets from circulating indefinitely.

### ICMP Becomes ICMPv6

IPv6 uses ICMPv6.

ICMPv6 is especially important for:

```
Neighbor Discovery
Router Discovery
Path MTU Discovery
```

### Routers Do Not Fragment IPv6 Packets

IPv6 routers never fragment packets in transit.

If a packet is too large:

```
Router
        ↓
Drops Packet
        ↓
ICMPv6 Packet Too Big
        ↓
Source Adjusts Packet Size
```

Only the IPv6 source can perform IPv6 fragmentation.

### NAT Is Normally Not Required for Address Conservation

A normal IPv6 deployment can use globally routable addresses internally without traditional IPv4-style address conservation NAT.

Security should be provided by firewall policy, not by relying on NAT.

Related topics:

- [06 - IPv6 Basics](../06-ipv6-basics/ipv6-basics.md)
- [07 - ARP and NDP](../07-arp-ndp/arp-ndp.md)
- [08 - ICMP](../08-icmp/icmp.md)
- [11 - Traffic Types](../11-traffic-types/traffic-types.md)
- [12 - MTU and Fragmentation](../12-mtu-fragmentation/mtu-fragmentation.md)

## 24. Performance Across the Same Connection

A network connection can be technically working while still providing poor performance.

Important metrics include:

```
Bandwidth
Throughput
Goodput
Latency
RTT
Jitter
Packet Loss
PPS
```

A slow or heavily utilized link can become the bottleneck for the complete path.

High latency can make interactive applications feel slow.

High jitter affects real-time traffic.

Packet loss can trigger TCP retransmissions and congestion-control behavior.

Heavy queueing can increase latency significantly even while throughput remains high.

Related topic:

- [13 - Network Performance Basics](../13-network-performance-basics/network-performance-basics.md)


## 25. End-to-End Encapsulation View

At the workstation:

```
Application Data
        ↓
TCP Segment
        ↓
IPv4 Packet
        ↓
Ethernet Frame
        ↓
Bits / Signals
```

At a router:

```
Layer 2 Frame Received
        ↓
Layer 2 Header Removed
        ↓
IP Packet Examined
        ↓
Routing / NAT Processing
        ↓
New Layer 2 Header Added
        ↓
Frame Transmitted
```

At the destination server:

```
Bits / Signals
        ↓
Ethernet Frame
        ↓
IPv4 Packet
        ↓
TCP Segment
        ↓
Application Data
```

Related topic:

- [01 - Network Models](../01-network-models/network-models.md)


## 26. Address Changes Across the Complete Path

One of the most useful ways to understand this example is to follow the addresses.

### Workstation to Client Gateway

```
Ethernet:
Workstation MAC
        →
Client Gateway MAC

IP:
192.168.10.25
        →
203.0.113.10

TCP:
52341
        →
443
```

### After Client PAT

```
IP:
198.51.100.10
        →
203.0.113.10

TCP:
62001
        →
443
```

### Across the Internet

The Layer 2 information changes between links, while the public Layer 3 / Layer 4 flow remains approximately:

```
198.51.100.10:62001
        →
203.0.113.10:443
```

### After Remote DNAT

```
198.51.100.10:62001
        →
10.20.30.50:443
```

### Final Local Hop

```
Ethernet:
Remote Router MAC
        →
Server MAC

IP:
198.51.100.10
        →
10.20.30.50

TCP:
62001
        →
443
```

This makes the difference clear:

```
Routing
→ changes local Layer 2 information

NAT
→ can change IP addresses

PAT
→ can also change port numbers
```

## 27. Common Failure Points

The complete communication path provides a useful troubleshooting model.

### Incorrect IP Configuration

A wrong IP address, subnet mask, or gateway can prevent the workstation from making the correct forwarding decision.

Related topics:

- [04 - IP Addressing](../04-ip-addressing/ip-addressing.md)
- [05 - IPv4 Subnetting](../05-ipv4-subnetting/ipv4-subnetting.md)


### ARP Failure

The workstation may know the gateway IP but be unable to resolve its MAC address.

Layer 3 configuration can be correct while Layer 2 delivery still fails.

Related topic:

- [07 - ARP and NDP](../07-arp-ndp/arp-ndp.md)

### Switching Problem

The Ethernet frame may not reach the gateway or server because of a Layer 2 problem.

Related topic:

- [03 - Ethernet and MAC Addressing](../03-ethernet-mac/ethernet-mac.md)

### Routing Problem

A router may have no usable route toward the destination.

Related topics:

- [04 - IP Addressing](../04-ip-addressing/ip-addressing.md)
- [08 - ICMP](../08-icmp/icmp.md)

### NAT / PAT Problem

Client-side translation may fail, preventing private traffic from reaching the Internet.

Remote DNAT may also be missing or incorrect, preventing the public service from reaching the internal server.

Related topics:

- [04 - IP Addressing](../04-ip-addressing/ip-addressing.md)
- [10 - Ports and Sockets](../10-ports-sockets/ports-sockets.md)

### TTL Reaches Zero

The packet is discarded and ICMP Time Exceeded is normally returned.

Related topic:

- [08 - ICMP](../08-icmp/icmp.md)

### Firewall Filtering

A firewall may block the SYN before the server receives it.

The IP path can therefore work while the TCP connection still fails.

Related topic:

- [10 - Ports and Sockets](../10-ports-sockets/ports-sockets.md)

### No Listening Service

The server may be reachable while nothing is listening on TCP/443.

Related topic:

- [10 - Ports and Sockets](../10-ports-sockets/ports-sockets.md)

### MTU / PMTUD Problem

Small packets may work while larger transfers fail or stall.

Related topics:

- [08 - ICMP](../08-icmp/icmp.md)
- [12 - MTU and Fragmentation](../12-mtu-fragmentation/mtu-fragmentation.md)

### Packet Loss

TCP retransmits missing data and may reduce its sending rate.

Related topics:

- [09 - TCP and UDP](../09-tcp-udp/tcp-udp.md)
- [13 - Network Performance Basics](../13-network-performance-basics/network-performance-basics.md)

### High Latency or Jitter

The connection may work correctly but provide poor user experience.

Related topic:

- [13 - Network Performance Basics](../13-network-performance-basics/network-performance-basics.md)

## 28. Complete Simplified Flow

The complete example can now be summarized as:

```
Application wants to connect to 203.0.113.10:443
        ↓
TCP socket and ephemeral source port are selected
        ↓
Workstation checks whether destination is local
        ↓
Destination is remote
        ↓
Default gateway 192.168.10.1 is selected
        ↓
ARP cache is checked
        ↓
ARP Request / Reply if gateway MAC is unknown
        ↓
TCP SYN is encapsulated into IPv4
        ↓
IPv4 packet is encapsulated into Ethernet
        ↓
Switch forwards frame toward gateway
        ↓
Client edge router removes Layer 2 frame
        ↓
TTL is reduced
        ↓
Routing lookup is performed
        ↓
SNAT / PAT translates:
192.168.10.25:52341
        →
198.51.100.10:62001
        ↓
Packet crosses the Internet
        ↓
Routers repeat Layer 3 forwarding
        ↓
Remote edge receives:
198.51.100.10:62001
        →
203.0.113.10:443
        ↓
Firewall / NAT policy is checked
        ↓
DNAT translates:
203.0.113.10:443
        →
10.20.30.50:443
        ↓
Remote router checks ARP cache
        ↓
ARP resolves server MAC if necessary
        ↓
Final Ethernet frame reaches server
        ↓
Server decapsulates Ethernet and IPv4
        ↓
TCP destination port 443 is checked
        ↓
Listening socket is found
        ↓
Server receives SYN
        ↓
Server sends SYN-ACK
        ↓
Return traffic crosses reverse NAT state
        ↓
Client receives SYN-ACK
        ↓
Client sends ACK
        ↓
TCP connection established
        ↓
Application-layer communication begins
```

## Key Takeaways

The fundamentals topics are not isolated concepts.

A single connection can involve:

```
Network Models
Network Types
Ethernet
MAC Addresses
IPv4
Subnetting
Default Gateways
ARP
Switching
Routing
NAT
PAT
TTL
ICMP
TCP
Ports
Sockets
Traffic Types
MTU
MSS
Path MTU Discovery
Packet Loss
Latency
Throughput
```

The most important relationships are:

```
MAC
→ local Layer 2 delivery

IP
→ communication between networks

Subnet
→ determines what is local and remote

Default Gateway
→ next hop for remote networks

ARP / NDP
→ neighbor Layer 2 resolution

Routing
→ next-hop path selection

NAT
→ changes IP addressing

PAT
→ may also change transport ports

TCP
→ reliable transport

Port
→ identifies application endpoints

ICMP / ICMPv6
→ control and error reporting

MTU
→ packet-size constraints

Performance Metrics
→ describe how well the path operates
```

Understanding the complete path makes troubleshooting easier because a problem can be associated with a particular layer or mechanism.

Instead of asking only:

```
Why does the network not work?
```

the communication can be broken into smaller questions:

```
Is the physical link working?

Is Layer 2 forwarding working?

Can the workstation resolve its gateway MAC?

Is the IP configuration correct?

Does the workstation know the destination is remote?

Is the default gateway reachable?

Does the router have a usable route?

Is client-side NAT/PAT working?

Can traffic cross the WAN?

Is remote DNAT configured correctly?

Does the remote router know the server MAC?

Does the firewall allow the traffic?

Is TCP/443 listening?

Does the TCP handshake complete?

Is MTU causing packet-size problems?

Is packet loss reducing performance?

Is latency or jitter affecting the connection?
```
