# DHCPv6

## Overview

IPv6 changes the way hosts obtain network configuration.

With IPv4, DHCP is commonly responsible for providing the client's address, subnet information, default gateway, DNS servers, and other options. IPv6 provides additional mechanisms, especially **Router Advertisements (RA)** and **Stateless Address Autoconfiguration (SLAAC)**.

As a result, DHCPv6 does not simply reproduce DHCPv4 using larger addresses.

An IPv6 host may use:

```
SLAAC only

SLAAC + Stateless DHCPv6

Stateful DHCPv6
```

The selected design depends on the network and administrative requirements.

This topic focuses specifically on DHCPv6. General IPv6 addressing and SLAAC concepts were introduced earlier in the IPv6 fundamentals topic.

---

## SLAAC

**SLAAC - Stateless Address Autoconfiguration** allows an IPv6 host to configure its own IPv6 address without requiring a DHCPv6 server to assign that address.

The IPv6 router sends **Router Advertisements** containing information about the local network.

Conceptually:

```
IPv6 Router
     |
     | Router Advertisement
     v
Client
     |
     +--> Learns prefix information
     |
     +--> Creates its own IPv6 address
     |
     +--> Learns about the default router
```

For example:

```
Advertised prefix:

2001:db8:20:10::/64
```

The client can use the advertised prefix to create an address belonging to that network.

```
Example client address:

2001:db8:20:10:xxxx:xxxx:xxxx:xxxx
```

The exact method used to construct the interface portion of the address depends on the client implementation and configuration.

---

## Default Router Discovery

One of the most important differences between DHCPv4 and DHCPv6 is how the default router is learned.

With DHCPv4, the DHCP server can provide the default gateway using DHCP options.

Conceptually:

```
DHCPv4
   |
   v
Default gateway option
```

IPv6 works differently.

The default router is learned through **Router Advertisements**.

```
IPv6 Router
     |
     | Router Advertisement
     v
Client
     |
     v
Learns that the router can be used
as a default router
```

DHCPv6 does not provide the IPv6 default gateway in the same way DHCPv4 does.

This remains true even when stateful DHCPv6 is used to assign IPv6 addresses.

---

## Why Use DHCPv6?

If SLAAC can provide address configuration, DHCPv6 may still be useful for additional centrally managed information.

Examples include:

```
DNS servers
Domain search information
Other DHCPv6 options
```

This means DHCPv6 can complement the configuration learned from Router Advertisements.

For example:

```
Router Advertisement
        |
        +--> Prefix information
        +--> Default router
        +--> Information about address configuration

DHCPv6
        |
        +--> DNS information
        +--> Domain search information
        +--> Other DHCPv6 options
```

---

## DNS Does Not Always Require DHCPv6

DHCPv6 is not the only mechanism capable of providing DNS-related information in IPv6 networks.

Router Advertisements can also provide DNS information through mechanisms such as:

```
RDNSS
Recursive DNS Server

DNSSL
DNS Search List
```

Therefore, an IPv6 network can potentially operate using:

```
Router Advertisement
        +
SLAAC
        +
DNS information through RA
```

without requiring DHCPv6.

The appropriate design depends on the network and the capabilities of the clients and infrastructure.

---

## Stateless DHCPv6

In **stateless DHCPv6**, the DHCPv6 server does not assign and maintain the client's IPv6 address.

The client obtains its address using SLAAC.

DHCPv6 provides additional configuration.

```
Router Advertisement
        |
        | Prefix information
        v
      SLAAC
        |
        v
Client creates IPv6 address


DHCPv6 Server
        |
        | Additional configuration
        v
DNS servers
Domain search information
Other DHCPv6 options
```

In simplified form:

```
Stateless DHCPv6

IPv6 address:
SLAAC

Default router:
Router Advertisement

Additional options:
DHCPv6
```

The DHCPv6 server does not maintain the client's address allocation state, which is why this model is described as **stateless**.

---

## Stateful DHCPv6

