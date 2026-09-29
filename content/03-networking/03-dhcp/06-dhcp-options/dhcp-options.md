# DHCP Options

## Overview

DHCP does more than assign an IP address to a client. It can also provide additional network configuration through **DHCP options**.

The basic DHCP allocation process was covered in [DHCP DORA Process](../03-dora-process/dora-process.md), while scopes and per-network DHCP configuration were introduced in [DHCP Scopes and Pools](../05-scopes-and-pools/scopes-and-pools.md).

DHCP options allow administrators to centrally provide clients with information such as the default gateway, DNS servers, domain name, time servers, and other environment-specific settings.

## Why DHCP Options Are Needed

Consider a client that receives only:

```
IP address:  192.168.10.105
Subnet mask: 255.255.255.0
```

The client has an address appropriate for `192.168.10.0/24` and can communicate directly with reachable devices on its local network.

However, additional information is normally required for full network functionality.

For example:

```
Default gateway: 192.168.10.1
DNS server:      192.168.10.10
```

Without the correct default gateway, the client does not have the normal route needed to reach destinations outside its local network.

Without DNS server information, the client cannot normally resolve names through DNS, although direct communication by IP address may still work when routing permits it.

The role of the default gateway was covered in [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md), while DNS was covered in [DNS Fundamentals](../../02-dns/01-dns-fundamentals/dns-fundamentals.md).

## DHCP Option Numbers

DHCP options are identified by option numbers.

Some commonly encountered DHCPv4 options include:

```
Option 3   - Router / Default Gateway
Option 6   - DNS Servers
Option 15  - Domain Name
Option 42  - NTP Servers
Option 55  - Parameter Request List
```

Administrators do not need to memorize every DHCP option number, but recognizing common options is useful when configuring DHCP servers, troubleshooting clients, or examining DHCP traffic in packet captures.

A broader lookup table is maintained separately so this article can remain focused on understanding how DHCP options work rather than becoming a long list of option numbers.

For additional DHCP options and their purposes, see the [DHCP Options Reference](appendix/dhcp-options-reference.md).

## Option 3 - Router

**Option 3** provides router addresses to the client and is commonly used to provide the client's default gateway.

For example:

```
Client network:
10.0.20.0/24

Option 3:
10.0.20.1
```

The client can then use `10.0.20.1` as the appropriate gateway for reaching remote networks.

Because different IP networks normally have different local gateways, this option is commonly configured according to the scope being served.

## Option 6 - DNS Servers

**Option 6** provides DNS server addresses.

For example:

```
Option 6:

10.0.10.10
10.0.10.11
```

The client can add both addresses to its DNS configuration:

```
DNS servers:

10.0.10.10
10.0.10.11
```

Providing multiple DNS servers can improve resilience by giving the client more than one configured resolver.

The exact behavior and order in which configured DNS servers are queried depends on the operating system and resolver implementation. It should not automatically be assumed that one server is always used until it fails and only then the second server is contacted.

DNS resolver behavior and DNS infrastructure were introduced in the [DNS Fundamentals](../../02-dns/01-dns-fundamentals/dns-fundamentals.md) section.

## Option 15 - Domain Name

**Option 15** can provide a domain name associated with the client network.

For example:

```
Option 15:

corp.example.com
```

This allows the DHCP server to communicate the configured domain information to clients instead of requiring it to be manually configured on each endpoint.

## Option 42 - NTP Servers

**Option 42** can provide addresses of NTP servers used for network time synchronization.

For example:

```
Option 42:

10.0.10.20
```

A network may use internal time infrastructure and distribute the appropriate NTP server information to clients through DHCP.

This is another example of centralized configuration. If the required NTP infrastructure changes, DHCP configuration can be updated centrally rather than manually changing every endpoint that receives the setting through DHCP.

## Centralized Configuration

One of the major advantages of DHCP options is centralized management.

Consider an organization with hundreds of workstations.

Instead of manually configuring each workstation with:

```
DNS servers:
10.0.10.10
10.0.10.11

Domain:
corp.example.com

NTP server:
10.0.10.20
```

the administrator can configure the appropriate DHCP options.

Clients can then receive those settings as part of their DHCP configuration.

If a value changes later, the administrator updates the DHCP configuration rather than manually modifying every DHCP-managed endpoint.

When existing clients receive updated configuration depends on their DHCP state and lease lifecycle, which was covered in [DHCP Leases](../04-dhcp-leases/dhcp-leases.md).

## Different Networks Need Different Options

Not every DHCP scope should necessarily provide identical options.

Consider:

```
VLAN 20 - Employees
10.0.20.0/24

VLAN 30 - IT
10.0.30.0/24

VLAN 40 - Guests
10.0.40.0/24
```

All corporate networks might use the same DNS infrastructure:

```
DNS servers:

10.0.10.10
10.0.10.11
```

