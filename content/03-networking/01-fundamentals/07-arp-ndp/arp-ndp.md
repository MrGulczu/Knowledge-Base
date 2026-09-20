# ARP and Neighbor Discovery Protocol

## Overview

Devices communicating over Ethernet need both Layer 3 and Layer 2 addressing.

IP addresses identify systems at the network layer, while MAC addresses are used to deliver Ethernet frames on the local network.

Before a device can send traffic to another device on the same local network, it needs to know which MAC address should be used as the Layer 2 destination.

IPv4 and IPv6 solve this problem differently:

```
IPv4
→ ARP

IPv6
→ Neighbor Discovery Protocol (NDP)
```

ARP is a relatively simple protocol used mainly to map IPv4 addresses to MAC addresses.

NDP provides similar neighbor-resolution functionality for IPv6, but it is broader and also supports router discovery, prefix discovery, Duplicate Address Detection, and neighbor reachability detection.

---

## ARP

ARP stands for:

```
Address Resolution Protocol
```

ARP is used in IPv4 networks to discover the MAC address associated with a specific IPv4 address on the local network.

For example:

```
Host A:
IP  192.168.1.10
MAC AA:AA:AA:AA:AA:AA

Host B:
IP  192.168.1.20
MAC BB:BB:BB:BB:BB:BB
```

Host A may know that it wants to communicate with:

```
192.168.1.20
```

but Ethernet requires a destination MAC address.

Host A therefore needs to discover which MAC address belongs to that IPv4 address.

---

## ARP Request

When a host does not know the MAC address associated with a local IPv4 address, it sends an ARP Request.

Conceptually:

```
Who has 192.168.1.20?
Tell 192.168.1.10
```

The ARP Request is sent as an Ethernet broadcast.

The destination MAC address is:

```
ff:ff:ff:ff:ff:ff
```

This means every device in the local broadcast domain receives the request.

Only the device that owns the requested IPv4 address should answer.

---

## ARP Reply

The target device responds with an ARP Reply.

Conceptually:

```
192.168.1.20 is at BB:BB:BB:BB:BB:BB
```

The requesting host can then associate:

```
192.168.1.20
```

with:

```
BB:BB:BB:BB:BB:BB
```

and use that MAC address as the destination of the Ethernet frame.

---

## Local Destination vs Remote Destination

ARP is only used to resolve addresses on the local Layer 2 network.

Before sending traffic, the host first determines whether the destination IP address belongs to the local subnet.

For example:

```
Host:
192.168.1.10/24

Destination:
192.168.1.20
```

Both addresses are inside:

```
192.168.1.0/24
```

The host therefore uses ARP to discover the MAC address of:

```
192.168.1.20
```

Conceptually:

```
Destination is local
        ↓
ARP for destination host
        ↓
Send Ethernet frame directly to destination MAC
```

---

## ARP and the Default Gateway

If the destination is outside the local subnet, the host does not ARP for the remote destination.

For example:

```
Host:
192.168.1.10/24

Default Gateway:
192.168.1.1

Remote Destination:
8.8.8.8
```

The host performs a route lookup and determines that the destination is outside its local subnet.

It therefore needs to send the packet to the next-hop router.

The host uses ARP to discover the MAC address of:

```
192.168.1.1
```

not the MAC address of:

```
8.8.8.8
```

Conceptually:

```
Destination is remote
        ↓
Route lookup
        ↓
Use default gateway / next hop
        ↓
ARP for next-hop IPv4 address
        ↓
Send Ethernet frame to router MAC
```

The IP packet still contains the final remote destination.

For example:

```
IP Packet

Source:
192.168.1.10

Destination:
8.8.8.8
```

while the Ethernet frame on the local LAN contains:

```
Ethernet Frame

Source MAC:
Host MAC

Destination MAC:
Default Gateway MAC
```

The router removes the incoming Layer 2 frame, examines the IP packet, performs another routing decision, and creates a new Layer 2 frame for the next hop.

This demonstrates an important rule:

```
IP destination
→ normally remains the final destination

Layer 2 destination
→ changes from hop to hop
```

---

## ARP Cache

A host does not need to send an ARP Request before every packet.

After learning an IPv4-to-MAC mapping, the system stores it temporarily in the ARP cache.

For example:

```
192.168.1.1
→ AA:BB:CC:DD:EE:FF
```

The host can reuse this information for later communication.

Conceptually:

