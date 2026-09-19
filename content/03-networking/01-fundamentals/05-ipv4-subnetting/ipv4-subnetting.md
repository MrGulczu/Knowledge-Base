# IPv4 Subnetting

## Overview

Subnetting is the process of dividing a larger IP network into smaller logical networks.

It is important not only because of address management, but also because it gives administrators better control over how devices communicate.

Without subnetting, a network can become one large flat environment where:

* users,
* servers,
* printers,
* infrastructure,
* network equipment,
* and other devices

all share the same logical network.

That may work technically, but it creates several problems.

It becomes harder to:

* control access between devices,
* separate sensitive systems,
* reduce broadcast traffic,
* understand traffic flow,
* troubleshoot problems,
* design clear firewall and routing policies.

Subnetting allows us to divide the environment into smaller logical areas.

For example:

```text
Users
Servers
Printers
Management
VoIP
Guest Devices
Departments
```

Each of these networks can then have its own addressing, routing, and security rules.

Subnetting should therefore be understood as both an addressing technique and a network-design tool.

---

## Why Subnet Networks

Imagine an organization with 200 employees.

Technically, it would be possible to place:

* all employees,
* all servers,
* printers,
* access points,
* switches,
* management interfaces,
* and other equipment

inside one large network.

However, this creates a flat environment where many devices can potentially communicate with each other even when there is no reason for them to do so.

By dividing the network into subnets, we can create logical separation.

For example:

```text
User Network
     |
     | Routing / Firewall
     |
Server Network
```

Communication between those networks can then be controlled.

This allows administrators to define things such as:

```text
Users
→ can access application servers

Users
→ cannot access management interfaces

Printers
→ cannot initiate connections to user devices

Guest Network
→ Internet access only
```

Subnetting also reduces the size of broadcast domains when combined with Layer 2 segmentation such as VLANs.

This helps limit unnecessary broadcast traffic and makes the network easier to understand.

---

## Subnets and VLANs

Subnets and VLANs are closely related, but they are not the same thing.

A subnet is a **Layer 3 logical network**.

A VLAN provides **Layer 2 segmentation**.

In many enterprise environments, one VLAN is mapped to one subnet.

For example:

```text
VLAN 10
→ 10.10.10.0/24

VLAN 20
→ 10.10.20.0/24

VLAN 30
→ 10.10.30.0/24
```

This creates separation at both Layer 2 and Layer 3.

Traffic between the networks must normally pass through a Layer 3 device such as:

* router,
* Layer 3 switch,
* firewall.

This gives administrators a point where communication can be controlled.

---

## Network and Host Bits

An IPv4 address contains 32 bits.

The CIDR prefix determines how many of those bits identify the network.

For example:

```text
192.168.10.0/24
```

The `/24` means:

```text
24 bits
→ Network portion

8 bits
→ Host portion
```

The remaining host bits determine how many addresses are available inside the subnet.

For IPv4:

```text
Host bits = 32 - prefix length
```

For example:

```text
/24
32 - 24 = 8 host bits
```

---

## Calculating the Number of Hosts

The number of total addresses inside an IPv4 subnet can be calculated using:

```text
2^(number of host bits)
```

For example, a `/24` has 8 host bits:

```text
2^8 = 256 total addresses
```

In a traditional IPv4 subnet, two addresses are reserved:

* network address,
* broadcast address.

This gives the common formula:

```text
Usable hosts = 2^(host bits) - 2
```

For a `/24`:

```text
2^8 = 256

256 - 2 = 254 usable hosts
```

There are exceptions to this rule, such as `/31` point-to-point networks.

---

## Example - /26 Network

Consider:

```text
192.168.10.0/26
```

A `/26` leaves:

```text
32 - 26 = 6 host bits
```

This gives:

```text
2^6 = 64 total addresses
```

and:

```text
64 - 2 = 62 usable hosts
```

Compared with a `/24`, the subnet supports fewer hosts but allows the original network space to be divided into more networks.

---

## Borrowing Host Bits

Subnetting works by taking some of the bits previously available for hosts and using them as additional network bits.

For example, starting with:

```text
192.168.10.0/24
```

and changing the prefix to:

```text
192.168.10.0/27
```

means:

```text
27 - 24 = 3 borrowed bits
```

Those 3 bits can be used to create:

```text
2^3 = 8 subnets
```

Each `/27` then has:

```text
32 - 27 = 5 host bits
```

which gives:

```text
2^5 = 32 total addresses

32 - 2 = 30 usable hosts
```

There is always a trade-off:

```text
More network bits
→ More subnets
→ Fewer hosts per subnet
```

```text
More host bits
→ Fewer subnets
→ More hosts per subnet
```

---

## Subnet Block Size

When working with subnet masks, it is useful to understand the **block size**.

The block size tells us where each subnet begins.

For example:

```text
255.255.255.224
```

The last octet is:

```text
224
```

The block size is:

```text
256 - 224 = 32
```

So the subnet boundaries are:

```text
0
32
64
96
128
160
192
224
```

This gives networks such as:

```text
192.168.10.0/27
192.168.10.32/27
192.168.10.64/27
192.168.10.96/27
192.168.10.128/27
192.168.10.160/27
192.168.10.192/27
192.168.10.224/27
```

---

## Common Subnet Masks

Some common IPv4 subnet sizes are:

```text
/24
255.255.255.0
254 usable hosts
```

```text
/25
255.255.255.128
126 usable hosts
```

```text
/26
255.255.255.192
62 usable hosts
```

```text
/27
255.255.255.224
30 usable hosts
```

```text
/28
255.255.255.240
14 usable hosts
```

```text
/29
255.255.255.248
6 usable hosts
```

```text
/30
255.255.255.252
2 usable hosts
```

The smaller the host portion becomes, the fewer usable addresses remain in each subnet.

---

## Finding the Network and Broadcast Address

A useful method for calculating subnet boundaries is:

```text
1. Determine the prefix
2. Determine the block size
3. Find which block contains the IP address
4. First address = network address
5. Last address = broadcast address
6. Addresses between them = usable hosts
```

Consider:

```text
192.168.10.34/28
```

A `/28` has the subnet mask:

```text
255.255.255.240
```

The block size is:

```text
256 - 240 = 16
```

The subnet boundaries are therefore:

```text
0
16
32
48
64
...
```

The address:

```text
192.168.10.34
```

belongs to the block:

```text
192.168.10.32 - 192.168.10.47
```

Therefore:

```text
Network:
192.168.10.32
```

```text
Usable hosts:
192.168.10.33 - 192.168.10.46
```

```text
Broadcast:
192.168.10.47
```

---

## Variable Length Subnet Masking

**VLSM**, or Variable Length Subnet Masking, allows different subnet sizes to be used inside the same larger address block.

Without VLSM, every subnet would need to use the same prefix size.

With VLSM, subnet sizes can be chosen according to their requirements.

For example:

```text
10.20.0.0/24
```

could be divided into:

```text
10.20.0.0/25
→ 126 usable hosts
```

```text
10.20.0.128/26
→ 62 usable hosts
```

```text
10.20.0.192/27
→ 30 usable hosts
```

```text
10.20.0.224/28
→ 14 usable hosts
```

VLSM allows address space to be used more efficiently.

However, address efficiency should not always be the only goal.

Making every subnet as small as mathematically possible can create operational problems later when the network needs to expand.

---

## Planning for Growth

Consider a department with 20 devices.

Technically, a `/27` provides:

```text
30 usable addresses
```

and would be large enough.

But if the department grows to 40 devices, the subnet becomes too small.

A larger subnet such as:

```text
/24
```

may use more address space, but it also provides more room for expansion.

When working inside a large private range such as:

```text
10.0.0.0/8
```

using larger subnets may be acceptable if it makes the environment easier to manage.

Subnetting should therefore balance:

* address efficiency,
* future growth,
* operational simplicity,
* routing design,
* troubleshooting requirements.

---

## Hierarchical Address Planning

A good addressing scheme should tell administrators something about the network.

Instead of allocating networks randomly, related systems can be grouped inside larger address blocks.

For example:

```text
10.10.0.0/16
→ User / department networks
```

```text
10.148.0.0/16
→ Printers and infrastructure
```

```text
10.186.0.0/16
→ Servers and services
```

Smaller subnets can then be created inside those blocks.

For example:

```text
10.186.10.0/24
10.186.20.0/24
10.186.30.0/24
```

An administrator looking at:

```text
10.186.20.15
```

may immediately understand that the device probably belongs to the server environment.

This makes:

* troubleshooting,
* firewall logs,
* routing,
* documentation,
* network analysis

much easier to understand.

---

## Route Summarization

Hierarchical address planning can also simplify routing.

Consider several server networks:

```text
10.186.10.0/24
10.186.20.0/24
10.186.30.0/24
10.186.40.0/24
```

If all server networks belong to a larger block such as:

```text
10.186.0.0/16
```

some parts of the network may only need a summarized route:

```text
10.186.0.0/16
→ Server infrastructure
```

instead of maintaining many individual routes.

Route summarization can reduce routing-table complexity and make network structure more predictable.

This is one of the advantages of planning address space hierarchically from the beginning.

---

## /30 Point-to-Point Networks

A `/30` subnet contains:

```text
2 host bits
```

which provides:

```text
2^2 = 4 addresses
```

Traditionally:

```text
1 Network address
2 Usable host addresses
1 Broadcast address
```

This makes `/30` a natural fit for a point-to-point connection between two routers.

For example:

```text
192.0.2.0/30

192.0.2.0
→ Network

192.0.2.1
→ Router A

192.0.2.2
→ Router B

192.0.2.3
→ Broadcast
```

---

## /31 Point-to-Point Networks

A `/31` contains only two addresses.

On point-to-point links, both addresses can be used by the two endpoints.

Conceptually:

```text
Router A
192.0.2.0/31

Router B
192.0.2.1/31
```

This provides:

```text
2 total addresses
2 usable addresses
```

Compared with `/30`:

```text
/30
→ 4 total
→ 2 usable
```

```text
/31
→ 2 total
→ 2 usable
```

Using `/31` therefore conserves IPv4 address space.

Older devices or older implementations may not support `/31` correctly, which is why `/30` remains common in some environments.

---

## Subnetting and Security

Subnetting itself does not automatically provide security.

A subnet only creates a logical network boundary.

Security comes from controlling communication between those networks.

This may be done with:

* firewalls,
* access control lists,
* Layer 3 policies,
* network segmentation rules.

For example:

```text
User Network
        |
     Firewall
        |
Server Network
```

The firewall may allow:

```text
Users
→ HTTPS to application servers
```

while blocking:

```text
Users
→ SSH to management servers
```

Subnetting makes this kind of control possible by providing clear network boundaries.

---

## Subnetting and Broadcast Traffic

Subnetting also helps reduce the number of devices affected by broadcast traffic when combined with VLAN segmentation.

Instead of one very large broadcast domain:

```text
All users
All servers
All printers
All infrastructure
```

the network can be divided into multiple smaller broadcast domains.

For example:

```text
User VLAN
→ User subnet
```

```text
Server VLAN
→ Server subnet
```

```text
Printer VLAN
→ Printer subnet
```

This limits the scope of Layer 2 broadcasts and makes the environment easier to manage.

---

## Practical Addressing Philosophy

Subnetting should not be treated only as a mathematical exercise.

A technically valid addressing plan can still be difficult to operate.

A good design should consider:

```text
What does this subnet represent?

Will administrators understand it later?

Can it grow?

Can related networks be summarized?

Will it overlap with remote networks?

Can security policies be applied clearly?
```

Using predictable blocks for specific purposes helps the addressing scheme tell a story about the environment.

For example:

```text
10.10.x.x
→ Users
```

```text
10.148.x.x
→ Infrastructure
```

```text
10.186.x.x
→ Servers
```

The exact ranges are a design decision.

The important principle is consistency.

---

## Subnetting and Troubleshooting

Understanding subnetting helps identify many Layer 3 problems.

Useful questions include:

```text
Is the IP address inside the expected subnet?

Is the prefix correct?

What is the network address?

What is the broadcast address?

Is the destination local or remote?

Is the default gateway inside the local subnet?

Are two networks overlapping?

Is the subnet large enough?

Does the routing table contain the expected network?
```

An incorrect subnet mask can completely change how a host interprets the network.

For example, a host may incorrectly believe that a remote destination is local or that a local destination must be reached through a router.

Understanding subnet boundaries makes these problems much easier to identify.

---

## Key Takeaways

Subnetting divides a larger IP network into smaller logical networks.

Its purpose is not only to conserve addresses.

Subnetting helps improve:

* network organization,
* segmentation,
* security design,
* routing,
* troubleshooting,
* broadcast control.

The CIDR prefix determines how many bits belong to the network and how many remain for hosts.

A useful formula is:

```text
Host bits = 32 - prefix
```

and for traditional IPv4 subnets:

```text
Usable hosts = 2^(host bits) - 2
```

More network bits provide more subnets but fewer hosts in each subnet.

More host bits provide larger subnets but fewer separate networks.

Subnetting should also be designed with future growth in mind.

The smallest possible subnet is not always the best subnet.

A good addressing plan should be:

* predictable,
* understandable,
* expandable,
* easy to summarize,
* easy to troubleshoot.

Most importantly, subnetting should help an administrator look at the network and understand how the environment is logically organized.
