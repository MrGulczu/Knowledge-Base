# Traffic Types

## Overview

Network traffic can be delivered in several different ways depending on how many receivers should receive the data and how the destination is addressed.

The main traffic types covered in this article are:

```
Unicast
Broadcast
Multicast
Anycast
```

These concepts are important because they affect how hosts, switches, routers, and applications handle network communication.

---

## Unicast

Unicast is communication between one sender and one specific receiver.

Conceptually:

```
One sender
    ↓
One receiver
```

For example:

```
192.168.0.1:54443
        →
203.0.113.20:443
TCP
```

In this case, one client communicates with one specific server.

Unicast does not require a connection-oriented protocol.

For example, UDP can also use unicast:

```
192.168.0.10:53000
        →
8.8.8.8:53
UDP
```

Therefore:

```
Unicast
→ one sender
→ one specific receiver
```

Unicast is the most common traffic type for normal client-to-server communication.

Examples include:

- web browsing,
- SSH sessions,
- file transfers,
- DNS queries sent to a specific DNS server.

---

## Broadcast

Broadcast traffic is sent from one host to all hosts in the same broadcast domain.

Conceptually:

```
One sender
    ↓
All hosts in the broadcast domain
```

Broadcast is commonly associated with IPv4.

A host may send traffic to:

```
255.255.255.255
```

or to a subnet-specific broadcast address such as:

```
192.168.10.255
```

for the subnet:

```
192.168.10.0/24
```

Every host in that broadcast domain can receive the traffic, although the operating system or application may ignore it if it is not relevant.

---

## Broadcast Domains

Broadcast traffic is normally limited to the local broadcast domain.

Routers do not normally forward broadcasts between subnets.

For example:

```
VLAN 10
   |
Broadcast
   |
Router
   X
   |
VLAN 20
```

The router separates the two broadcast domains.

This is one reason VLANs and routed interfaces are important in larger networks: they prevent broadcasts from spreading across the entire network.

---

## DHCP Broadcast Example

DHCP is a common example of IPv4 broadcast traffic.

A client that does not yet have an IPv4 address can send a DHCP message using:

```
Source IP:
0.0.0.0

Destination IP:
255.255.255.255

Source Port:
68/UDP

Destination Port:
67/UDP
```

The client uses broadcast because it may not yet know:

- its own IP address,
- the DHCP server address,
- the local subnet configuration.

---

## DHCP Relay

Broadcast traffic itself does not normally cross a router.

However, a router or Layer 3 switch can be configured as a DHCP relay agent.

Conceptually:

```
Client
   |
   | DHCP broadcast
   v
DHCP Relay
   |
   | forwarded toward DHCP server
   v
DHCP Server
```

The relay receives the local DHCP broadcast and forwards the request toward a configured DHCP server using routed communication.

The relay also provides information that helps the DHCP server determine which client subnet the request came from.

This allows one centralized DHCP server to provide addresses to clients in multiple VLANs.

---

## Limited Broadcast

The IPv4 address:

```
255.255.255.255
```

is called the limited broadcast address.

It means:

```
send to all hosts on the local IPv4 network
```

This is useful when a host does not yet know enough about the network to calculate the subnet's broadcast address.

DHCP discovery is a common example.

Routers do not forward limited broadcasts.

---

## Directed Broadcast

A subnet-specific broadcast address is called a directed broadcast address.

For example:

```
Network:
192.168.10.0/24

Broadcast:
192.168.10.255
```

The address:

```
192.168.10.255
```

represents all hosts in:

```
192.168.10.0/24
```

Conceptually:

```
255.255.255.255
→ local limited broadcast

192.168.10.255
→ broadcast directed at the 192.168.10.0/24 subnet
```

Although directed broadcasts identify a specific subnet, routers commonly do not forward them by default.

One reason is historical abuse in amplification attacks such as Smurf attacks.

---

## Ethernet Broadcast

At Layer 2, Ethernet uses the broadcast MAC address:

```
FF:FF:FF:FF:FF:FF
```

For example, an IPv4 broadcast packet may be carried inside an Ethernet frame with:

```
Destination MAC:
FF:FF:FF:FF:FF:FF

Destination IPv4:
255.255.255.255
```

A switch receiving an Ethernet broadcast frame normally floods it out all relevant ports in the same VLAN, except the port on which it was received.

This gives us an important distinction:

```
Layer 2 broadcast
→ FF:FF:FF:FF:FF:FF

Layer 3 IPv4 broadcast
→ 255.255.255.255
   or a subnet-directed broadcast address
```

---

## IPv6 Does Not Use Broadcast

IPv6 does not use broadcast.

Instead, IPv6 relies heavily on multicast.

For example, the IPv6 multicast address:

```
ff02::1
```

means:

```
all IPv6 nodes on the local link
```

Another example is:

```
ff02::2
```

which means:

```
all IPv6 routers on the local link
```

IPv6 Neighbor Discovery also uses multicast instead of an IPv4-style broadcast mechanism.