```
ARP Request
    ↓
ARP Reply
    ↓
Store mapping
    ↓
Reuse cached mapping
```

ARP cache entries are not normally permanent.

Entries age out and may need to be resolved again later.

---

## Viewing the ARP or Neighbor Cache

On Linux, the neighbor table can be viewed using:

```
ip neigh
```

Older tools may also provide:

```
arp -n
```

On Windows, ARP information can be viewed using:

```
arp -a
```

Modern operating systems may expose both IPv4 ARP and IPv6 neighbor information through more general neighbor-table commands.

---

## ARP Security

ARP does not provide authentication of IPv4-to-MAC mappings.

A device may therefore accept false ARP information.

For example, the legitimate mapping may be:

```
192.168.1.1
→ AA:BB:CC:DD:EE:FF
```

An attacker may attempt to convince another host that:

```
192.168.1.1
→ 11:22:33:44:55:66
```

where:

```
11:22:33:44:55:66
```

belongs to the attacker.

If the victim accepts the false information, traffic intended for the gateway may be sent to the attacker instead.

Conceptually:

```
Victim
   |
   | traffic intended for gateway
   v
Attacker
   |
   v
Real Gateway
```

This is known as:

```
ARP Spoofing
```

or:

```
ARP Poisoning
```

Possible consequences include:

- man-in-the-middle traffic interception,
- traffic modification,
- denial of service,
- redirection of local traffic.

ARP security protections depend on the network environment and may include switch features, segmentation, monitoring, and static mappings in limited scenarios.

---

## Neighbor Discovery Protocol

## Overview

IPv6 does not use ARP.

Instead, IPv6 uses:

```
Neighbor Discovery Protocol
```

or:

```
NDP
```

NDP uses ICMPv6 messages.

It provides similar address-resolution functionality to ARP, but it also performs several other important IPv6 functions.

These include:

- neighbor address resolution,
- router discovery,
- prefix discovery,
- Duplicate Address Detection,
- Neighbor Unreachability Detection,
- redirect information.

Because NDP provides more functionality than ARP, it should not be described simply as "ARP for IPv6."

It is better understood as a broader IPv6 neighbor and router discovery mechanism.

---

## ARP and NDP Comparison

A simple comparison is:

```
IPv4
ARP Request
→ ARP Reply
```

and:

```
IPv6
Neighbor Solicitation
→ Neighbor Advertisement
```

However, NDP also includes other message types that ARP does not provide.

For example:

```
Router Solicitation
Router Advertisement
```

These are used for IPv6 router and network discovery.

---

## Neighbor Solicitation

A Neighbor Solicitation, or NS, is an ICMPv6 message used to request information about a neighbor.

One important use is discovering the Layer 2 address associated with an IPv6 address.

Conceptually:

```
Who is using this IPv6 address?
```

The receiving device can respond with a Neighbor Advertisement.

Unlike ARP Requests, Neighbor Solicitations do not rely on Ethernet broadcast.

IPv6 uses multicast instead.

---

## Neighbor Advertisement

A Neighbor Advertisement, or NA, is used to provide neighbor information.

Conceptually:

```
IPv6 address
→ Layer 2 address
```

The combination:

```
Neighbor Solicitation
→ Neighbor Advertisement
```

provides the IPv6 equivalent of the basic address-resolution function performed by:

```
ARP Request
→ ARP Reply
```

---

## Router Solicitation

A Router Solicitation, or RS, is sent by an IPv6 host when it wants local routers to provide Router Advertisement information.

A newly connected host may send an RS rather than waiting for the next periodic Router Advertisement.

Conceptually:

```
Host
   |
   | Router Solicitation
   v
Local IPv6 Routers
```

The host is effectively asking:

```
Are there IPv6 routers on this link?
Please provide network information.
```

---

## Router Advertisement

Routers send Router Advertisement, or RA, messages to hosts on the local link.

Router Advertisements can provide information such as:

- IPv6 prefix information,
- default-router information,
- prefix lifetime information,
- SLAAC-related configuration,
- other IPv6 network parameters.

Conceptually:

```
Router
   |
   | Router Advertisement
   v
Host
```

The host can use this information to learn about the IPv6 network.

Router Advertisements are sent periodically, but they can also be sent in response to a Router Solicitation.

---

## Default Router Discovery

IPv6 hosts normally learn their default router through Router Advertisements.

This differs from typical IPv4 networks, where DHCP commonly provides default-gateway information.

