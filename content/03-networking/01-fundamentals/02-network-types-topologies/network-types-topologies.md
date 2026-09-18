# Network Types and Topologies

## Overview

Networks can be described in different ways depending on their size, purpose, and structure.

One way is to describe the **area in which the network operates**. This gives us terms such as:

* PAN,
* LAN,
* WLAN,
* MAN,
* WAN.

Another way is to describe the **topology of the network**, meaning how devices are connected and how communication is organized between them.

When discussing topology, it is especially important to distinguish between:

* physical topology,
* logical topology.

A network can have one physical structure while being divided into multiple logical networks.

Understanding this distinction helps when designing infrastructure, identifying single points of failure, planning redundancy, and troubleshooting connectivity.

## Network Types

Network types are commonly categorized based on their geographic scope and purpose.

The borders between these categories are not always strict. For example, one organization may describe communication between several offices as a WAN even when all offices are located within the same city.

The terms are mainly useful for describing the general scale and purpose of the network.

## PAN - Personal Area Network

A **Personal Area Network** is a very small network built around one person and their nearby devices.

PANs are extremely common even though many users may not realize they are using one.

Examples include communication between:

* smartphones,
* wireless headphones,
* smartwatches,
* keyboards,
* mice,
* other personal devices.

PANs commonly use wireless technologies such as Bluetooth.

Their range is normally much smaller than a traditional LAN.

A simple PAN could look like:

```text
        Headphones
            |
Smartwatch--Phone--Laptop
```

The main purpose of a PAN is short-range communication between personal devices.

## LAN - Local Area Network

A **Local Area Network** connects devices within a relatively limited geographic area.

Examples include:

* a home,
* an office,
* a building,
* a small campus.

LAN infrastructure is commonly controlled by one person or organization.

A typical LAN may include:

```text
Computers
    |
Switches
    |
Servers
    |
Printers
    |
Routers
```

LANs commonly use private IPv4 addressing, but private addressing does not define whether a network is a LAN.

A LAN can technically use public addresses, and private addresses can also be used across larger networks.

The important characteristic is that the network operates within a relatively local environment.

## WLAN - Wireless Local Area Network

A **Wireless Local Area Network** provides local network connectivity using wireless communication instead of a physical cable to the endpoint.

WLAN and LAN have many things in common.

The main difference is the access medium.

A traditional wired LAN commonly uses:

* Ethernet cables,
* network switches,
* physical switch ports.

A WLAN commonly uses:

* radio waves,
* wireless access points,
* SSIDs,
* wireless channels.

A typical environment contains both wired LAN and WLAN infrastructure.

For example:

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

Wireless environments introduce additional considerations such as:

* signal strength,
* interference,
* channel usage,
* access point coverage,
* wireless authentication.

## MAN - Metropolitan Area Network

A **Metropolitan Area Network** is larger than a typical LAN but smaller in geographic scope than a traditional WAN.

It commonly connects networks across a city or metropolitan area.

For example, an organization may have several offices located across one city:

```text
Office A LAN
      |
City Network
      |
Office B LAN
      |
Office C LAN
```

A MAN can be used to connect these separate LANs.

In real environments, the distinction between MAN and WAN is not always strict. Networks connecting separate locations are often simply described as WANs.

## WAN - Wide Area Network

A **Wide Area Network** connects networks over larger geographic distances.

A WAN may span:

* cities,
* regions,
* countries,
* continents.

WANs are commonly used to connect separate LANs.

For example:

```text
Office LAN
   |
   | WAN
   |
Datacenter
   |
   | WAN
   |
Remote Office
```

WAN connectivity often depends on infrastructure provided by telecommunications operators or Internet service providers.

Examples may include:

* leased lines,
* MPLS,
* VPN connections,
* Internet connectivity,
* carrier networks.

WAN does not mean that public IP addresses must be used.

A WAN may use either public or private addressing depending on its design.

## IPv4 and IPv6 Addressing

IPv4 and IPv6 approach addressing differently.

IPv4 commonly uses private address ranges inside internal networks and public addresses for Internet-routable communication.

IPv6 does not use RFC1918 private addressing in the same way.

IPv6 includes several address types such as:

* Global Unicast,
* Unique Local Addresses,
* Link-Local addresses.

Subnetting is also approached differently in IPv6.

These subjects are covered in more detail in the IP addressing and IPv6 topics.

## Network Topologies

Network topology describes the arrangement of devices and communication paths in a network.

There are two important ways to describe topology:

```text
Physical Topology
Logical Topology
```

These describe different aspects of the same network.

## Physical Topology

Physical topology describes how network devices are actually connected.

It shows things such as:

* which device connects to which,
* physical network links,
* switches and routers,
* access switches,
* distribution switches,
* core infrastructure,
* redundant connections,
* single points of failure.

For example:

```text
            Core Switch
             /      \
            /        \
Distribution 1    Distribution 2
     |                 |
Access Switch      Access Switch
     |                 |
Endpoints          Endpoints
```

Looking at the physical topology allows an administrator to answer questions such as:

* Is there only one path between devices?
* Is the network redundant?
* What happens if this switch fails?
* Which devices depend on this connection?
* Where are the single points of failure?

## Logical Topology

Logical topology describes how communication is organized across the physical network.

It may include:

* IP addressing,
* subnets,
* VLANs,
* routing relationships,
* broadcast domains,
* network segmentation,
* tunnels,
* logical overlays.