However, each network requires an appropriate gateway:

```
VLAN 20 -> 10.0.20.1
VLAN 30 -> 10.0.30.1
VLAN 40 -> 10.0.40.1
```

Giving every client the same default gateway would result in incorrect configuration for clients located in other subnets.

This is why DHCP implementations commonly allow options to be associated with particular scopes or pools.

## Shared and Scope-Specific Configuration

Some configuration may be common across many networks, while other settings need to be network-specific.

Conceptually:

```
Shared configuration

DNS:
10.0.10.10
10.0.10.11


VLAN 20

Gateway:
10.0.20.1


VLAN 30

Gateway:
10.0.30.1


VLAN 40

Gateway:
10.0.40.1
```

The exact configuration hierarchy and inheritance behavior depend on the DHCP implementation.

The important concept is that DHCP allows administrators to provide appropriate configuration to different groups of clients without requiring every network to receive identical settings.

## Client-Specific Options

Some DHCP implementations also allow configuration to be associated with a specific reservation or client.

Conceptually:

```
General DHCP configuration
        |
        v
Scope-specific configuration
        |
        v
Reservation or client-specific configuration
```

This can be useful when one particular device requires different or additional DHCP-provided settings without applying those settings to every other client in the same scope.

For example, most devices in a scope may receive standard gateway and DNS information, while a specific reserved device may require additional configuration.

The exact override and inheritance behavior depends on the DHCP server implementation.

## Environment-Specific DHCP Options

DHCP can support more specialized network requirements in addition to common gateway, DNS, domain, and time settings.

One example is **PXE network booting**, where clients may require additional boot-related information.

A deployment network may therefore require DHCP-related configuration that is unnecessary on a normal employee or guest network.

PXE should not be simplified to a single universal DHCP option. Depending on the environment, PXE boot can involve multiple DHCP options and may also use services such as proxyDHCP.

The important DHCP concept is that different networks and different types of clients may require different sets of options.

## Incorrect DHCP Options

A client receiving an IP address does not automatically mean that its complete DHCP configuration is correct.

Consider:

```
Client IP: 10.0.20.105
Mask:      255.255.255.0
Gateway:   10.0.30.1
DNS:       10.0.10.10
```

The client successfully obtained an address from the correct network, but the configured gateway is wrong.

`10.0.30.1` belongs to a different `/24` network than the client.

The important troubleshooting distinction is:

```
DHCP communication succeeded.

The client received a lease.

But the DHCP server delivered incorrect configuration.
```

Some DHCP implementations may validate parts of an administrator's configuration and warn about obvious mistakes, but this should not be assumed for every platform or every option.

When troubleshooting DHCP, the administrator should therefore verify not only that the client received an address, but also that the options supplied to the client are correct.

## Option 55 - Parameter Request List  

DHCP clients do not necessarily need every option supported by a DHCP server.

A DHCPv4 client can include **Option 55**, the Parameter Request List, to indicate which configuration parameters it would like the server to provide.

Conceptually:

```
Client:

"I would like these parameters:
- Subnet mask
- Router
- DNS servers
- Domain name
..."
```

The DHCP server can then provide applicable configured options in its response.

The exact options requested can vary between clients and operating systems.

Option 55 is especially useful to recognize when examining DHCP traffic in packet captures because it provides information about which parameters a particular client is requesting.

## Configured Options and Client Requests

The DHCP server can only provide options that are applicable and available through its configuration and DHCP behavior.

At the same time, clients can indicate which parameters they are interested in receiving.

The resulting DHCP configuration therefore depends on factors such as:

```
Server configuration
        +
Scope configuration
        +
Client-specific configuration
        +
Client requests and behavior
```

The exact precedence and behavior depend on the DHCP implementation.

## Key Takeaways

- DHCP options provide additional network configuration beyond the client's IP address.
- Common DHCPv4 options include the default gateway, DNS servers, domain name, and NTP servers.
- DHCP options are identified by standardized option numbers.
- Option 3 is commonly used to provide router or default gateway information.
- Option 6 provides DNS server addresses and can contain multiple DNS servers.
- Option 15 provides domain name information.
- Option 42 can provide NTP server addresses.
- DHCP options allow network settings to be managed centrally instead of manually configuring every client.
- Different scopes can require different DHCP options, especially different default gateways.
- Some DHCP implementations support shared, scope-specific, and reservation or client-specific configuration.
- Specialized environments such as PXE deployment networks may require additional DHCP-related information.
- Receiving a valid IP address does not prove that all DHCP options are configured correctly.
- Incorrect DHCP options can cause connectivity problems even when the DHCP protocol itself is working.
- Option 55 allows a client to provide a Parameter Request List indicating which configuration parameters it would like to receive.
- The exact option hierarchy, inheritance, and client behavior can vary between DHCP implementations.