Therefore:

```
IPv4
→ supports broadcast

IPv6
→ no broadcast
→ uses multicast instead
```

---

## Multicast

Multicast is communication from one sender to a selected group of receivers.

Conceptually:

```
One sender
    ↓
Selected group of receivers
```

Unlike broadcast, multicast is not intended for every host in the broadcast domain.

Only hosts interested in a particular multicast group need the traffic.

For example:

```
Sender
   |
   | multicast stream
   v
Multicast Group
   |
   +---- Receiver A
   |
   +---- Receiver B
   |
   +---- Receiver C
```

A common use case is live video or IPTV distribution.

Instead of sending a separate copy of the same stream to every receiver, the sender can transmit traffic to a multicast group.

---

## Broadcast vs Multicast

The main difference is the intended audience.

```
Broadcast
→ one sender to everyone in the broadcast domain

Multicast
→ one sender to a selected group
```

Multicast can therefore reduce unnecessary traffic when many receivers need the same data but not every host should receive it.

---

## IPv4 Multicast MAC Addresses

IPv4 multicast traffic uses special Ethernet multicast MAC addresses.

IPv4 multicast destination MAC addresses begin with:

```
01:00:5E
```

Conceptually:

```
IPv4 multicast address
        ↓
Mapped to Ethernet multicast MAC
        ↓
01:00:5E:xx:xx:xx
```

Only part of the IPv4 multicast address is mapped into the Ethernet MAC address.

Because of this, more than one IPv4 multicast group can sometimes map to the same Ethernet multicast MAC address.

The receiving host may therefore still need to inspect the IP packet to determine whether the traffic belongs to the expected multicast group.

---

## IPv6 Multicast MAC Addresses

IPv6 multicast traffic also uses special Ethernet multicast MAC addresses.

IPv6 multicast MAC addresses begin with:

```
33:33
```

The final 32 bits of the IPv6 multicast address are mapped into the final 32 bits of the Ethernet destination MAC.

For example:

```
IPv6 multicast:
ff02::1
```

maps to:

```
Ethernet multicast MAC:
33:33:00:00:00:01
```

Therefore:

```
Ethernet broadcast
→ FF:FF:FF:FF:FF:FF

IPv4 multicast
→ starts with 01:00:5E

IPv6 multicast
→ starts with 33:33
```

---

## IGMP

In IPv4 networks, hosts use:

```
IGMP
Internet Group Management Protocol
```

to communicate multicast group membership.

A host can indicate that it is interested in receiving traffic for a particular multicast group.

Conceptually:

```
Host
   |
   | IGMP membership information
   v
Local network
```

IGMP is concerned with IPv4 multicast group membership.

---

## IGMP Snooping

A switch can use:

```
IGMP Snooping
```

to observe IGMP messages and learn which switch ports have receivers interested in particular multicast groups.

For example:

```
Multicast Group:
239.1.1.10

Switch learns:

Port 2
→ interested receiver

Port 5
→ interested receiver

Port 8
→ no interested receiver
```

The switch can then forward the multicast traffic only where it is needed.

Conceptually:

```
IGMP
→ hosts communicate IPv4 multicast membership

IGMP Snooping
→ switch observes IGMP traffic
→ learns which ports need the multicast
```

Without multicast-aware forwarding, multicast traffic may be flooded more broadly inside the VLAN.

---

## MLD

IPv6 uses:

```
MLD
Multicast Listener Discovery
```

for multicast listener membership.

MLD performs a similar role for IPv6 that IGMP performs for IPv4.

Switches can also use:

```
MLD Snooping
```

to learn which ports have IPv6 multicast listeners.

Therefore:

```
IPv4:
IGMP
IGMP Snooping

IPv6:
MLD
MLD Snooping
```

---

## Multicast Routing

Multicast traffic is not normally forwarded automatically between routed networks.

A router needs multicast-routing support and appropriate configuration.

Conceptually:

```
Sender
   |
VLAN 10
   |
Router
   X
   |
VLAN 20
```

Without multicast routing, the traffic does not simply cross into another subnet.

With multicast routing configured:

```
Sender
   |
VLAN 10
   |
Multicast-capable router
   |
VLAN 20
   |
Interested receivers
```

A useful distinction is:

```
IGMP / MLD
→ receiver membership

Multicast routing
→ forwarding multicast between routed networks
```

Protocols such as PIM can be used for multicast routing.

The details of PIM and multicast-routing design belong in a dedicated article.

---

## Anycast

Anycast allows the same IP address to be advertised from multiple servers or locations.

The routing system determines which instance receives the traffic.

Conceptually:

```
Same destination IP
        ↓
Advertised from multiple locations
        ↓
Routing chooses one path
        ↓
Client reaches one instance
```

For example, the same service address could be advertised from:

```
Warsaw
Frankfurt
London
```

A client in Poland may be routed toward Warsaw, while a client in the United Kingdom may be routed toward London.

Anycast is commonly used by globally distributed services such as DNS infrastructure and large content-delivery platforms.

