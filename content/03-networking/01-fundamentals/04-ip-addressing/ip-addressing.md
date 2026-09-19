# IP Addressing

## Overview

IP addressing is one of the foundations of Layer 3 communication.

While MAC addresses are mainly used for communication across the local Layer 2 link, IP addresses provide logical addressing that allows devices to communicate across different networks.

IP addressing also allows us to logically divide infrastructure into separate networks and subnets.

The two IP versions commonly used today are:

* IPv4,
* IPv6.

IPv4 uses 32-bit addresses, while IPv6 uses 128-bit addresses.

Understanding IP addressing is important because routing, subnetting, DHCP, NAT, firewalls, VPNs, and many other networking technologies depend on it.

---

## IPv4 Address Structure

An IPv4 address consists of 32 bits divided into four 8-bit sections called octets.

For example:

```text
192.168.10.25
```

Each octet can contain a value from:

```text
0 - 255
```

In binary, an IPv4 address can be represented as:

```text
11000000.10101000.00001010.00011001
```

An IPv4 address contains two logical parts:

* network portion,
* host portion.

The subnet mask or CIDR prefix determines where the network portion ends and the host portion begins.

For example:

```text
192.168.10.25/24
```

The `/24` means that the first 24 bits belong to the network portion.

The remaining 8 bits identify hosts inside that network.

Conceptually:

```text
192.168.10.25/24

Network:
192.168.10.0/24

Host portion:
25
```

---

## CIDR Prefixes and Subnet Masks

CIDR notation tells us how many bits of the IP address belong to the network.

For example:

```text
/24
→ 24 network bits
→ 8 host bits
```

```text
/16
→ 16 network bits
→ 16 host bits
```

```text
/8
→ 8 network bits
→ 24 host bits
```

The same information can also be written using a subnet mask.

Common examples include:

```text
/24 → 255.255.255.0

/16 → 255.255.0.0

/8  → 255.0.0.0
```

The host uses this information to determine whether a destination belongs to the local network or whether the traffic must be sent to a router.

---

## Local and Remote Networks

Consider these two addresses:

```text
192.168.10.25/24
192.168.20.25/24
```

The first belongs to:

```text
192.168.10.0/24
```

The second belongs to:

```text
192.168.20.0/24
```

Even though both addresses end in `.25`, they belong to different networks.

Communication between them therefore normally requires a Layer 3 device such as:

* router,
* Layer 3 switch,
* firewall performing routing.

A host decides whether a destination is local by comparing the destination address with its own address and subnet mask.

If the destination is local, the host sends traffic directly.

If the destination is remote, traffic is sent toward the default gateway.

---

## Network and Broadcast Addresses

Every traditional IPv4 subnet contains an address representing the network itself.

For example:

```text
192.168.10.0/24
```

The address:

```text
192.168.10.0
```

represents the network.

It is not normally assigned to an individual host.

For a standard `/24` network:

```text
192.168.10.0
→ Network address

192.168.10.1 - 192.168.10.254
→ Host addresses

192.168.10.255
→ Broadcast address
```

The broadcast address is the final address in the subnet and is used to address all hosts in that broadcast domain.

There are exceptions to these traditional rules, such as `/31` point-to-point links, which are covered in the subnetting topic.

---

## Default Gateway

The default gateway is the next-hop IP address used when a destination is outside the local subnet.

For example:

```text
Host:
192.168.10.25/24

Default Gateway:
192.168.10.1

Destination:
8.8.8.8
```

The host determines that `8.8.8.8` is not part of:

```text
192.168.10.0/24
```

and therefore sends the traffic toward:

```text
192.168.10.1
```

At Layer 2, the Ethernet frame is addressed to the MAC address of the gateway.

At Layer 3, the destination IP remains the original remote destination.

A simple rule is:

```text
Local destination
→ Send directly

Remote destination
→ Use the matching route

If no specific route exists
→ Use the default gateway
```

---

## Public and Private IPv4 Addresses

IPv4 addresses can broadly be divided into public and private address space.

### Public IPv4 Addresses

Public IPv4 addresses are globally routable on the Internet.

They must be uniquely assigned to avoid conflicts between networks.

### Private IPv4 Addresses

Private IPv4 addresses are intended for internal networks and are not globally routed across the public Internet.

The private IPv4 ranges are:

```text
10.0.0.0/8
```