In **stateful DHCPv6**, the DHCPv6 server participates in assigning and tracking IPv6 addresses.

For example:

```
Client
   |
   | DHCPv6
   v
DHCPv6 Server
   |
   | Assigns and manages
   v
2001:db8:20:10::105
```

The DHCPv6 infrastructure maintains information about the address assignment.

In simplified form:

```
Stateful DHCPv6

IPv6 address:
DHCPv6

Additional DHCPv6 options:
DHCPv6

Default router:
Router Advertisement
```

Stateful DHCPv6 can be useful in environments where administrators want centralized control and visibility over DHCPv6-managed address assignments.

---

## Stateful DHCPv6 Does Not Replace Router Advertisements

Even when DHCPv6 assigns the client's IPv6 address, Router Advertisements remain important.

For example:

```
                 IPv6 Router
                      |
                      | Router Advertisement
                      | Default router information
                      v
                    Client
                      ^
                      |
                      | IPv6 address
                      | Additional DHCPv6 options
                      |
                DHCPv6 Server
```

The DHCPv6 server and IPv6 router therefore perform different parts of the configuration process.

This is an important difference from the common DHCPv4 model.

---

## SLAAC vs Stateless vs Stateful DHCPv6

The three approaches can be summarized as:

### SLAAC Only

```
Router Advertisement
        |
        +--> Prefix information
        +--> Default router
        |
        v
Client creates its own address
```

Additional information such as DNS can also be provided through Router Advertisements when the environment supports it.

### SLAAC + Stateless DHCPv6

```
Router Advertisement
        |
        +--> Prefix information
        +--> Default router
        |
        v
SLAAC
        |
        v
Client creates IPv6 address


DHCPv6
        |
        v
Additional configuration
```

### Stateful DHCPv6

```
Router Advertisement
        |
        v
Default router information


DHCPv6
        |
        +--> IPv6 address
        +--> Additional options
```

The correct design depends on administrative requirements rather than one method being universally better than the others.

---

## Router Advertisement Flags

Router Advertisements contain information that helps IPv6 hosts determine how network configuration should be obtained.

Two important flags are commonly described as:

```
M flag
Managed Address Configuration

O flag
Other Configuration
```

---

## M Flag

The **M flag** indicates that managed address configuration is available through DHCPv6.

Conceptually:

```
M = 1
   |
   v
Managed address configuration
through DHCPv6
```

This is associated with stateful DHCPv6 address configuration.

---

## O Flag

The **O flag** indicates that additional configuration information is available through DHCPv6.

Conceptually:

```
O = 1
   |
   v
Additional configuration
available through DHCPv6
```

This is commonly associated with stateless DHCPv6, where SLAAC handles addressing while DHCPv6 provides additional information.

---

## The Autonomous Address-Configuration Flag

The M and O flags are not the only information relevant to IPv6 address configuration.

Prefix information in Router Advertisements can contain the **Autonomous address-configuration flag**, commonly called the **A flag**.

When appropriate, this indicates that the advertised prefix can be used for autonomous address configuration through SLAAC.

Conceptually:

```
Prefix Information
        |
        | A flag
        v
Can this prefix be used
for SLAAC?
```

Therefore, M and O alone should not be treated as a complete description of how every client will configure IPv6.

Client behavior can also depend on the operating system and local configuration.

---

## IPv6 Does Not Use Broadcast

DHCPv4 relies heavily on broadcast communication during initial address acquisition.

IPv6 does not use broadcast.

Instead, IPv6 uses mechanisms including multicast.

DHCPv6 clients therefore use multicast when discovering DHCPv6 infrastructure.

---

## DHCPv6 Multicast

A DHCPv6 client can use the well-known multicast address:

```
ff02::1:2

All_DHCP_Relay_Agents_and_Servers
```

Conceptually:

```
Client
   |
   | DHCPv6 SOLICIT
   | Multicast
   v
ff02::1:2
   |
   v
DHCPv6 Server / Relay
```

