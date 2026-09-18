# Ethernet and MAC Addressing

## Overview

Ethernet is one of the most common communication technologies used in modern local networks.

It is more than just a cable standard. Ethernet defines both physical communication methods and rules used to exchange data between devices on a local network.

It includes concepts such as:

* Ethernet frames,
* MAC addressing,
* switching,
* frame sizes,
* physical media,
* communication rules.

For many beginners, Ethernet is often associated only with network cables, but Ethernet is a much broader communication standard.

Understanding Ethernet is important because it forms the foundation of most wired local networks and explains how devices communicate at Layer 2.

## Ethernet as a Standard

Ethernet is a family of communication standards primarily used in wired LAN environments.

It defines how devices can physically connect and how data is formatted and transmitted between devices.

Ethernet can use different physical media, including:

* copper cabling,
* fiber-optic cabling,
* different connector types,
* different link speeds.

Although the physical implementation can change, the basic Layer 2 communication principles remain similar.

One of the most important parts of Ethernet communication is the Ethernet frame.

## MAC Addresses

A **MAC address** is a Layer 2 identifier associated with a network interface.

A traditional Ethernet MAC address contains 48 bits and is commonly represented using 12 hexadecimal characters.

For example:

```text
AA:BB:CC:DD:EE:FF
```

A MAC address identifies a network interface rather than the entire computer.

This means one system may have multiple MAC addresses.

For example, a workstation may have separate MAC addresses for:

* Ethernet,
* Wi-Fi,
* virtual network interfaces,
* virtualization adapters.

MAC addresses are mainly used for communication across the local Layer 2 network.

Switches use them to determine where Ethernet frames should be forwarded.

## MAC Address Structure

A traditional 48-bit MAC address can be divided into two conceptual parts.

For example:

```text
AA:BB:CC:DD:EE:FF
```

can be viewed as:

```text
AA:BB:CC : DD:EE:FF
   OUI    : Device-specific part
```

The first 24 bits are commonly associated with an **Organizationally Unique Identifier**, or OUI.

The OUI identifies the organization or manufacturer to which that address range was assigned.

The remaining part is assigned by the organization to individual network interfaces.

One manufacturer may own multiple OUIs.

This means different devices from the same vendor do not necessarily have the same first three bytes.

## MAC Addresses Are Not Absolute Identities

MAC addresses are often described as unique hardware identifiers, but they should not be treated as absolute identities.

MAC addresses can be:

* changed,
* spoofed,
* duplicated,
* virtualized,
* randomized.

Modern operating systems may also use randomized MAC addresses, especially when connecting to wireless networks.

This means the vendor information contained in the first part of the address may not always represent the actual hardware manufacturer.

MAC-based security controls should therefore not be treated as strong authentication by themselves.

## Ethernet Frames

At Layer 2, data is carried inside an **Ethernet frame**.

A frame is a Protocol Data Unit used by the Data Link layer.

Ethernet frames follow a standardized structure.

A simplified Ethernet II frame looks like:

```text
Preamble
SFD
Destination MAC
Source MAC
EtherType
Payload
FCS
```

Each field has a specific purpose.

## Preamble

The preamble helps the receiving device synchronize with the incoming transmission.

It prepares the receiver before the actual Ethernet frame begins.

## Start Frame Delimiter

The **Start Frame Delimiter**, or SFD, indicates where the Ethernet frame begins.

It separates synchronization information from the actual frame.

## Destination MAC Address

The destination MAC address identifies which Layer 2 endpoint should receive the frame.

Switches use this address to determine where the frame should be forwarded.

## Source MAC Address

The source MAC address identifies the Layer 2 interface that transmitted the frame.

Switches also use the source MAC address to learn where devices are located within the network.

## EtherType

The EtherType field identifies which protocol is encapsulated inside the Ethernet frame.

Examples include:

* IPv4,
* IPv6,
* ARP.

This allows the receiving system to understand how the payload should be interpreted.

## Payload

The payload contains the actual encapsulated higher-layer data.

In normal Ethernet communication, this is often an IP packet.

## Frame Check Sequence

The **Frame Check Sequence**, or FCS, is used to detect transmission errors.

It allows the receiving system to determine whether the frame was corrupted while being transmitted.

## Ethernet Frame Size

Standard Ethernet frames have defined size limits.

A normal Ethernet frame has a minimum size of:

```text
64 bytes
```

and a common maximum size of:

```text
1518 bytes
```

without additional tagging.

Frame sizes may change depending on additional technologies, such as VLAN tagging.

## MTU

**MTU**, or Maximum Transmission Unit, defines the maximum amount of Layer 3 data that can normally be carried inside a frame.

For standard Ethernet, the commonly used MTU is:

```text
1500 bytes
```

MTU does not describe the entire Ethernet frame.

Instead, it normally refers to the maximum size of the Layer 3 payload.

Conceptually:

```text
Ethernet Frame
├── Ethernet Header
├── IP Packet
└── FCS
```

where the IP packet is commonly limited to:

```text
1500 bytes
```

## Jumbo Frames

Frames larger than standard Ethernet frames are often referred to as **jumbo frames**.

A common jumbo-frame configuration uses an MTU around:

```text
9000 bytes
```

Jumbo frames can reduce protocol overhead when transferring large amounts of data.

However, every device along the communication path must support the configured size.

MTU mismatches can cause:

* dropped traffic,
* fragmentation problems,
* unstable application communication,
* difficult-to-diagnose connectivity issues.

MTU is normally configured on network interfaces rather than negotiated as part of normal Ethernet communication.

## Unicast

Unicast communication means one sender sends traffic to one specific destination.

Conceptually:

```text
Device A → Device B
```

The Ethernet frame contains the destination MAC address of the receiving device.

Unicast is one of the most common forms of Ethernet communication.

## Broadcast

Broadcast communication means one sender sends traffic to every device inside the same broadcast domain.

Ethernet uses the following destination MAC address for broadcast traffic:

```text
FF:FF:FF:FF:FF:FF
```

Switches flood broadcast frames to all relevant ports within the same VLAN.

Broadcast traffic does not normally cross routers.

## Multicast

Multicast communication is used when traffic should be delivered to a selected group of devices rather than one endpoint or every endpoint.

Conceptually:

```text
Sender
  |
  ├── Receiver A
  ├── Receiver B
  └── Receiver C
```

Multicast is useful when the same data must be delivered to multiple interested receivers.

Switches may use additional mechanisms such as IGMP snooping to handle multicast traffic more efficiently.

## MAC Address Types

Ethernet MAC addresses can represent different communication types.

### Unicast MAC

A unicast MAC address identifies one specific Layer 2 interface.

### Multicast MAC

A multicast MAC address identifies a group of receivers.

Ethernet uses part of the MAC address itself to indicate whether an address represents an individual or group destination.

### Broadcast MAC

Broadcast uses:

```text
FF:FF:FF:FF:FF:FF
```

This represents every device inside the local Layer 2 broadcast domain.

## MAC Address Tables

Switches use a **MAC address table** to determine which MAC addresses are reachable through which switch ports.

When a switch receives a frame, it examines the source MAC address.

For example:

```text
Frame arrives on Port 5

Source MAC:
AA:AA:AA:AA:AA:AA
```

The switch learns:

```text
AA:AA:AA:AA:AA:AA → Port 5
```

This information is stored in the MAC address table.

Later, when the switch receives a frame destined for that MAC address, it knows which port should be used.

## Unknown Unicast Flooding

If the destination MAC address does not exist in the MAC address table, the switch does not know where the device is located.

In this situation, the switch performs **unknown unicast flooding**.

The frame is forwarded through other relevant ports inside the same VLAN.

When the destination responds, the switch can learn its source MAC address and update the MAC table.

## Multiple MAC Addresses on One Port

A single switch port may have multiple MAC addresses associated with it.

For example, another switch may be connected to that port.

```text
Switch Port 10
      |
      |
Second Switch
  |   |   |
 PC  PC  PC
```

The first switch may learn several MAC addresses behind Port 10.

This is normal in environments containing:

* additional switches,
* wireless access points,
* hypervisors,
* virtualization hosts,
* other Layer 2 infrastructure.

## MAC Address Aging

Dynamic MAC table entries do not normally remain forever.

They are removed after a period of inactivity.

Entries may also disappear because of:

* interface disconnection,
* switch restart,
* topology changes,
* manual clearing.

Aging is important because devices can move between ports.

If a workstation was previously connected to Port 4 and later moves to Port 12, the switch must eventually update its table.

## Broadcast Domains

A **broadcast domain** is a group of devices that receive the same Layer 2 broadcast traffic.

In modern switched networks, one VLAN normally represents one broadcast domain.

For example:

```text
VLAN 10
→ Broadcast Domain 1

VLAN 20
→ Broadcast Domain 2
```

A broadcast frame inside VLAN 10 is not normally forwarded into VLAN 20.

Routers separate broadcast domains.

This prevents Layer 2 broadcasts from spreading across different networks.

## ARP and MAC Discovery

Ethernet requires a destination MAC address before it can deliver a frame.

A workstation may already know the destination IP address but not know the corresponding MAC address.

For IPv4, **ARP**, or Address Resolution Protocol, is used to determine which MAC address belongs to a specific IP address.

The process is:

```text
Destination IP known
        ↓
Check ARP cache
        ↓
MAC address unknown
        ↓
Send ARP Request
        ↓
Receive ARP Reply
        ↓
Store IP-to-MAC mapping
```

An ARP Request is normally sent as a broadcast because the sender does not yet know the destination MAC address.

The device owning the requested IP address responds with its MAC address.

## ARP and Remote Networks

If the destination is located outside the local subnet, the workstation does not attempt to discover the MAC address of the remote server.

Instead, it discovers the MAC address of the next-hop router, usually the default gateway.

For example:

```text
Workstation:
192.168.1.50

Default Gateway:
192.168.1.1

Remote Server:
203.0.113.10
```

The Layer 3 destination remains:

```text
203.0.113.10
```

but the Layer 2 destination MAC belongs to the default gateway.

Conceptually:

```text
Destination IP
→ Remote Server

Destination MAC
→ Default Gateway
```

This is one of the most important differences between Layer 2 and Layer 3 communication.

## MAC Addresses Between Router Hops

MAC addresses are relevant only to the current Layer 2 link.

When traffic reaches a router, the router removes the current Layer 2 frame.

It then examines the IP packet, determines the next hop, and creates a new Layer 2 frame.

For example:

```text
PC → Router A

Source MAC:
PC

Destination MAC:
Router A
```

Then:

```text
Router A → Router B

Source MAC:
Router A

Destination MAC:
Router B
```

The frame changes between network links.

The IP packet usually keeps the same source and destination addresses while it travels toward the destination.

There are exceptions, such as NAT, where IP addresses may also be modified.

A useful rule to remember is:

> MAC addresses are hop-local.

> IP addresses are logical end-to-end addresses.

## Ethernet and Switching

Ethernet and switching are closely related.

A switch examines Ethernet frames and uses MAC addresses to make forwarding decisions.

The basic logic is:

```text
Frame received
     ↓
Learn source MAC
     ↓
Check destination MAC
     ↓
Destination known?
     |
   Yes ----→ Forward to known port
     |
    No
     ↓
Unknown unicast flooding
```

This process allows switches to dynamically build an understanding of where devices exist on the local network.

## Ethernet Troubleshooting

Understanding Ethernet helps narrow down many Layer 2 problems.

Typical troubleshooting areas include:

* interface status,
* switch port configuration,
* MAC address tables,
* VLAN membership,
* ARP cache,
* MTU mismatch,
* frame errors,
* broadcast-domain issues.

For example, if a device has a valid IP configuration but cannot communicate with another device in the same VLAN, useful checks may include:

```text
Is the switch port active?

Is the correct MAC address being learned?

Is the device in the correct VLAN?

Does ARP resolve correctly?

Are frames reaching the expected interface?
```

## Key Takeaways

Ethernet is one of the main technologies used for local wired communication.

It defines more than physical cables. It also defines how Layer 2 data is formatted and exchanged.

MAC addresses identify network interfaces at Layer 2.

Ethernet frames carry data across the local network.

Switches learn which MAC addresses exist behind which ports and use this information to forward frames.

When the destination MAC is unknown, the switch performs unknown unicast flooding.

Broadcast frames are delivered throughout the local broadcast domain.

ARP connects Layer 3 IPv4 addressing with Layer 2 MAC addressing.

Most importantly:

```text
Ethernet
→ Local network communication

MAC address
→ Layer 2 interface identification

Frame
→ Layer 2 data unit

Switch
→ Learns MAC-to-port mappings

Router
→ Ends the current Layer 2 frame and creates a new one
```

The Layer 2 frame changes as traffic moves between routed networks.

The IP packet normally continues toward its final destination while each local link uses the MAC addresses required for that specific hop.

Understanding this relationship between Ethernet, MAC addresses, frames, switches, and routers is essential for troubleshooting local network communication.
