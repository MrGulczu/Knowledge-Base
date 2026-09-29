# DHCP Relay

## Overview

DHCP clients often need to obtain configuration from a DHCP server located in a different IP network.

The problem is that the initial DHCPv4 communication relies on local broadcast traffic, and routers do not normally forward broadcasts between IP networks.

A **DHCP relay** solves this problem by receiving DHCP messages from clients on a local network and forwarding them to a DHCP server located elsewhere.

The basic role of the relay was introduced in [DHCP Components](../02-dhcp-components/dhcp-components.md). This topic focuses on how DHCPv4 relay works, how the server identifies the client's network, and how relay-related problems can be troubleshot.

## Why DHCP Relay Is Needed

Consider:

```
VLAN 20 - Employees
10.0.20.0/24

Client
   |
   |
Gateway / Router
10.0.20.1
   |
   | Routed network
   |
DHCP Server
10.0.10.10
```

A new client does not yet have a valid IPv4 configuration.

It begins the DHCP process by sending a DHCPDISCOVER on its local network.

```
Client
   |
   | DHCPDISCOVER
   | Broadcast
   v
VLAN 20
10.0.20.0/24
```

Routers separate broadcast domains and do not normally forward this broadcast into another IP network.

Without additional functionality:

```
Client
   |
   | DHCPDISCOVER
   v
Router
   |
   X  Broadcast not normally routed
   |
DHCP Server
10.0.10.10
```

The remote DHCP server never receives the request.

The normal DHCP exchange was covered in [DHCP DORA Process](../03-dora-process/dora-process.md).

## The DHCP Relay

A router, firewall, Layer 3 switch, or another suitable device can act as a DHCP relay.

The relay receives the DHCP message from the local client network and forwards it toward the configured DHCP server.

```
Client
   |
   | DHCPDISCOVER
   | Local broadcast
   v
DHCP Relay
10.0.20.1
   |
   | Forwarded DHCP message
   v
DHCP Server
10.0.10.10
```

This allows a centralized DHCP server to provide addresses to clients located in other routed networks.

## DHCPv4 and Broadcast Traffic

For DHCPv4, the initial client communication commonly uses broadcast traffic because the client does not yet have the complete IPv4 configuration required for normal communication.

This topic focuses on **DHCPv4 relay**.

DHCPv6 uses different mechanisms, including multicast, and should not be treated as identical to DHCPv4 relay behavior.

## How Does the Server Know the Client Network?

Forwarding the DHCP request creates another problem.

Suppose one DHCP server supports:

```
VLAN 20 - Employees
10.0.20.0/24

VLAN 30 - Wi-Fi
10.0.30.0/24

VLAN 40 - Guests
10.0.40.0/24
```

The DHCP server must determine which scope should be used for each relayed request.

If a request came from VLAN 20, the server should provide an address from:

```
10.0.20.0/24
```

It should not accidentally provide an address belonging to VLAN 30 or VLAN 40.

DHCP scopes and address selection were covered in [DHCP Scopes and Pools](../05-scopes-and-pools/scopes-and-pools.md).

## The giaddr Field

In DHCPv4 relay operation, an important field is:

```
giaddr
Gateway IP Address
```

The relay uses this field to provide information that allows the DHCP server to identify the client network.

For example:

```
Client network:
10.0.20.0/24

Relay interface:
10.0.20.1

DHCP Server:
10.0.10.10
```

The flow can be represented as:

```
Client
   |
   | DHCPDISCOVER
   v
DHCP Relay
10.0.20.1
   |
   | giaddr: 10.0.20.1
   v
DHCP Server
10.0.10.10
```

The DHCP server can use this information to select the appropriate configuration for the client network.

```
giaddr:
10.0.20.1
   |
   v
Select scope:
10.0.20.0/24
   |
   v
Offer address:
10.0.20.105
```

## Returning the DHCP Response

The DHCP server is located in another network and does not simply treat the new client as an ordinary configured routed host.

The server returns the DHCP response toward the relay.

For example:

```
DHCP Server
10.0.10.10
   |
   | DHCPOFFER
   v
DHCP Relay
10.0.20.1
   |
   | Delivers response
   | to client network
   v
Client

Offered address:
10.0.20.105
```

The relay handles delivery of the response onto the client's local network.

Depending on the DHCP flags and circumstances, delivery toward the client can involve broadcast or unicast behavior.

Conceptually, the relay connects:

```
Client side
Local DHCP communication
        |
        v
DHCP Relay
        |
        v
Server side
Routed communication
```

## One Relay Can Serve Multiple Networks

A separate physical relay device is not required for every VLAN.

One router, firewall, or Layer 3 switch can provide relay functionality for multiple connected client networks.

For example:

