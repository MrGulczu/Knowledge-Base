# Network Types and Topologies

## Overview

Networks can be described in different ways depending on their size, purpose, and structure.

One way is to describe the **area in which the network operates**. This gives us terms such as PAN, LAN, WLAN, MAN, and WAN.

Another way is to describe the **topology of the network**, meaning how devices are connected and how communication is organized between them.

When discussing topology, it is especially important to distinguish between:

- physical topology
- logical topology

A network can have one physical structure while being divided into multiple logical networks.

Understanding this distinction helps when designing infrastructure, identifying single points of failure, planning redundancy, and troubleshooting connectivity.

---

## Network Types

Network types are commonly categorized based on their geographic scope and purpose.

The borders between these categories are not always strict. The terms are mainly useful for describing the general scale and purpose of the network.

### PAN - Personal Area Network

A **Personal Area Network** is a very small network built around one person and their nearby devices.

PANs are extremely common even though many users may not realize they are using one.

Examples include communication between smartphones, wireless headphones, smartwatches, keyboards, mice, and other personal devices.

PANs commonly use short-range wireless technologies such as Bluetooth.

### LAN - Local Area Network

A **Local Area Network** connects devices within a relatively limited geographic area.

Examples include:

- a home
- an office
- a building
- a small campus

LAN infrastructure is commonly controlled by one person or organization.

LANs often use private IPv4 addressing, but private addressing does not define whether a network is a LAN. A LAN can technically use public addresses, and private addresses can also be used across larger networks.

The important characteristic is that the network operates within a relatively local environment.

### WLAN - Wireless Local Area Network

A **Wireless Local Area Network** provides local network connectivity using wireless communication instead of a physical cable to the endpoint.

WLAN and LAN have many things in common. The main difference is the access medium.

A traditional wired LAN commonly uses Ethernet cables, switches, and physical switch ports.

A WLAN commonly uses radio waves, wireless access points, SSIDs, and wireless channels.

A typical environment contains both wired LAN and WLAN infrastructure.

```text
Laptop
   |
 Wi-Fi
   |
Access Point
   |
Ethernet
   |
Switch
```

The laptop is connected through WLAN, while the access point itself may be connected to the wired LAN.

### MAN - Metropolitan Area Network

A **Metropolitan Area Network** is larger than a typical LAN but smaller in geographic scope than a traditional WAN.

It commonly connects networks across a city or metropolitan area.

For example, an organization may have several offices located across one city and use a metropolitan network to connect them.

In real environments, the distinction between MAN and WAN is not always strict. Networks connecting separate locations are often simply described as WANs.

### WAN - Wide Area Network

A **Wide Area Network** connects networks over larger geographic distances.

A WAN may span cities, regions, countries, or continents.

WANs are commonly used to connect separate LANs and may rely on infrastructure provided by telecommunications operators or Internet service providers.

Examples may include:

- leased lines
- MPLS
- VPN connections
- Internet connectivity
- carrier networks

WAN does not mean that public IP addresses must be used. A WAN may use either public or private addressing depending on its design.

### IPv4 and IPv6 Addressing

IPv4 and IPv6 approach addressing differently.

IPv4 commonly uses private address ranges inside internal networks and public addresses for Internet-routable communication.

IPv6 does not use RFC1918 private addressing in the same way. It includes address types such as Global Unicast, Unique Local, and Link-Local addresses.

Subnetting is also approached differently in IPv6. These subjects belong in the dedicated IP addressing and IPv6 topics.

---

## Physical and Logical Topology

The most important distinction when discussing network topology is the difference between **physical topology** and **logical topology**.

### Physical Topology

Physical topology describes how network devices are actually connected.

It shows things such as:

- which device connects to which
- physical network links
- switches and routers
- access switches
- distribution switches
- core infrastructure
- redundant connections
- single points of failure

Looking at the physical topology allows an administrator to answer questions such as:

- Is there only one path between devices?
- Is the network redundant?
- What happens if this switch fails?
- Which devices depend on this connection?
- Where are the single points of failure?

### Logical Topology

Logical topology describes how communication is organized across the physical network.

It may include:

- IP addressing
- subnets
- VLANs
- routing relationships
- broadcast domains
- network segmentation
- tunnels
- logical overlays

Two computers may be physically connected to the same switch but still belong to different VLANs and different IP subnets. Physically they are close together, but logically they may require Layer 3 routing to communicate.

A useful way to remember the difference is:

> Physical topology describes how devices are connected.

> Logical topology describes how communication is organized across those connections.

---