A router may advertise itself using its link-local address.

For example:

```
Default Router:
fe80::1
```

The host can then use that router as the next hop when sending traffic to remote IPv6 destinations.

---

## Duplicate Address Detection

Duplicate Address Detection, or DAD, is used to check whether an IPv6 address is already in use on the local link.

Before a newly configured IPv6 address becomes fully usable, it can be considered:

```
tentative
```

The host then checks whether another device is already using the same address.

Conceptually:

```
Generate IPv6 address
        ↓
Mark address tentative
        ↓
Send Neighbor Solicitation
        ↓
Check for conflict
        ↓
No conflict detected
        ↓
Address becomes usable
```

If another device already uses the address, a conflict can be detected and the new host should not begin normal use of that duplicated address.

DAD is used for automatically generated addresses and can also apply to manually configured IPv6 addresses.

---

## Neighbor Unreachability Detection

Neighbor Unreachability Detection, or NUD, allows an IPv6 host to determine whether a previously known neighbor is still reachable.

Knowing a neighbor's Layer 2 address does not guarantee that the neighbor is still operational.

For example, a router could:

- shut down,
- lose connectivity,
- move,
- experience an interface failure.

NUD helps detect this more quickly.

Conceptually:

```
Neighbor known
    ↓
Reachability stops being confirmed
    ↓
Host checks neighbor
    ↓
Neighbor responds
→ continue using neighbor

No response
→ neighbor considered unreachable
```

This can help a system react to network failures instead of continuing to send traffic indefinitely toward an unreachable next hop.

---

## Neighbor Cache

IPv6 hosts maintain a neighbor cache.

The neighbor cache contains information about devices on the local link.

It is similar in purpose to an IPv4 ARP cache, but NDP keeps more state than a simple IPv4-to-MAC mapping.

Conceptually:

```
IPv6 Address
→ Layer 2 Address
→ Reachability State
```

The exact neighbor-state machine is more detailed than required for basic networking knowledge.

The important point is that IPv6 systems track not only address mappings but also information about whether a neighbor appears reachable.

---

## Solicited-Node Multicast

IPv6 does not use broadcast for neighbor discovery.

Instead, NDP uses multicast.

One important multicast mechanism is:

```
Solicited-Node Multicast
```

The solicited-node multicast range is:

```
ff02::1:ff00:0/104
```

A solicited-node multicast address is derived from the last 24 bits of an IPv6 address.

For example:

```
Target IPv6:
2001:db8::1234:5678
```

The last 24 bits are:

```
34:5678
```

The corresponding solicited-node multicast address becomes:

```
ff02::1:ff34:5678
```

A Neighbor Solicitation can then be sent to this multicast group instead of being sent to every device on the LAN.

Conceptually:

```
IPv4 ARP

ARP Request
        ↓
Broadcast
        ↓
Every host in broadcast domain
```

Compared with:

```
IPv6 NDP

Neighbor Solicitation
        ↓
Solicited-Node Multicast
        ↓
Smaller multicast group
```

Multiple IPv6 addresses can map to the same solicited-node multicast group because only the final 24 bits are used.

The target IPv6 address inside the Neighbor Solicitation determines which device should respond.

---

## Why Solicited-Node Multicast Is Useful

Broadcast-style communication forces every host on the local network to receive and inspect a request.

Solicited-node multicast limits the request to a smaller group of interested hosts.

Conceptually:

```
Broadcast
→ every host receives the frame
→ every host must process it
```

while:

```
Solicited-node multicast
→ only hosts subscribed to that multicast group receive it
→ less unnecessary processing
```

This is one of the mechanisms IPv6 uses to avoid broadcast communication.

---

## NDP Security

NDP does not automatically guarantee that received discovery information is trustworthy.

An attacker on the local network may attempt to send malicious NDP messages.

One important example is a rogue Router Advertisement.

An attacker may send false information claiming to be an IPv6 router.

For example:

```
Legitimate Router:
fe80::1

Attacker:
fe80::bad
```

If a host accepts the attacker's Router Advertisement, it may learn incorrect routing or network information.

Potential consequences include:

- traffic interception,
- man-in-the-middle attacks,
- denial of service,
- incorrect prefix configuration,
- use of a rogue default router.

Conceptually:

```
Victim
   |
   | remote traffic
   v
Attacker acting as router
   |
   v
Real network
```

