# HCP Components

## Overview

DHCP operation involves several components working together to provide  
clients with appropriate network configuration.

The basic purpose of DHCP was introduced in [DHCP Overview](../01-dhcp-overview/dhcp-overview.md). This topic focuses on the components that make DHCP work.

The three main roles are:

- **DHCP client** --- requests network configuration.
- **DHCP server** --- manages address pools and provides configuration  
    to clients.
- **DHCP relay** --- forwards DHCP communication between clients and  
    servers located in different networks.

A DHCP environment can be very small, with a router providing DHCP  
directly to devices on a home network, or centralized, with one DHCP  
service supporting many different networks.

## DHCP Client

A **DHCP client** is a device that requests network configuration from a  
DHCP server.

Typical DHCP clients include:

- Workstations
- Laptops
- Mobile phones
- Tablets
- Printers
- IP phones
- Other network-connected devices

When a DHCP-enabled client connects to a network, it can request the  
configuration required to communicate on that network.

At a simplified level:

```
DHCP Client
     |
     | Requests configuration
     v
DHCP Server
```

The exact messages exchanged between the client and server will be  
discussed in the DORA topic.

## DHCP Server

The **DHCP server** provides network configuration to DHCP clients.

It does more than simply select random IP addresses. The server  
maintains configured address ranges and keeps track of addresses that  
have been allocated to clients.

A DHCP server may provide information such as:

```
IP address
Subnet mask
Default gateway
DNS servers
Lease duration
Other DHCP options
```

The DHCP server also manages multiple networks by maintaining separate  
address pools or scopes.

For example:

```
VLAN 20
Network:   10.0.20.0/24
DHCP pool: 10.0.20.50 - 10.0.20.200

VLAN 30
Network:   10.0.30.0/24
DHCP pool: 10.0.30.50 - 10.0.30.200
```

When a request arrives, the server must determine which network the  
client belongs to and select an appropriate address from the  
corresponding pool.

## Where Can a DHCP Server Run?

DHCP is a network service and does not require a dedicated physical  
appliance.

DHCP services can be provided by many different systems, including:

- Routers
- Firewalls
- Windows Server
- Linux servers
- Network appliances

In a small network, the router or firewall may act as the DHCP server.

In a larger environment, DHCP may instead be provided by dedicated or  
centralized servers.

A pure Layer 2 switching function does not itself require a DHCP server.  

However, switches may still participate in DHCP-related functionality,  
such as DHCP snooping, and some network devices can provide additional  
services beyond basic Layer 2 switching.

## DHCP Scopes and Address Pools

A **DHCP scope** or **address pool** defines addresses that the DHCP  
server can allocate to clients.

For example, a `/24` network does not necessarily need to make every  
usable address available for dynamic assignment.

```
Network:       10.0.20.0/24
Default GW:    10.0.20.1
DHCP pool:     10.0.20.50 - 10.0.20.200
```

Addresses outside the dynamic pool can remain available for  
infrastructure or devices using static addressing.

A DHCP server commonly contains multiple pools for different networks.

```
DHCP Server
     |
     +-- 10.0.20.0/24
     |     Pool: 10.0.20.50 - 10.0.20.200
     |
     +-- 10.0.30.0/24
     |     Pool: 10.0.30.50 - 10.0.30.200
     |
     +-- 10.0.40.0/24
           Pool: 10.0.40.50 - 10.0.40.200
```

Scopes, pools, exclusions, and their management will be covered in more detail in a dedicated topic.

## DHCP Relay

A DHCP server does not have to exist inside every client network.

Consider a centralized DHCP server:

```
                    DHCP Server
                    10.0.10.10
                         |
                    10.0.10.0/24
                         |
                    Router / L3
                    /          \
                   /            \
          VLAN 20                 VLAN 30
       10.0.20.0/24            10.0.30.0/24
              |                      |
          Client A                Client B
```

A DHCPv4 client initially uses broadcast communication when attempting  
to find a DHCP server.

Routers normally do not forward these broadcasts between IP networks.

Without an additional mechanism, clients in VLAN 20 and VLAN 30 would  
therefore not be able to reach the DHCP server in `10.0.10.0/24` through  
their initial DHCP broadcasts.

This is where a **DHCP relay** is used.

## Role of the DHCP Relay

The DHCP relay receives DHCP communication from the client network and  
forwards it toward a configured DHCP server.

Conceptually:

```
DHCP Client
     |
     | DHCP broadcast
     v
DHCP Relay
     |
     | Forwarded toward DHCP server
     v
DHCP Server
```

The relay also provides the DHCP server with information that allows it 
to determine from which network the request originated.

This is essential when one DHCP server supports multiple networks.

For example:

```
Request from VLAN 20
        |
        v
DHCP Relay
        |
        v
DHCP Server
        |
        v
Select 10.0.20.0/24 pool
```

While:

```
Request from VLAN 30
        |
        v
DHCP Relay
        |
        v
DHCP Server
        |
        v
Select 10.0.30.0/24 pool
```

Without this information, the centralized DHCP server would not know  
which network configuration should be provided to the client.

The mechanisms used by DHCP relay will be discussed in more detail in  
the dedicated DHCP relay topic.

## Centralized DHCP Example

Consider the following environment:

```
                    DHCP Server
                    10.0.10.10
                         |
                    Server Network
                    10.0.10.0/24
                         |
                    Router / L3
                    /          \
                   /            \
          VLAN 20                 VLAN 30
       10.0.20.0/24            10.0.30.0/24
              |                      |
          Client A                Client B
```

Suppose Client B connects to VLAN 30.

At a high level, the process is:

```
Client B
   |
   | Sends DHCP request
   v
Router / DHCP Relay
   |
   | Forwards request
   | Identifies originating network
   v
DHCP Server
   |
   | Determines appropriate scope
   | Checks for an applicable reservation
   | Selects an available address
   v
DHCP response
   |
   v
DHCP Relay
   |
   v
Client B
```

Because the request originated from VLAN 30, the DHCP server selects  
configuration appropriate for:

```
10.0.30.0/24
```

rather than incorrectly assigning an address belonging to VLAN 20.

This architecture allows a single centralized DHCP service to support  
many separate networks.

## DHCPv4 and DHCPv6

The examples in this topic primarily describe **DHCPv4**.

DHCPv4 clients can use broadcast communication during the initial  
process of locating a DHCP server.

DHCPv6 operates differently and uses IPv6 multicast rather than  
IPv4-style broadcast communication.

DHCPv6 and its relationship with IPv6 Router Advertisements and SLAAC  
will be covered separately.

## Key Takeaways

The main DHCP components are the **client**, **server**, and **relay**.

The client requests network configuration, while the server manages  
address pools and provides appropriate configuration.

A DHCP server can run on many different platforms and can maintain  
separate pools for multiple networks.

When the DHCP server is located in another network, a DHCP relay can  
forward DHCP communication between the client network and the  
centralized server. The relay also helps the server determine which  
network the client belongs to so that the correct address pool can be  
selected.

Together, these components allow DHCP to scale from a small local  
network to environments containing many separate subnets and VLANs.