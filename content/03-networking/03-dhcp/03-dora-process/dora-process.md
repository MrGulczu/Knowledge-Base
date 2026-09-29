# DORA Process

## Overview

The basic purpose of DHCP and the roles of the DHCP client, server, and relay were covered in [DHCP Overview](../01-dhcp-overview/dhcp-overview.md) and [DHCP Components](../02-dhcp-components/dhcp-components.md).

When a DHCPv4 client connects to a network without a usable IPv4 configuration, it needs a way to discover a DHCP server and obtain network settings. The initial address allocation process is commonly described as **DORA**:

```
D - DHCPDISCOVER
O - DHCPOFFER
R - DHCPREQUEST
A - DHCPACK
```

At a high level:

```
DHCP Client                         DHCP Server
     |                                   |
     |---------- DHCPDISCOVER ---------->|
     |<----------- DHCPOFFER ------------|
     |----------- DHCPREQUEST ---------->|
     |<------------ DHCPACK -------------|
     |                                   |
```

The client begins without a usable IPv4 address, so broadcast communication plays an important role during the initial exchange. Broadcast, unicast, and multicast traffic were introduced earlier in [Traffic Types](../../01-fundamentals/11-traffic-types/traffic-types.md).

## DHCP Uses UDP

DHCPv4 by default uses UDP for communication.

```
UDP 67 - DHCP server
UDP 68 - DHCP client
```

UDP itself was covered in [TCP and UDP](../../01-fundamentals/09-tcp-udp/tcp-udp.md), while the purpose of transport-layer ports was covered in [Ports and Sockets](../../01-fundamentals/10-ports-sockets/ports-sockets.md).

## Step 1 - DHCPDISCOVER

When a DHCP-enabled client connects to a network, it may not yet have a usable IPv4 address and may not know the address of a DHCP server.

The client therefore sends a **DHCPDISCOVER** message to locate available DHCP servers.

At this stage, the addressing can look like:

```
Source IP:        0.0.0.0
Destination IP:   255.255.255.255

UDP source port:       68
UDP destination port:  67
```

Conceptually, the client is asking:

```
"I do not have a usable IPv4 configuration.
Is there a DHCP server that can provide one?"
```

The client includes information that allows the DHCP process to identify it. The client's hardware address can participate in this identification, and DHCP can also use a client identifier.

The role of MAC addresses was covered earlier in [Ethernet and MAC](../../01-fundamentals/03-ethernet-mac/ethernet-mac.md).

## Step 2 - DHCPOFFER

A DHCP server that receives the Discover can respond with a **DHCPOFFER**.

Suppose the server manages the following network:

```
Network:      192.168.10.0/24
DHCP pool:    192.168.10.100 - 192.168.10.200
Gateway:      192.168.10.1
DNS server:   192.168.10.10
```

The server might offer:

```
Offered IP:       192.168.10.105
Subnet mask:      255.255.255.0
Default gateway:  192.168.10.1
DNS server:       192.168.10.10
Lease time:       8 hours
```

The purpose of IP addresses, subnet masks, and default gateways was covered in [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md). DNS was covered separately in [DNS Fundamentals](../../02-dns/01-dns-fundamentals/dns-fundamentals.md).

A DHCPOFFER is a proposal. Receiving an offer does not mean that the address allocation process is complete.

## Multiple DHCP Offers

More than one DHCP server may receive the client's Discover and respond.

For example:

```
                    DHCP Client
                    /         \
                   /           \
                  v             v
          DHCP Server A     DHCP Server B
                  |             |
          Offers .105       Offers .150
                  \             /
                   \           /
                    v         v
                    DHCP Client
```

The client can therefore receive multiple offers and select one of them.

This is one reason DHCP needs a separate Request stage rather than allowing the client to immediately begin using the first offered address.

## Step 3 - DHCPREQUEST

After selecting an offer, the client sends a **DHCPREQUEST**.

During the initial allocation process, this message communicates which DHCP server and offered address the client selected.

For example:

```
Server A -> offers 192.168.10.105
Server B -> offers 192.168.10.150

Client selects Server A.

Client broadcasts DHCPREQUEST:

"I selected the offer from Server A
and request 192.168.10.105."
```

