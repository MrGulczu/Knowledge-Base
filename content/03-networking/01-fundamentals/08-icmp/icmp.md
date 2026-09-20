# ICMP

## Overview

ICMP stands for:

```
Internet Control Message Protocol
```

ICMP is a network-layer protocol used by IP devices for control messages, error reporting, and diagnostics.

It is commonly associated with tools such as:

```
ping
traceroute
```

but ICMP is not used only for troubleshooting.

IPv4 uses ICMP, while IPv6 uses ICMPv6.

---

## ICMP Is Not TCP or UDP

ICMP is a separate protocol at the IP layer.

It does not use TCP or UDP ports.

```
TCP
→ uses ports

UDP
→ uses ports

ICMP
→ does not use ports
```

Instead, ICMP messages are identified using:

```
Type
Code
```

The Type identifies the general message category, while the Code provides more detailed information.

This is important when configuring firewalls because allowing or blocking ICMP is different from allowing or blocking a TCP or UDP port.

---

## Ping

The `ping` command normally uses:

```
ICMP Echo Request
```

and:

```
ICMP Echo Reply
```

The sender transmits an Echo Request.

If the destination is reachable and configured to respond, it sends back an Echo Reply.

```
Client
   |
   | Echo Request
   v
Destination
   |
   | Echo Reply
   v
Client
```

Ping can test basic IP reachability and measure round-trip time.

A failed ping does not automatically mean that the destination is offline.

For example, if:

```
ping fails
```

but:

```
HTTPS works
```

the destination is reachable, but ICMP Echo traffic may be filtered or ignored.

Possible reasons include:

- host firewall rules,
- network firewall rules,
- upstream filtering,
- security policy,
- a host configured not to answer Echo Requests.

The website application itself does not normally control ICMP because ICMP operates below the application layer.

---

## TTL and Hop Limit

IPv4 uses:

```
TTL
```

IPv6 uses:

```
Hop Limit
```

Each router that forwards a packet decreases the value.

If a router receives an IPv4 packet with:

```
TTL = 1
```

it decreases it to:

```
TTL = 0
```

The router then discards the packet instead of forwarding it and normally returns:

```
ICMP Time Exceeded
```

to the sender.

```
Packet arrives
TTL = 1
    ↓
Router decrements TTL
    ↓
TTL = 0
    ↓
Packet dropped
    ↓
ICMP Time Exceeded
```

This prevents packets from circulating forever during routing loops.

---

## Traceroute

Traceroute uses the TTL or Hop Limit mechanism to discover the path toward a destination.

It sends probes with increasing values:

```
TTL = 1
→ first router

TTL = 2
→ second router

TTL = 3
→ third router

...
```

Each router where the TTL reaches zero normally returns an ICMP Time Exceeded message.

This allows traceroute to identify the path hop by hop.

Depending on the operating system and implementation, traceroute probes may use UDP, ICMP, or TCP.

The important mechanism is the ICMP Time Exceeded response.

---

## Destination Unreachable

ICMP Destination Unreachable reports that a packet could not be delivered.

The exact reason is represented by the ICMP Code.

Possible reasons include:

- no route to a destination network,
- a specific host cannot be reached,
- a UDP destination port is closed,
- traffic is administratively prohibited,
- an IPv4 packet requires fragmentation but fragmentation is not allowed.

A router may receive a packet for a network that it cannot reach and return a Destination Unreachable message to the sender.

A firewall may also silently drop traffic, so an ICMP error is not guaranteed in every failure scenario.

---

## Port Unreachable

A common example occurs with UDP.

Suppose a client sends traffic to:

```
UDP/9999
```

on a reachable server.

If nothing is listening on that port, the server may return:

```
ICMP Destination Unreachable
Code: Port Unreachable
```

```
Client
   |
   | UDP packet to port 9999
   v
Server
   |
   | no application listening
   v
ICMP Destination Unreachable
Port Unreachable
```

Because UDP does not establish a connection like TCP, ICMP is often how the sender learns that the destination UDP port is closed.

---

## ICMP Redirect

ICMP Redirect can be used by a router to tell a host that a better next hop exists for a particular destination.

For example:

```
Host
   |
   v
Router A
   |
   v
Router B
   |
   v
Destination
```

Router A may tell the host that Router B should be used directly.

The path can then become:

```
Host
   |
   v
Router B
   |
   v
Destination
```

ICMP Redirects are often treated cautiously because incorrect or malicious redirect information could influence a host's routing decisions.

Many modern networks restrict or disable them unless they are specifically needed.

---

## ICMP and Path MTU Discovery

Different network links can support different maximum packet sizes.

This is called:

```
MTU
```

or:

```
Maximum Transmission Unit
```

ICMP helps hosts discover when packets are too large for part of the network path.

This is part of:

```
Path MTU Discovery
```

---

## IPv4 Fragmentation Needed

In IPv4, a router may encounter a packet that is too large for the next link.

If fragmentation is not allowed, the router can discard the packet and return an ICMP Destination Unreachable message indicating that fragmentation is required.

The sender can then reduce the packet size.

---

## ICMPv6 Packet Too Big

IPv6 routers do not fragment packets while forwarding them.

If an IPv6 packet is too large for the next link, the router drops it and returns:

```
ICMPv6 Packet Too Big
```

```
IPv6 Sender
    |
    | packet too large
    v
Router
    |
    X
    |
    └── ICMPv6 Packet Too Big
            |
            v
        Sender reduces size
```