## Common Physical Topologies

### Star Topology

In a **star topology**, multiple devices connect to one central device.

In modern Ethernet networks, this central device is usually a switch.

```text
        PC
         |
PC --- Switch --- Server
         |
       Printer
```

Each endpoint has its own connection to the central switch.

Advantages include easy expansion, easier troubleshooting, and the fact that failure of one endpoint cable usually affects only that endpoint.

The main disadvantage is that the central device can become a single point of failure.

#### Extended Star

Large networks commonly use an **extended star topology**.

Instead of all devices connecting to one switch, multiple switches are connected together in a hierarchy.

```text
              Core
             /    \
            /      \
Distribution     Distribution
     |                |
Access Switches   Access Switches
     |                |
Endpoints          Endpoints
```

This allows the network to scale and introduces additional functionality such as segmentation, redundancy, routing, and security controls.

### Bus Topology

In a **bus topology**, all devices share one common communication medium.

Traffic transmitted onto the shared medium can be seen by connected devices, while only the intended recipient should process the communication addressed to it.

Bus networks were used by older Ethernet technologies and have several disadvantages, including shared bandwidth, difficult troubleshooting, poor scalability, and the possibility that a failure in the main communication path affects many devices.

Classic bus Ethernet networks are rarely used in modern computer networks, but bus-style communication is still heavily used in other environments, particularly automotive and industrial systems. A well-known example is CAN bus.

### Ring Topology

In a **ring topology**, each device is connected to two neighboring devices, creating a closed loop.

```text
Device A ----- Device B
   |              |
   |              |
Device D ----- Device C
```

Traffic travels through the ring until it reaches its destination.

Some ring technologies, such as Token Ring, use a token that determines which device is currently allowed to transmit.

In a basic single-ring topology, failure of one device or connection may interrupt communication across the whole network. More advanced ring designs may include redundancy.

Classic ring networks are uncommon today but may still be found in industrial environments, factories, older infrastructure, or specialized systems.

### Mesh Topology

In a **mesh topology**, network devices have multiple connections to other devices.

The main goal is usually redundancy.

Instead of depending on one communication path, traffic may have several possible paths between devices.

Mesh designs are commonly used between infrastructure devices such as switches, routers, firewalls, and wireless access points rather than normal user endpoints.

#### Full Mesh

In a full mesh, every device has a direct connection to every other device.

This provides very high redundancy, but the number of required links grows quickly as additional devices are added.

A full mesh therefore requires more interfaces, more cabling, additional configuration, and higher cost.

#### Partial Mesh

In a partial mesh, devices have multiple paths, but not every device is connected directly to every other device.

This is much more common in real networks because it provides redundancy without requiring the number of connections needed for a full mesh.

### Point-to-Point Topology

A **point-to-point topology** is the simplest network topology.

It connects exactly two devices.

```text
Router A -------- Router B
```

Point-to-point connections are commonly used between routers, firewalls, network appliances, buildings, and service-provider equipment.

Because only two devices exist on the link, the addressing requirements can be smaller than on networks containing multiple hosts.

IPv4 point-to-point links may use prefixes such as `/30` or `/31`. IPv6 routed point-to-point links may use `/127`.

IPv6 link-local addressing can also be used between directly connected devices, although link-local addresses are not limited only to point-to-point networks.

---

## Comparing Common Topologies

A simplified comparison can be made as follows:

```text
Star
→ Devices connect to a central point

Bus
→ Devices share one communication medium

Ring
→ Devices form a closed loop

Mesh
→ Devices have multiple possible paths

Point-to-point
→ Exactly two devices are connected
```

Real enterprise networks often combine several topology types.

For example:

```text
Endpoints
   |
   | Star
   |
Access Switch
   |
   | Extended Star
   |
Distribution
   |
   | Partial Mesh / Redundant Links
   |
Core Routers
   |
   | Point-to-Point WAN Links
   |
Remote Networks
```

---

## Key Takeaways

Networks can be categorized by the geographic area and purpose they serve.

Common network types include PAN, LAN, WLAN, MAN, and WAN.

The most important distinction when discussing topology is between **physical topology and logical topology**.

Physical topology describes:

> How are the devices actually connected?

Logical topology describes:

> How is communication organized across those connections?

Understanding both views is important because physical topology helps identify redundant links, infrastructure dependencies, communication paths, and single points of failure.

Logical topology helps explain IP addressing, subnets, VLANs, routing, segmentation, and traffic flow.

The important skill is understanding **how the network is physically built and how communication is logically organized on top of that infrastructure**.