Broadcasting the request allows the DHCP servers that participated in the exchange to learn the client's decision.

The selected server can continue the allocation process, while other servers learn that their offers were not selected and can make those proposed addresses available for other clients.

DHCPREQUEST is not used only during the initial DORA process. It also appears during lease renewal and other DHCP states. Those uses are covered in the DHCP leases topic.

## Step 4 - DHCPACK

The selected DHCP server receives the Request and, if the requested configuration is valid, responds with **DHCPACK**, or DHCP Acknowledgement.

Conceptually:

```
DHCPDISCOVER
Client -> "I need network configuration."

DHCPOFFER
Server -> "I can offer you 192.168.10.105."

DHCPREQUEST
Client -> "I want to use 192.168.10.105 from this server."

DHCPACK
Server -> "Confirmed. You can use 192.168.10.105."
```

The DHCPACK confirms that the server has granted the configuration to the client.

After receiving the acknowledgement, the client can configure its interface with the provided parameters and use them for the duration of the lease.

The address is not permanently owned by the client. DHCP normally leases addresses for a defined period, which is covered in [DHCP Leases](../04-dhcp-leases/dhcp-leases.md).

## Complete DORA Exchange

The entire initial process can be represented as:

```
DHCP Client                                      DHCP Server
     |                                                |
     | DHCPDISCOVER                                   |
     | "Is there a DHCP server?"                      |
     |----------------------------------------------->|
     |                                                |
     |                                   DHCPOFFER    |
     |                     "I can offer 192.168.10.105"|
     |<-----------------------------------------------|
     |                                                |
     | DHCPREQUEST                                    |
     | "I want 192.168.10.105 from this server."      |
     |----------------------------------------------->|
     |                                                |
     |                                      DHCPACK   |
     |             "Confirmed. The lease is granted." |
     |<-----------------------------------------------|
     |                                                |
     |          Client configures interface           |
     |                                                |
```

This sequence is the origin of the name **DORA**:

```
Discover -> Offer -> Request -> Acknowledge
```

## DHCPNAK

Not every DHCP request can be accepted.

A DHCP server can respond with **DHCPNAK**, or DHCP Negative Acknowledgement, when the configuration requested by the client is not valid for the current network or cannot be accepted.

A DHCPNAK does not provide the client with a replacement address. Instead, it tells the client that the requested configuration must not be used.

Conceptually:

```
Client
   |
   | DHCPREQUEST
   v
DHCP Server
   |
   | DHCPNAK
   v
Client

"That configuration is not valid.
Do not use it."
```

The client must then return to the DHCP address acquisition process and obtain valid configuration.

One situation where this can matter is when a device moves between different networks.

For example:

```
Previous network: 192.168.10.0/24
Previous address: 192.168.10.105

New network:      10.20.30.0/24
```

The previous IPv4 configuration is not appropriate for the new network. Network addressing and the importance of using configuration appropriate for the local subnet were covered in [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md).

## Broadcast and Unicast DHCP Communication

During initial DHCP discovery, the client does not yet have a usable IPv4 configuration and does not initially know which DHCP server will provide it.

Broadcast communication therefore allows the client to locate DHCP services on its local network.

Once the client has successfully obtained an address and knows the DHCP server, unicast DHCP communication becomes possible where appropriate.

This does not mean that all DHCP communication becomes unicast permanently. Depending on the client's DHCP state, later communication may use either unicast or broadcast.

The exact behavior becomes particularly important during lease renewal and rebinding and is covered in the DHCP leases topic.

## Key Takeaways

- DORA stands for **Discover, Offer, Request, Acknowledge**.
- DHCPv4 uses UDP ports **67 for servers** and **68 for clients**.
- DHCPDISCOVER allows a client without usable IPv4 configuration to locate DHCP servers.
- DHCPOFFER proposes an address and other network configuration to the client.
- A client can receive offers from multiple DHCP servers.
- DHCPREQUEST identifies the offer selected by the client during initial allocation.
- DHCPACK confirms that the server has granted the lease.
- DHCPNAK tells a client that its requested configuration cannot be used.
- Broadcast communication is important while the client is acquiring its initial configuration.
- Once valid addressing exists, DHCP can use unicast where appropriate.
- A DHCP lease is temporary rather than permanent and will be covered in the next topic.