```
                    DHCP Server
                     10.0.10.10
                         |
                         |
                    Router / L3
                    DHCP Relay
                   /     |      \
                  /      |       \
                 /       |        \
        10.0.20.1    10.0.30.1    10.0.40.1
             |           |            |
          VLAN 20     VLAN 30      VLAN 40
        10.0.20.0/24 10.0.30.0/24 10.0.40.0/24
```

Requests from different networks can carry different relay gateway information.

For example:

```
Request from VLAN 20

giaddr:
10.0.20.1

        |
        v

Use scope:
10.0.20.0/24
```

while:

```
Request from VLAN 40

giaddr:
10.0.40.1

        |
        v

Use scope:
10.0.40.0/24
```

This allows one centralized DHCP infrastructure to support many routed networks.

## Relay Configuration Must Cover the Client Network

Although one physical device can relay DHCP for many networks, relay functionality must be configured to process requests from the networks that require it.

For example:

```
Router / DHCP Relay

VLAN 20
10.0.20.1
Relay configured
      |
      +----> DHCP Server 10.0.10.10
      |
      Result: DHCP works


VLAN 30
10.0.30.1
Relay not configured
      |
      X
      |
      Result: Remote DHCP fails
```

The exact configuration method differs between routers, firewalls, and Layer 3 switches.

Some platforms configure DHCP relay directly on individual interfaces or VLAN interfaces, while others represent the configuration differently.

The important concept is that DHCP requests from each required client network must be handled by the relay.

## Multiple DHCP Servers

A relay can be configured to forward DHCP requests toward multiple DHCP servers.

For example:

```
                    DHCP Server 1
                    10.0.10.10
                         ^
                         |
Client                   |
VLAN 20 ----> DHCP Relay |
                         |
                         v
                    DHCP Server 2
                    10.0.10.11
```

Both reachable servers may respond with offers.

```
DHCP Server 1
Offer: 10.0.20.105
          \
           \
            > Client
           /
          /
DHCP Server 2
Offer: 10.0.20.106
```

The client can select an offer and send a DHCPREQUEST indicating the selected server and requested address.

## DHCP Server Redundancy Requires Planning

Simply pointing a relay toward two independent DHCP servers does not automatically create a safe redundant DHCP design.

If two unrelated servers independently allocate addresses from the same complete pool without coordination, conflicting address assignments can occur.

Redundant DHCP deployments should use an appropriate design such as:

```
DHCP failover
High availability
Split-scope design
Other coordinated allocation mechanism
```

The exact mechanism depends on the DHCP implementation.

The important point is that the relay can forward requests to multiple servers, but address allocation between those servers must still be designed correctly.

## What Happens Without a Relay?

Consider:

```
Client VLAN:
10.0.20.0/24

Gateway:
10.0.20.1

DHCP Server:
10.0.10.10
```

Suppose normal routing works perfectly.

A statically configured client can reach:

```
10.0.10.10
```

However, DHCP relay is not configured.

A new DHCP client sends:

```
DHCPDISCOVER
```

The broadcast reaches the local network and gateway but is not normally routed to the remote DHCP server.

```
Client
   |
   | DHCPDISCOVER
   v
Gateway
10.0.20.1
   |
   X  No DHCP relay
   |
DHCP Server
10.0.10.10
```

The client cannot complete the normal DHCP process with that remote server and does not receive a lease from it.

## Routing Working Does Not Prove DHCP Works

This creates an important troubleshooting scenario:

```
Static IP configured:
Remote communication works

DHCP enabled:
No DHCP lease received
```

Normal routed communication working proves that basic IP connectivity may exist between the networks.

It does not prove that DHCP relay is configured correctly.

When DHCP fails only for remote clients, relay configuration should be investigated.

## DHCP Reservations Also Depend on Relay

A DHCP reservation does not bypass the DHCP process.

Suppose a remote client has:

```
Reservation:
Client -> 10.0.20.75
```

If the client network requires a relay and the relay is missing, the client's DHCP request still cannot reach the remote server.

The reservation therefore cannot be used until DHCP communication succeeds.

Reservations were covered in [DHCP Reservations and Exclusions](../07-reservations-and-exclusions/reservations-and-exclusions.md).

## Per-Network Relay Failures

A relay problem does not necessarily affect every network.

Consider a router serving several VLANs:

```
VLAN 20
Relay configured
DHCP works

VLAN 30
Relay configured
DHCP works

VLAN 40
Relay missing
DHCP fails
```

If multiple VLANs use the same DHCP server and only one network stops receiving addresses, this is a useful troubleshooting clue.

The DHCP server itself may still be functioning normally.

The administrator should investigate the affected network's:

```
Relay configuration
Interface or VLAN configuration
Routing
Firewall or ACL rules
DHCP scope
```

## Firewalls and ACLs

DHCP relay requires communication between the relay and DHCP server.

Even if routing is correct and the relay is configured properly, a firewall or access control list can prevent DHCP from working.