This performs a similar discovery purpose to the initial DHCPv4 broadcast while using IPv6 multicast rather than broadcast.

---

## DHCPv6 Message Exchange

DHCPv6 does not use the DHCPv4 DORA message sequence.

A basic stateful DHCPv6 exchange can use:

```
SOLICIT
   |
   v
ADVERTISE
   |
   v
REQUEST
   |
   v
REPLY
```

Conceptually:

```
Client
   |
   | SOLICIT
   v
DHCPv6 Server
   |
   | ADVERTISE
   v
Client
   |
   | REQUEST
   v
DHCPv6 Server
   |
   | REPLY
   v
Client
```

The overall goal is familiar from DHCPv4: the client discovers DHCP service, selects configuration, requests it, and receives a response.

However, the DHCPv6 protocol uses its own message types and should not simply be described as DORA.

---

## DHCPv6 Relay

DHCPv6 multicast used by clients is local to the link and does not simply cross routers toward a DHCPv6 server located in another network.

For example:

```
Client network:
2001:db8:20:10::/64

        |
        v

IPv6 Router

        |
        | Different IPv6 network
        v

DHCPv6 Server:
2001:db8:10:10::10
```

If the DHCPv6 server is remote, the router can provide **DHCPv6 relay** functionality.

```
Client
        |
        | DHCPv6 multicast
        v
IPv6 Router
DHCPv6 Relay
        |
        | Relayed DHCPv6 communication
        v
DHCPv6 Server
```

This is conceptually similar to the DHCPv4 relay role covered in [DHCP Relay](../08-dhcp-relay/dhcp-relay.md).

---

## DHCPv6 Relay Messages

DHCPv6 relay does not use the DHCPv4 `giaddr` field.

Instead, DHCPv6 defines relay-specific messages.

Two important examples are:

```
RELAY-FORW
Relay Forward

RELAY-REPL
Relay Reply
```

The relay forwards client DHCPv6 communication toward the server using relay information that allows the server to understand the client link and return the response correctly.

Conceptually:

```
Client
   |
   | DHCPv6
   v
Relay
   |
   | RELAY-FORW
   v
DHCPv6 Server
   |
   | RELAY-REPL
   v
Relay
   |
   v
Client
```

The exact relay configuration depends on the router, firewall, or Layer 3 platform being used.

---

## DHCPv6 Client Identification

DHCPv6 uses a client identification mechanism called a **DUID - DHCP Unique Identifier**.

This differs from simply treating the MAC address of one network interface as the identity of the entire DHCP client.

Conceptually:

```
Device
   |
   v
DUID
   |
   v
DHCPv6 client identity
```

This can be useful when DHCPv6 needs to maintain information associated with a particular client.

---

## DUID and Hardware Changes

Consider a laptop with:

```
Old Ethernet MAC:

AA:BB:CC:11:22:33
```

The network adapter is replaced:

```
New Ethernet MAC:

AA:BB:CC:44:55:66
```

A DHCPv6 implementation using a suitable DUID can identify the DHCPv6 client independently from simply matching the current interface MAC address.

This can help maintain client-related DHCPv6 configuration, including managed address assignments or reservations.

However, a DUID should not be described as universally permanent.

Whether it remains unchanged after hardware replacement, operating-system reinstallation, or other changes depends on the **DUID type and client implementation**.

---

## IAID

DHCPv6 also commonly uses an **IAID - Identity Association Identifier**.

The DUID identifies the DHCPv6 client, while an IAID helps distinguish identity associations associated with the client.

This is useful when one client has multiple interfaces or requires multiple DHCPv6-managed address associations.

In simplified form:

```
DUID
Identifies the DHCPv6 client

IAID
Helps identify an address association
for that client
```

The exact implementation details are more advanced than required for basic DHCPv6 administration, but recognizing both terms is useful when reading DHCPv6 leases, logs, and packet captures.

---

## DHCPv6 Reservations

Stateful DHCPv6 implementations can support predictable assignments associated with DHCPv6 client identity.