```text
172.16.0.0/12
```

```text
192.168.0.0/16
```

These ranges can be reused by different organizations because they are not globally routed.

Private addressing makes internal network design easier and helps reduce consumption of public IPv4 addresses.

---

## Choosing Private Address Space

All three private ranges are valid for internal networks.

However, the choice of range can have practical consequences.

Large corporate environments commonly use:

```text
10.0.0.0/8
```

or:

```text
172.16.0.0/12
```

because they provide large amounts of address space and reduce the risk of overlapping with common home networks.

Home routers commonly use networks such as:

```text
192.168.0.0/24
```

or:

```text
192.168.1.0/24
```

This can become important when remote users connect to a corporate environment through VPN.

For example:

```text
Home Network:
192.168.1.0/24

Corporate Network:
192.168.1.0/24
```

Now the user's workstation has two possible paths for the same network.

This may create routing conflicts and make corporate resources unreachable through the VPN.

For this reason, using less common address ranges in corporate environments can reduce the chance of overlapping with home networks.

This is a design preference rather than a technical requirement.

---

## Special IPv4 Addresses

IPv4 contains several special-use address ranges.

### Loopback

The loopback range is:

```text
127.0.0.0/8
```

The most commonly used address is:

```text
127.0.0.1
```

This is usually referred to as:

```text
localhost
```

Loopback communication stays inside the local host.

It can be used to test the local TCP/IP stack or expose services only to applications running on the same machine.

For example:

```text
127.0.0.1:8080
```

may represent a service that is available only locally.

A command such as:

```bash
ping 127.0.0.1
```

tests the local TCP/IP stack.

It does not test the physical network.

### Link-Local / APIPA

IPv4 link-local addressing uses:

```text
169.254.0.0/16
```

On systems such as Windows, an address from this range may be automatically assigned when the interface is configured for automatic addressing but cannot obtain an address from DHCP.

For example:

```text
169.254.42.17
```

may indicate a problem with:

* DHCP,
* VLAN configuration,
* switching,
* network connectivity.

The address can still be used for communication on the local link, but it is not normally routed to other networks.

### Broadcast

The broadcast address is normally the final address in an IPv4 subnet.

For:

```text
192.168.10.0/24
```

the broadcast address is:

```text
192.168.10.255
```

Traffic addressed to it is intended for all hosts in the subnet.

---

## Static and Dynamic IP Assignment

IP addresses can be configured manually or assigned automatically.

### Static IP Address

A static address is manually configured on a device.

It remains unchanged unless someone modifies the configuration.

Static addresses are commonly used for infrastructure where predictable addressing is important.

Examples include:

* servers,
* switches,
* routers,
* firewalls,
* printers,
* network appliances.

### DHCP

DHCP allows addresses to be assigned automatically.

A DHCP server can provide clients with information such as:

* IP address,
* subnet mask,
* default gateway,
* DNS servers.

Addresses are assigned from configured pools.

This removes the need to configure each endpoint individually.

DHCP can also support:

* exclusions,
* reservations,
* fixed assignments based on MAC address or client identifier.

A reservation allows a device to receive its configuration automatically while still consistently receiving the same address.

A simple comparison is:

```text
Static IP
→ Manually configured
→ Predictable
→ Common for infrastructure

DHCP
→ Automatically assigned
→ Easy to manage
→ Common for endpoints

DHCP Reservation
→ Automatically configured
→ Predictable address
```

---

## NAT and Private Addressing

Private IPv4 addresses cannot be routed directly across the public Internet.

NAT, or Network Address Translation, can translate internal addresses into public addresses.

For example:

```text
192.168.10.25
192.168.10.26
192.168.10.27
        ↓
       NAT
        ↓
203.0.113.5
```

From the Internet side, multiple internal devices may appear to use one public IP address.

NAT is especially common because IPv4 public address space is limited.

It is important to understand that NAT itself is not a replacement for a firewall.

NAT changes addressing.

Firewall policies determine whether communication is allowed.

---

## Common NAT Types

NAT is not one single mechanism.

There are several common forms.

### Static NAT

Static NAT creates a fixed one-to-one mapping between internal and external addresses.

Conceptually:

```text
192.168.10.10
↔
203.0.113.10
```

### Dynamic NAT

Dynamic NAT translates internal addresses using a pool of available public addresses.

Different internal hosts may receive different public addresses from that pool.