For example:

```
Client VLAN
10.0.20.0/24
      |
      v
DHCP Relay
10.0.20.1
      |
      | Firewall / ACL
      v
DHCP Server
10.0.10.10
```

If the required DHCP traffic is blocked, the DHCP exchange cannot complete.

The necessary traffic must therefore be permitted according to the network design and DHCP implementation.

## DHCPv4 Ports

The fundamental DHCPv4 UDP ports are:

```
UDP 67 - DHCP server
UDP 68 - DHCP client
```

These ports are useful to remember when troubleshooting firewall rules, ACLs, packet captures, and DHCP communication.

With a relay involved, the exact packet flow differs from the simplest client-to-server exchange, but UDP ports 67 and 68 remain fundamental to DHCPv4.

## DHCP Relay and Option 82

A DHCP relay can provide more information than the basic client network identification.

**DHCP Option 82**, the Relay Agent Information option, can carry additional information about where or how a client request entered the network.

Depending on the environment, this can include information such as:

```
Circuit information
Relay identification
Subscriber information
Access information
```

Option 82 can be particularly useful in larger access networks where the DHCP infrastructure needs more detailed information about the client's attachment point.

The common Option 82 sub-options are listed in the [DHCP Options Reference](../06-dhcp-options/appendix/dhcp-options-reference.md).

Option 82 supplements relay information and should not be confused with the basic `giaddr` field used in DHCPv4 relay operation.

## DHCP Relay Troubleshooting

When clients in a remote network cannot obtain DHCP configuration, useful checks include:

```
Is DHCP relay configured for the client network?

Is the correct DHCP server address configured on the relay?

Can the relay reach the DHCP server?

Can the DHCP server return traffic to the relay?

Are firewall and ACL rules permitting the required DHCP traffic?

Is the correct DHCP scope configured?

Does the scope contain available addresses?

Is the relay providing the expected client-network information?

Is the problem affecting one VLAN or every VLAN?

Are redundant DHCP servers configured correctly?
```

Troubleshooting should distinguish between:

```
Client-to-relay problem
Relay configuration problem
Relay-to-server connectivity problem
Firewall / ACL problem
DHCP server problem
Scope or pool problem
```

If DHCP works in several networks but fails in only one, checking the affected network's relay configuration is a useful early step.

## Example End-to-End Flow

Consider:

```
Client network:
10.0.20.0/24

Relay / Gateway:
10.0.20.1

DHCP Server:
10.0.10.10

Available DHCP address:
10.0.20.105
```

The simplified flow is:

```
1. Client sends DHCPDISCOVER as a local broadcast.

Client
   |
   v
10.0.20.0/24


2. DHCP relay receives the request.

Client
   |
   v
Relay
10.0.20.1


3. Relay forwards the DHCP message to the server.

Relay
10.0.20.1
   |
   | giaddr: 10.0.20.1
   v
DHCP Server
10.0.10.10


4. Server selects the appropriate scope.

giaddr: 10.0.20.1
        |
        v
Scope: 10.0.20.0/24


5. Server selects an available address.

10.0.20.105


6. Server returns the response through the relay.

DHCP Server
10.0.10.10
   |
   v
Relay
10.0.20.1
   |
   v
Client


7. DHCP communication continues until the client receives its configuration.
```

The relay allows DHCP to cross the routed boundary without turning normal IP broadcasts into routed broadcasts.

## Key Takeaways

- DHCPv4 clients commonly begin address acquisition using local broadcast traffic.
- Routers do not normally forward broadcasts between IP networks.
- A DHCP relay allows clients to communicate with DHCP servers located in other routed networks.
- Routers, firewalls, Layer 3 switches, and other suitable devices can provide DHCP relay functionality.
- In DHCPv4, the `giaddr` field helps the server determine the network from which a relayed request originated.
- The DHCP server can use relay information to select the correct scope for the client.
- DHCP responses are returned through the relay, which delivers them to the client network.
- One physical relay device can provide DHCP relay services for multiple VLANs or networks.
- Relay functionality must cover each client network that requires access to a remote DHCP server.
- A relay can forward requests to multiple DHCP servers.
- Multiple DHCP servers still require a coordinated redundancy or address-allocation design.
- Normal routing working does not prove that DHCP relay is working.
- Without the required relay, clients cannot obtain leases from a DHCP server located across the routed boundary.
- DHCP reservations do not bypass the need for relay communication.
- A relay problem can affect one VLAN while DHCP continues working normally in other networks.
- Firewalls and ACLs must permit the required DHCP communication.
- DHCPv4 fundamentally uses UDP port 67 for servers and UDP port 68 for clients.
- Option 82 can provide additional relay-agent information beyond the basic `giaddr` mechanism.
- Relay configuration, routing, firewall policy, scope configuration, and address availability should all be considered when troubleshooting remote DHCP clients.