Conceptually:

```
Client identity
DUID / IAID information
        |
        v
DHCPv6 Server
        |
        v
Reserved or managed IPv6 assignment
```

The exact reservation mechanism depends on the DHCPv6 server implementation.

As with DHCPv4 reservations, administrators should understand which client identifier the server actually uses when matching the reservation.

DHCPv4 reservations were covered in [DHCP Reservations and Exclusions](../07-reservations-and-exclusions/reservations-and-exclusions.md).

---

## DHCPv4 and DHCPv6 Are Independent

A dual-stack host can use IPv4 and IPv6 simultaneously.

The configuration mechanism used for one protocol does not need to match the mechanism used for the other.

For example:

```
Dual-stack Client
        |
        +-----------------------+
        |                       |
        v                       v
      IPv4                    IPv6
        |                       |
        v                       v
     DHCPv4            Router Advertisement
                                |
                                v
                              SLAAC
                                +
                       Stateless DHCPv6
```

The client could receive:

```
IPv4:

Address:
10.0.20.105

Configuration method:
DHCPv4
```

while simultaneously using:

```
IPv6:

Address:
2001:db8:20:10::1234

Address configuration:
SLAAC

Default router:
Router Advertisement

Additional configuration:
Stateless DHCPv6
```

Another network might instead use:

```
IPv4:
DHCPv4

IPv6:
Stateful DHCPv6 + Router Advertisements
```

or:

```
IPv4:
DHCPv4

IPv6:
SLAAC without DHCPv6
```

These are separate configuration processes.

---

## DHCPv4 Working Does Not Prove DHCPv6 Works

Because IPv4 and IPv6 configuration are independent, one protocol can work while the other is misconfigured.

For example:

```
DHCPv4:
Working

IPv4 connectivity:
Working


DHCPv6:
Misconfigured

IPv6 connectivity:
Not working correctly
```

The opposite is also possible.

When troubleshooting a dual-stack network, IPv4 and IPv6 should therefore be checked independently.

---

## Stateful DHCPv6 in a Corporate Network

An organization may choose stateful DHCPv6 for centrally managed employee networks.

For example:

```
Corporate Workstation
        |
        +---- Router Advertisement
        |        |
        |        +--> Default router
        |
        +---- Stateful DHCPv6
                 |
                 +--> Managed IPv6 address
                 +--> DNS information
                 +--> Domain information
                 +--> Other DHCPv6 options
```

This can provide centralized visibility and management of DHCPv6-controlled address assignments.

However, stateful DHCPv6 should not automatically be considered more secure or universally better than SLAAC.

SLAAC is a normal IPv6 address-configuration mechanism and can also be appropriate in enterprise environments.

The design should match the organization's operational and management requirements.

---

## SLAAC in Less Centrally Managed Networks

A network such as guest Wi-Fi may not require centrally managed DHCPv6 address allocation.

A simpler design could use:

```
Router Advertisement
        |
        v
SLAAC
        |
        v
IPv6 address configuration
```

DNS information could potentially be provided through Router Advertisements as well.

This reduces the requirement for a DHCPv6 server while still providing normal IPv6 connectivity.

Again, this is a design choice rather than a requirement that all guest networks must use SLAAC.

---

## DHCPv6 Troubleshooting

When IPv6 clients are not receiving the expected configuration, useful checks include:

```
Are Router Advertisements reaching the client?

Is the expected prefix being advertised?

Is the prefix configured for SLAAC when SLAAC is expected?

What are the RA M and O flags?

Is the client expected to use SLAAC or stateful DHCPv6?

Is DHCPv6 reachable?

Is a DHCPv6 relay required?

Is the DHCPv6 relay configured correctly?

Can the relay reach the DHCPv6 server?

Is the DHCPv6 server assigning the expected addresses or options?

Is the client receiving DNS information?

Is DNS being provided by DHCPv6 or Router Advertisements?

Does the DHCPv6 server see the expected DUID and IAID?

Is IPv4 working while only IPv6 is failing?
```