### PAT / NAT Overload

PAT allows many internal hosts to share one public address by translating port numbers in addition to IP addresses.

For example:

```text
192.168.10.25:51001
→
203.0.113.5:60001
```

```text
192.168.10.26:51001
→
203.0.113.5:60002
```

This is one of the most common forms of NAT used for normal Internet access.

### Destination NAT / Port Forwarding

Destination NAT changes the destination address of inbound traffic.

For example:

```text
203.0.113.5:443
        ↓
192.168.10.50:443
```

This can be used to expose an internal service through a public address.

Different vendors may use slightly different terminology, but the underlying concepts remain similar.

NAT is covered in more detail in the dedicated NAT topic.

---

## Introduction to IPv6

IPv6 was created with a much larger address space than IPv4.

The most obvious difference is address length.

IPv4 uses:

```text
32 bits
```

IPv6 uses:

```text
128 bits
```

IPv4 is normally written in decimal:

```text
192.168.10.25
```

IPv6 uses hexadecimal notation:

```text
2001:db8:1234:5678::25
```

The much larger address space allows IPv6 to provide an enormous number of possible addresses.

---

## IPv6 Addressing Differences

IPv6 uses different address concepts from IPv4.

Common IPv6 address types include:

* Global Unicast,
* Unique Local,
* Link-Local.

IPv6 does not use IPv4 RFC1918 private addressing in the same way.

It also does not use broadcast.

IPv6 uses multicast instead.

IPv6 interfaces commonly have a Link-Local address beginning with:

```text
fe80::/10
```

even when the same interface also has a globally routable address.

---

## IPv6 Subnetting

IPv6 still uses subnetting, but network design is normally approached differently from IPv4.

A common LAN prefix is:

```text
/64
```

This means:

```text
64 bits
→ Network prefix

64 bits
→ Interface portion
```

IPv4 networks frequently use many different subnet sizes depending on the number of hosts required.

IPv6 generally has enough address space that conserving addresses is much less important.

The detailed structure of IPv6 addressing and subnetting is covered in the dedicated IPv6 topic.

---

## IPv4 and IPv6 Comparison

A simplified comparison looks like:

```text
IPv4
→ 32-bit addressing
→ Decimal notation
→ Public and private address ranges
→ Broadcast supported
→ NAT commonly used
```

```text
IPv6
→ 128-bit addressing
→ Hexadecimal notation
→ Global Unicast, ULA, Link-Local
→ No broadcast
→ /64 commonly used for LANs
→ Very large address space
```

IPv4 and IPv6 can exist together in the same environment.

Many networks currently operate in dual-stack mode, supporting both protocols.

---

## IP Addressing and Troubleshooting

Understanding IP addressing helps quickly identify many network problems.

Useful checks include:

```text
Does the host have an IP address?

Is the subnet mask or prefix correct?

Is the default gateway correct?

Is the destination local or remote?

Does the routing table contain the expected route?

Is the host using the expected DNS configuration?

Did DHCP assign the correct network information?

Is there an address overlap with another network?

Is NAT being applied where expected?
```

For example, seeing:

```text
169.254.x.x
```

on a workstation that should receive an address from DHCP immediately narrows the troubleshooting area.

Similarly, if two VPN networks use the same subnet, routing behavior may explain why remote resources are unreachable.

---

## Key Takeaways

IP addresses are Layer 3 logical addresses.

They allow communication beyond the local Layer 2 network and provide the foundation for routing and network segmentation.

The subnet prefix determines which part of an address identifies the network and which part identifies the host.

A host uses this information to determine whether a destination is:

```text
Local
→ Send directly
```

or:

```text
Remote destination
→ Use the matching route

If no specific route exists
→ Use the default gateway
```

IPv4 includes public and private address ranges.

Private addresses are designed for internal networks and are commonly translated through NAT when accessing the public Internet.

Address planning matters.

Poorly chosen internal ranges can create problems such as overlapping networks, especially with VPN connections.

IPv4 and IPv6 use different addressing models, but both provide logical addressing used for Layer 3 communication.

The most important skill is not memorizing individual ranges.

It is understanding:

> Which network does this address belong to?

> Is the destination local or remote?

> Which gateway or route will be used?

> Is the addressing configuration consistent with the intended network design?

Once these questions can be answered, troubleshooting Layer 3 connectivity becomes much faster.