This makes ICMPv6 Packet Too Big messages important for reliable IPv6 communication.

---

## Path MTU Discovery Black Holes

If required ICMP messages are blocked, Path MTU Discovery can fail.

Symptoms may include:

```
Small packets work
Large packets fail
```

Possible effects include:

- websites partially loading,
- connections hanging,
- VPN problems,
- large transfers failing while small packets succeed.

This situation is sometimes called a PMTUD black hole.

---

## ICMPv6

ICMPv6 performs diagnostics and error reporting for IPv6, but it also supports several core IPv6 functions.

These include:

- Echo Request and Echo Reply,
- Destination Unreachable,
- Time Exceeded,
- Packet Too Big,
- Neighbor Discovery,
- Router Solicitation,
- Router Advertisement,
- Neighbor Solicitation,
- Neighbor Advertisement,
- Duplicate Address Detection,
- Path MTU Discovery.

ICMPv6 should therefore not be treated as an optional diagnostic protocol.

---

## ICMPv6 and Neighbor Discovery

Neighbor Discovery Protocol uses ICMPv6.

Important messages include:

```
Router Solicitation
Router Advertisement
Neighbor Solicitation
Neighbor Advertisement
```

These support functions such as:

- router discovery,
- prefix discovery,
- address resolution,
- Duplicate Address Detection,
- Neighbor Unreachability Detection.

Blocking all ICMPv6 can therefore break basic IPv6 networking.

---

## ICMP and Firewalls

Blocking all ICMP can cause more than troubleshooting problems.

It can interfere with:

```
Ping
Traceroute
Destination Unreachable reporting
Path MTU Discovery
```

Blocking ICMPv6 can have even greater impact because IPv6 depends on ICMPv6 for several core protocol functions.

A better approach is:

```
Understand the ICMP message types
        ↓
Allow required traffic
        ↓
Filter unnecessary traffic appropriately
```

rather than:

```
Block all ICMP
```

---

## Security Considerations

ICMP can reveal information about network reachability and topology, so some environments restrict specific ICMP message types.

However, filtering should be performed carefully.

Important points include:

- a failed ping does not prove that a host is offline,
- blocking Echo Requests does not automatically make a host secure,
- required ICMP messages are important for Path MTU Discovery,
- required ICMPv6 messages are necessary for normal IPv6 operation,
- ICMP Redirects should only be accepted where they are expected,
- firewalls should distinguish useful control traffic from unnecessary exposure.

---

## Practical Troubleshooting

Useful questions include:

```
Can the destination be pinged?

If ping fails, does another protocol still work?

Does traceroute reach the destination?

Where does traceroute stop?

Are ICMP Time Exceeded messages returning?

Is the destination returning Destination Unreachable?

Are UDP Port Unreachable messages being received?

Are ICMP messages blocked by a firewall?

Do small packets work while larger packets fail?

Is Path MTU Discovery working?

For IPv6, are required ICMPv6 messages allowed?
```

---

## Useful Commands

### Linux

Ping IPv4:

```
ping 8.8.8.8
```

Ping IPv6:

```
ping -6 <destination>
```

Traceroute:

```
traceroute <destination>
```

Traceroute IPv6:

```
traceroute -6 <destination>
```

Capture ICMP:

```
tcpdump icmp
```

Capture ICMPv6:

```
tcpdump icmp6
```

### Windows

Ping:

```
ping <destination>
```

Force IPv4:

```
ping -4 <destination>
```

Force IPv6:

```
ping -6 <destination>
```

Traceroute:

```
tracert <destination>
```

Traceroute IPv6:

```
tracert -6 <destination>
```

---

## ICMP vs ICMPv6

A simplified comparison:

```
ICMP
→ IPv4
→ diagnostics
→ error reporting
→ Echo Request / Reply
→ Time Exceeded
→ Destination Unreachable
→ Redirect
→ supports IPv4 Path MTU Discovery
```

```
ICMPv6
→ IPv6
→ diagnostics
→ error reporting
→ Echo Request / Reply
→ Time Exceeded
→ Destination Unreachable
→ Packet Too Big
→ Neighbor Discovery
→ Router Discovery
→ Duplicate Address Detection
→ Path MTU Discovery
```

---

## Key Takeaways

ICMP is a separate network-layer protocol used for control, error reporting, and diagnostics.

It does not use TCP or UDP ports.

ICMP messages use:

```
Type
Code
```

Ping uses:

```
Echo Request
Echo Reply
```

Traceroute relies on:

```
Time Exceeded
```

messages generated when TTL or Hop Limit reaches zero.

Destination Unreachable reports that a packet could not be delivered.

A common example is:

```
Port Unreachable
```

when a UDP packet reaches a host but no application is listening on the destination port.

ICMP Redirect can inform a host about a more appropriate next hop.

ICMP is also important for Path MTU Discovery.

In IPv6, routers do not fragment transit packets.

Instead, they return:

```
ICMPv6 Packet Too Big
```

so the sender can reduce the packet size.

Blocking all ICMP can therefore break legitimate network functions.

This is especially important for IPv6 because ICMPv6 supports several core mechanisms, including Neighbor Discovery and Router Advertisement.

A failed ping should not automatically be interpreted as a failed network connection.

If an HTTPS service works while ping fails, the destination is reachable but ICMP Echo traffic is likely being filtered or ignored.