---

## Anycast Does Not Mean Fastest Server

Anycast is often described as sending users to the "closest" or "fastest" server.

That can be a useful simplification, but it is not technically exact.

Routing selects a path according to routing information, metrics, and policy.

Therefore:

```
Anycast
→ routing chooses the preferred available path

not necessarily
→ application measures every server and chooses the fastest response
```

The result often reduces latency, but this is a consequence of routing design rather than an application-level speed test.

---

## Anycast vs Unicast

Anycast can look like normal unicast from the client's perspective.

The client sends traffic to one destination IP address.

The difference is that the same address exists in multiple locations.

Conceptually:

```
Unicast
→ one destination address
→ one specific endpoint

Anycast
→ one destination address
→ multiple possible endpoints
→ routing selects one
```

---

## Comparing the Traffic Types

```
Unicast
→ one sender
→ one specific receiver

Broadcast
→ one sender
→ everyone in the local broadcast domain

Multicast
→ one sender
→ selected group of receivers

Anycast
→ one destination address
→ multiple possible receivers
→ routing selects one
```

---

## Broadcast Storms

A broadcast storm occurs when excessive broadcast traffic consumes network resources.

One common cause is a Layer 2 switching loop.

For example:

```
Switch A
  |   \\
  |    \\
  |     Switch B
  |     /
  |    /
Switch C
```

If a broadcast frame enters a loop, switches may continue flooding it.

Ethernet frames do not have an IP-style TTL that is reduced by every switch.

Conceptually:

```
Broadcast frame
→ flooded
→ reaches another switch
→ flooded again
→ returns through the loop
→ flooded again
→ repeats
```

The resulting traffic can consume large amounts of bandwidth and processing capacity.

Possible effects include:

```
Link saturation
High switch CPU usage
MAC table instability
Packet loss
Very high latency
Loss of normal network communication
```

---

## Spanning Tree Protocol

Spanning Tree Protocol helps prevent Layer 2 loops.

STP examines redundant Layer 2 paths and creates a logical loop-free topology.

Conceptually:

```
Primary path
→ forwarding

Redundant path
→ non-forwarding
```

If the active path fails, a redundant path can become active.

The purpose is not simply to remove duplicate links.

STP controls which switch ports forward traffic so that the Ethernet topology does not contain an active Layer 2 loop.

Modern variants such as RSTP provide faster convergence, but those details belong in a dedicated switching article.

---

## Security and Operational Considerations

Different traffic types introduce different operational considerations.

Broadcast traffic can consume resources on every host in a broadcast domain.

Large broadcast domains can therefore increase unnecessary traffic and make failures such as broadcast storms more disruptive.

Multicast can be efficient, but it should be properly controlled using mechanisms such as:

```
IGMP Snooping
MLD Snooping
Multicast Routing
```

Unnecessary multicast flooding can waste bandwidth.

Anycast depends on routing behavior and should be monitored as part of the routing design.

For both IPv4 and IPv6, network administrators should understand which traffic types are permitted across:

- VLAN boundaries,
- routers,
- firewalls,
- VPNs,
- WAN links.

---

## Troubleshooting

Useful questions include:

```
Is the traffic unicast, broadcast, multicast, or anycast?

Is the sender and receiver in the same broadcast domain?

Is a router expected to forward this traffic?

Is the traffic IPv4 or IPv6?

Is the destination an IPv4 broadcast address?

Is the destination an IPv4 or IPv6 multicast group?

Is IGMP or MLD membership working?

Is IGMP or MLD snooping configured correctly?

Is multicast routing required?

Is a firewall blocking the traffic?

Is a switching loop creating excessive broadcasts?

Is STP functioning correctly?
```

Packet captures are particularly useful because they allow the administrator to inspect:

```
Source IP
Destination IP
Source MAC
Destination MAC
Protocol
```

and determine exactly how the traffic is being delivered.

---

## Key Takeaways

The four main traffic-delivery concepts are:

```
Unicast
Broadcast
Multicast
Anycast
```

Unicast means:

```
one sender
→ one specific receiver
```

Broadcast means:

```
one sender
→ all hosts in the broadcast domain
```

IPv4 supports broadcast.

IPv6 does not use broadcast and relies on multicast instead.

The Ethernet broadcast MAC address is:

```
FF:FF:FF:FF:FF:FF
```

IPv4 multicast MAC addresses begin with:

```
01:00:5E
```

IPv6 multicast MAC addresses begin with:

```
33:33
```

IGMP is used for IPv4 multicast group membership.

MLD is used for IPv6 multicast listener membership.

IGMP Snooping and MLD Snooping allow switches to learn which ports need multicast traffic.

Multicast does not normally cross routers without multicast-routing support.

Anycast allows multiple locations to advertise the same destination IP address, with routing selecting which instance receives the traffic.

Broadcast storms can make a switched network unusable, especially when caused by Layer 2 loops.

STP and related protocols help prevent Layer 2 loops by maintaining a loop-free forwarding topology.