For example, two computers may be physically connected to the same switch:

```text
PC-A
  |
Switch
  |
PC-B
```

but they may belong to different VLANs:

```text
PC-A
VLAN 10
192.168.10.0/24

PC-B
VLAN 20
192.168.20.0/24
```

Physically, both devices are connected to the same switch.

Logically, they belong to different networks and may require Layer 3 routing to communicate.

This is why physical and logical topology should not be treated as the same thing.

A useful way to remember the difference is:

> Physical topology describes how devices are connected.

> Logical topology describes how communication is organized across those connections.

## Star Topology

In a **star topology**, multiple devices connect to one central device.

In modern Ethernet networks, this central device is usually a switch.

For example:

```text
        PC
         |
PC --- Switch --- Server
         |
       Printer
```

Each endpoint has its own connection to the central switch.

This topology is very common in modern wired LANs.

Advantages include:

* easy expansion,
* easier troubleshooting,
* failure of one endpoint cable usually affects only that endpoint,
* good compatibility with switched Ethernet.

The main disadvantage is that the central device can become a single point of failure.

If the central switch fails, devices connected through it may lose connectivity.

### Extended Star

Large networks commonly use an **extended star topology**.

Instead of all devices connecting to one switch, multiple switches are connected together.

For example:

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

This allows the network to scale and introduces additional functionality such as:

* segmentation,
* redundancy,
* routing,
* security controls.

## Bus Topology

In a **bus topology**, all devices share one common communication medium.

Conceptually:

```text
Device --- Device --- Device --- Device
              |
         Shared Medium
```

Traffic transmitted onto the shared medium can be seen by the connected devices, while only the intended recipient should process the communication addressed to it.

Bus networks were used by older Ethernet technologies.

They have several disadvantages:

* shared communication medium,
* poor scalability,
* difficult troubleshooting,
* collisions in older Ethernet implementations,
* failure of the main communication path can affect many devices.

Classic bus Ethernet networks are rarely used in modern computer networks.

However, bus-style communication is still heavily used in other environments, particularly automotive and industrial systems.

A well-known example is **CAN bus**.

## Ring Topology

In a **ring topology**, each device is connected to two neighboring devices, creating a closed loop.

For example:

```text
Device A ----- Device B
   |              |
   |              |
Device D ----- Device C
```

Traffic travels through the ring until it reaches its destination.

Some ring technologies, such as Token Ring, use a **token** that determines which device is currently allowed to transmit.

In a basic single-ring topology, failure of one device or connection may interrupt communication across the whole network.

More advanced ring designs may include redundancy to prevent a single failure from taking down communication.

Classic ring networks are uncommon today but may still be found in:

* industrial environments,
* factories,
* older infrastructure,
* specialized systems.

## Mesh Topology

In a **mesh topology**, network devices have multiple connections to other devices.

The main goal is usually redundancy.

Instead of depending on one communication path, traffic may have several possible paths between devices.

Mesh designs are commonly used between infrastructure devices such as:

* switches,
* routers,
* firewalls,
* wireless access points.

They are less commonly used directly between normal user endpoints.

### Full Mesh

In a full mesh, every device has a direct connection to every other device.

For example:

```text
A ------- B
|\       /|
| \     / |
|  \   /  |
|   \ /   |
|   / \   |
|  /   \  |
| /     \ |
C ------- D
```

This provides very high redundancy.

However, the number of required links grows quickly when additional devices are added.

A full mesh therefore requires:

* more interfaces,
* more cabling,
* additional configuration,
* higher cost.

### Partial Mesh

In a partial mesh, devices have multiple paths, but not every device is connected directly to every other device.

This is much more common in real networks.

It provides redundancy without requiring the number of connections needed for a full mesh.

## Point-to-Point Topology

A **point-to-point topology** is the simplest network topology.

It connects exactly two devices.

For example:

```text
Router A -------- Router B
```

Point-to-point connections are commonly used between:

* routers,
* firewalls,
* network appliances,
* buildings,
* service-provider equipment.

Because only two devices exist on the link, the addressing requirements can be smaller than on networks containing multiple hosts.

IPv4 point-to-point links may use prefixes such as:

```text
/30
```

or:

```text
/31
```

IPv6 routed point-to-point links may use:

```text
/127
```

IPv6 link-local addressing can also be used for communication between directly connected devices, although link-local addresses are not limited only to point-to-point networks.

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

Different topologies are useful for different purposes.

Modern enterprise environments often combine several topology types.

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

Real networks therefore rarely use only one topology everywhere.

## Key Takeaways

Networks can be categorized by the geographic area and purpose they serve.

Common network types include:

```text
PAN
LAN
WLAN
MAN
WAN
```

These categories help describe the scale of a network, but their boundaries are not always strict.

The most important distinction when discussing network topology is between **physical topology and logical topology**.

Physical topology describes:

> How are the devices actually connected?

Logical topology describes:

> How is communication organized across those connections?

A network may therefore have one physical structure while containing many logical networks.

Understanding both views is important because physical topology helps identify:

* redundant links,
* infrastructure dependencies,
* single points of failure,
* physical communication paths.

Logical topology helps understand:

* IP addressing,
* subnets,
* VLANs,
* routing,
* segmentation,
* traffic flow.

The goal is not only to recognize names such as star, mesh, LAN, or WAN.

The important skill is understanding **how the network is physically built and how communication is logically organized on top of that infrastructure**.