This is conceptually similar to ARP spoofing in that a host accepts false local-network information, but Router Advertisement attacks can affect more than a single IPv6-to-MAC mapping.

---

## RA Guard

Managed switches may provide protections such as:

```
RA Guard
```

RA Guard can help prevent unauthorized Router Advertisement messages from reaching hosts through ports where routers should not exist.

For example:

```
Access Port
→ normal workstation
→ Router Advertisements blocked
```

while:

```
Trusted Router Port
→ legitimate router
→ Router Advertisements allowed
```

RA Guard is one example of why IPv6 security should be considered explicitly when configuring switches and access networks.

---

## ARP vs NDP

ARP and NDP solve related problems, but they are not versions of the same protocol.

A better comparison is:

```
ARP
→ IPv4
→ separate protocol
→ mainly maps IPv4 addresses to MAC addresses
→ uses broadcast for requests
```

```
NDP
→ IPv6
→ uses ICMPv6
→ resolves neighbor Layer 2 addresses
→ discovers routers
→ advertises prefixes
→ performs Duplicate Address Detection
→ checks neighbor reachability
→ uses multicast instead of broadcast
```

ARP can therefore be seen as a simpler IPv4 address-resolution mechanism, while NDP provides a broader set of discovery and reachability functions required by IPv6.

---

## Practical Troubleshooting

When troubleshooting IPv4 ARP problems, useful questions include:

```
Is the destination inside the local subnet?

Is the correct default gateway configured?

Does the ARP cache contain the expected mapping?

Can the gateway be reached?

Does the MAC address in the ARP table belong to the expected device?

Is another host using the same IPv4 address?

Are VLANs configured correctly?

Is ARP traffic reaching the correct broadcast domain?
```

Useful commands include:

### Linux

```
ip neigh
ip route
ping <destination>
```

### Windows

```
arp -a
route print
ping <destination>
```

For IPv6 NDP troubleshooting, useful questions include:

```
Does the interface have a link-local address?

Does the neighbor cache contain the expected entry?

Are Router Advertisements being received?

Is the correct default router present?

Did Duplicate Address Detection succeed?

Is ICMPv6 being blocked?

Is the neighbor marked reachable?

Are Neighbor Solicitation and Neighbor Advertisement messages flowing?

Are rogue Router Advertisements present?
```

Useful commands include:

### Linux

```
ip -6 neigh
ip -6 addr
ip -6 route
ping -6 <destination>
```

### Windows

```
netsh interface ipv6 show neighbors
ipconfig
route print -6
ping -6 <destination>
```

---

## Security Considerations

ARP and NDP should both be treated as security-relevant local-network protocols.

Important risks include:

```
ARP
→ spoofed IPv4-to-MAC mappings
→ ARP poisoning
→ man-in-the-middle attacks
→ denial of service
```

```
NDP
→ fake Neighbor Advertisements
→ rogue Router Advertisements
→ incorrect routing information
→ traffic interception
→ denial of service
```

Network security controls should cover both IPv4 and IPv6.

An environment that protects ARP and IPv4 traffic but ignores NDP and IPv6 may still contain exploitable local-network paths.

---

## Key Takeaways

ARP is used by IPv4 hosts to discover which MAC address belongs to a local IPv4 address.

ARP Requests are normally broadcast to:

```
ff:ff:ff:ff:ff:ff
```

The device owning the requested IPv4 address responds with an ARP Reply.

If the destination is outside the local subnet, the host does not ARP for the remote Internet host.

Instead, it resolves the Layer 2 address of the next-hop router or default gateway.

ARP mappings are stored temporarily in an ARP cache.

ARP does not provide strong authentication, making ARP spoofing and ARP poisoning possible.

IPv6 does not use ARP.

Instead, IPv6 uses Neighbor Discovery Protocol over ICMPv6.

The most important NDP messages covered here are:

```
Neighbor Solicitation
→ request neighbor information

Neighbor Advertisement
→ provide neighbor information

Router Solicitation
→ request router information

Router Advertisement
→ advertise router and prefix information
```

NDP also provides:

```
Duplicate Address Detection
Neighbor Unreachability Detection
Router Discovery
Prefix Discovery
```

IPv6 uses solicited-node multicast instead of ARP-style broadcast for neighbor discovery.

This reduces unnecessary processing by limiting requests to smaller multicast groups.

ARP is therefore a simpler IPv4 address-resolution mechanism, while NDP is a broader IPv6 discovery and reachability framework.