Troubleshooting should separate the different IPv6 configuration components.

For example:

```
IPv6 address problem
        |
        +--> SLAAC?
        +--> Stateful DHCPv6?


Default router problem
        |
        +--> Router Advertisement


DNS configuration problem
        |
        +--> DHCPv6?
        +--> RDNSS / RA?


Remote DHCPv6 server problem
        |
        +--> DHCPv6 relay?
```

This makes it easier to identify which part of the IPv6 configuration process is actually failing.

---

## Example End-to-End Stateful DHCPv6 Environment

Consider:

```
Client network:
2001:db8:20:10::/64

Router:
2001:db8:20:10::1

DHCPv6 Server:
2001:db8:10:10::10
```

A simplified process could be:

```
1. Router sends a Router Advertisement.

Router
   |
   | RA
   v
Client


2. Client learns information about the IPv6 network
   and default router.


3. RA indicates that managed address configuration
   is available.


4. Client begins DHCPv6 communication.

Client
   |
   | SOLICIT
   v
DHCPv6 Relay


5. Relay forwards the DHCPv6 communication.

Relay
   |
   | RELAY-FORW
   v
DHCPv6 Server


6. DHCPv6 server selects appropriate configuration.


7. Response returns through the relay.

DHCPv6 Server
   |
   | RELAY-REPL
   v
Relay
   |
   v
Client


8. Client receives its DHCPv6-managed address
   and configured DHCPv6 options.


9. The default router remains learned through
   Router Advertisements.
```

This demonstrates why DHCPv6, Router Advertisements, and IPv6 routing should be understood as related but separate components.

---

## Key Takeaways

- DHCPv6 is not simply DHCPv4 with larger addresses.
- IPv6 provides SLAAC as an additional address-configuration mechanism.
- SLAAC allows a host to create an IPv6 address using prefix information received through Router Advertisements.
- IPv6 hosts learn default routers through Router Advertisements rather than a DHCPv6 default-gateway option.
- DHCPv6 can provide additional configuration such as DNS and domain search information.
- DNS information can also be provided through Router Advertisements using mechanisms such as RDNSS and DNSSL.
- Stateless DHCPv6 uses SLAAC for address configuration while DHCPv6 provides additional options.
- Stateful DHCPv6 allows the DHCPv6 server to assign and track IPv6 addresses.
- Stateful DHCPv6 still depends on Router Advertisements for default-router discovery.
- The RA M flag indicates managed address configuration.
- The RA O flag indicates that other configuration is available through DHCPv6.
- The RA prefix A flag is relevant to whether an advertised prefix can be used for SLAAC.
- IPv6 does not use broadcast.
- DHCPv6 uses multicast for server and relay discovery.
- `ff02::1:2` is the All_DHCP_Relay_Agents_and_Servers multicast address.
- DHCPv6 uses message types such as SOLICIT, ADVERTISE, REQUEST, and REPLY rather than DHCPv4 DORA terminology.
- A DHCPv6 relay is required when clients need to reach a DHCPv6 server across a routed boundary and direct communication is not available.
- DHCPv6 relay uses relay-specific messages such as RELAY-FORW and RELAY-REPL rather than the DHCPv4 `giaddr` field.
- DHCPv6 uses DUIDs to identify DHCPv6 clients.
- IAIDs help distinguish address associations belonging to a DHCPv6 client.
- DUID persistence depends on the DUID type and client implementation and should not be assumed to survive every hardware or operating-system change.
- DHCPv4 and DHCPv6 operate independently in dual-stack networks.
- A host can use DHCPv4 for IPv4 while using SLAAC, stateless DHCPv6, or stateful DHCPv6 for IPv6.
- DHCPv4 working does not prove that IPv6 configuration is working.
- Stateful DHCPv6 can be useful when centralized IPv6 address management is desired, but SLAAC remains a normal and valid IPv6 configuration method.