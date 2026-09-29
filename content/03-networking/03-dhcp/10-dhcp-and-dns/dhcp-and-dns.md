# DHCP and DNS

## Overview

DHCP and DNS provide different network services, but they are often closely integrated.

DHCP provides clients with IP configuration, while DNS allows systems to locate hosts and services by name. When these services work together, DHCP can distribute DNS configuration to clients and can also participate in keeping DNS records synchronized with dynamically assigned addresses.

This topic focuses on the relationship between DHCP and DNS rather than repeating DNS fundamentals.

For the underlying DNS concepts, see the DNS section of the knowledge base. DHCP options are covered in [DHCP Options](../06-dhcp-options/dhcp-options.md).

## DHCP Providing DNS Configuration

A DHCP server can provide clients with the DNS information required for name resolution.

For example:

```
Client receives:

IP address:  10.0.20.105
Subnet:      255.255.255.0
Gateway:     10.0.20.1
DNS servers: 10.0.10.10
             10.0.10.11
Domain:      corp.example.com
```

Two important DHCP options are:

```
Option 6
DNS Servers

Option 15
Domain Name
```

Additional DHCP option numbers and their purposes are available in the [DHCP Options Reference](../06-dhcp-options/appendix/dhcp-options-reference.md).

## Why Distribute DNS Configuration Through DHCP?

Without DHCP, administrators may need to configure DNS settings manually on every endpoint.

For a few devices this may be manageable, but it becomes increasingly difficult as the environment grows.

With DHCP:

```
Administrator
      |
      v
DHCP configuration
      |
      +------> Client 1
      +------> Client 2
      +------> Client 3
      +------> Client 4
```

If the organization replaces a DNS server, the DHCP configuration can be updated centrally.

For example:

```
Old DNS server:
10.0.10.10

New DNS server:
10.0.10.20
```

Instead of manually changing every DHCP client, the administrator updates the appropriate DHCP configuration.

Clients can receive the updated information through the DHCP lease lifecycle.

Lease behavior is covered in [DHCP Leases](../04-dhcp-leases/dhcp-leases.md).

## DHCP and Dynamic DNS

There is another important relationship between DHCP and DNS.

DHCP knows which IP addresses are currently being assigned to DHCP clients.

DNS can use this information to maintain hostname-to-address mappings.

For example:

```
DHCP assigns:

PC-01 -> 10.0.20.105
```

DNS can then contain:

```
PC-01.corp.example.com
        |
        v
10.0.20.105
```

Automatically creating or modifying DNS information as addressing changes is commonly called **Dynamic DNS**, or **DDNS**.

## Why Dynamic DNS Is Useful

DHCP addresses can change.

For example:

```
Monday:

PC-01
10.0.20.105
```

Later the lease expires and the client receives another address:

```
Tuesday:

PC-01
10.0.20.143
```

If DNS still contains:

```
PC-01.corp.example.com -> 10.0.20.105
```

the DNS information is stale.

A system attempting to communicate with:

```
PC-01.corp.example.com
```

would receive the old IP address.

With dynamic DNS integration, the DNS record can be updated to reflect the current DHCP allocation:

```
DHCP:

PC-01 -> 10.0.20.143

        |
        v

DNS:

PC-01.corp.example.com -> 10.0.20.143
```

This helps keep hostname-based communication consistent with current addressing.

## DNS Is Especially Important in Domain Environments

Domain environments depend heavily on DNS.

Systems and services commonly use names rather than relying on administrators to know individual IP addresses.

If DNS contains incorrect or outdated information, communication with the intended system can fail or traffic can be directed toward an address that no longer belongs to that system.

For example:

```
Domain system
      |
      | Looks for PC-01
      v
DNS
      |
      | PC-01 -> 10.0.20.105
      v
Network
```

If `10.0.20.105` no longer belongs to PC-01, the DNS information no longer represents the actual network state.

## Who Updates DNS?

There are multiple possible dynamic DNS designs.

A DHCP client may participate in registering DNS information:

```
DHCP Client
     |
     +------> DHCP Server
     |
     +------> DNS Server
```

Another design allows the DHCP server to perform DNS updates on behalf of DHCP clients:

```
DHCP Client
     |
     v
DHCP Server
     |
     | Address assigned:
     | PC-01 -> 10.0.20.105
     v
DNS Server
     |
     v
PC-01.corp.example.com
-> 10.0.20.105
```

The exact behavior depends on the operating systems, DHCP implementation, DNS implementation, and administrative configuration.

## Centralized DNS Updates

Organizations may prefer to control which systems are allowed to modify DNS records.

Instead of allowing arbitrary endpoints to freely modify important DNS information, dynamic updates can be restricted to authorized systems.

Depending on the environment, authorized updates may be performed by:

```
DHCP servers
Authorized domain clients
Administratively managed systems
ICT administrators
```

In environments supporting authenticated or secure dynamic DNS updates, DNS zones can be configured so that only authorized systems can create or modify records.

The important principle is that DNS updates should be controlled rather than allowing untrusted devices to arbitrarily modify important DNS information.

## Forward DNS Records

Suppose DHCP assigns:

```
Hostname:
PC-01

IP address:
10.0.20.105
```

A forward DNS record can provide:

```
PC-01.corp.example.com
        |
        | A record
        v
10.0.20.105
```

This allows systems to locate the workstation using its hostname.

For normal hostname-to-IPv4 communication, the forward record is the important mapping.

## Reverse DNS Records

A reverse DNS record provides the opposite relationship:

```
10.0.20.105
        |
        | PTR record
        v
PC-01.corp.example.com
```

A PTR record is not required for ordinary hostname-to-IP communication with a workstation.

However, reverse DNS can still be useful for:

```
Troubleshooting
Logging
Monitoring
Administrative tools
Security systems
Services that perform reverse lookups
```

Whether reverse records for DHCP clients are dynamically maintained depends on the organization's DNS design.

Some environments dynamically update both forward and reverse records, while others manage reverse DNS differently or do not maintain PTR records for every ordinary workstation.

For more information about A and PTR records, see the DNS records topic in the DNS section.

## DHCP Lease Creation and DNS

When DHCP assigns an address, DNS information can be created or updated to reflect the new lease.

Conceptually:

```
Client:
PC-01

        |
        v

DHCP lease:
10.0.20.105

        |
        v

DNS update

        |
        v

PC-01.corp.example.com
-> 10.0.20.105
```

This allows DNS to follow the addressing information managed by DHCP.

## What Happens When a Lease Disappears?

Suppose DNS contains:

```
PC-01.corp.example.com
-> 10.0.20.105
```

The workstation is later decommissioned and its DHCP lease is no longer valid.

Eventually, DHCP may make `.105` available for another client.

The old DNS information should not remain indefinitely if it no longer represents the actual host using that address.

In a DHCP/DNS-integrated environment, the DHCP server can participate in removing DNS records associated with leases that are no longer valid.

Conceptually:

```
Lease removed or expired
        |
        v
DHCP Server
        |
        | Dynamic DNS update
        v
DNS Server
        |
        v
Old record removed
```

The exact cleanup behavior depends on the DHCP and DNS configuration.

DHCP does not universally delete every DNS record immediately when a lease expires.

## DNS Aging and Scavenging

DNS platforms can also provide mechanisms for identifying and removing stale dynamically registered records.

These mechanisms are commonly referred to as **aging and scavenging** in DNS implementations that support them.

Conceptually:

```
Dynamic DNS record
      |
      | Becomes old and is no longer refreshed
      v
Stale record
      |
      | Cleanup mechanism
      v
Record removed
```

Depending on the environment, stale-record cleanup may therefore involve:

```
DHCP-driven DNS deletion

and/or

DNS aging and scavenging
```

The exact implementation and timing should be configured carefully so valid records are not removed unexpectedly.

## Stale DNS Records

A stale DNS record is information that remains in DNS even though it no longer represents the current host-to-address relationship.

For example:

```
Old machine:

PC-01 -> 10.0.20.105
Lease expired
```

DHCP later assigns the same address to another machine:

```
New machine:

PC-27 -> 10.0.20.105
```

If the old record remains, DNS might contain:

```
PC-01.corp.example.com -> 10.0.20.105
PC-27.corp.example.com -> 10.0.20.105
```

An administrator attempting to reach:

```
PC-01.corp.example.com
```

could resolve:

```
10.0.20.105
```

but that address now belongs to PC-27.

The traffic can therefore reach the wrong machine.

## Why Stale Records Matter

Stale records can affect systems that rely on accurate hostname information.

Examples include:

```
Remote administration
Monitoring
Inventory systems
Automation scripts
Domain-related communication
Security tools
Troubleshooting
```

DNS itself may still be functioning correctly.

The problem is that the DNS database contains outdated information.

This distinction is useful during troubleshooting.

## DHCP Reservation and DNS

A device with a DHCP reservation consistently receives a particular address through DHCP.

For example:

```
Printer-01
     |
     v
DHCP reservation:
10.0.20.50
```

DNS can provide:

```
printer-01.corp.example.com
        |
        v
10.0.20.50
```

Even though the IP address is predictable, using a hostname remains useful.

DHCP reservations were covered in [DHCP Reservations and Exclusions](../07-reservations-and-exclusions/reservations-and-exclusions.md).

## Why Use a Name for a Reserved Device?

Suppose users and administrators access a printer using:

```
printer-01.corp.example.com
```

Today DNS contains:

```
printer-01.corp.example.com
        |
        v
10.0.20.50
```

Later the printer is replaced or moved to another network:

```
printer-01.corp.example.com
        |
        v
10.0.30.50
```

The device address changed, but the logical name remained the same.

Systems using the hostname do not need to know the new address as long as DNS is updated correctly.

This separates:

```
Name of the resource
```

from:

```
Current IP address of the resource
```

The DHCP reservation provides predictable addressing, while DNS allows clients and administrators to avoid depending directly on the numeric address.

## Reservation Does Not Automatically Mean DNS Registration

A DHCP reservation and a DNS record are separate concepts.

A reservation tells DHCP which address should be provided to a particular DHCP client.

```
Reservation:

Printer-01 -> 10.0.20.50
```

A DNS record provides name resolution:

```
printer-01.corp.example.com
-> 10.0.20.50
```

Creating a DHCP reservation does not universally guarantee that a corresponding DNS record will automatically exist.

Dynamic DNS registration must also be configured appropriately, or the DNS record must be managed separately.

## DHCP and DNS Troubleshooting

Consider a workstation with:

```
Hostname:    PC-01
IP:          10.0.20.105
Gateway:     10.0.20.1
DNS server:  10.0.10.10
```

The workstation can successfully communicate with a remote IP address:

```
ping 10.0.10.20
```

but cannot resolve:

```
server-01.corp.example.com
```

This provides an important troubleshooting clue.

Basic IP communication is working, and the client already has a valid IP configuration.

The problem is more likely related to DNS than to basic DHCP address allocation.

## Troubleshooting a Static DNS Record

If the destination is a server with a manually maintained DNS record, useful checks include:

```
Does the DNS record exist?

Does the record contain the correct IP address?

Is the client using the expected DNS server?

Can the client reach the DNS server?

Can the DNS server answer the query?

Is the client using the expected DNS suffix/domain configuration?
```

If the record is missing or incorrect, the problem should be investigated in DNS.

## Troubleshooting a Dynamically Registered Client

For a DHCP client whose DNS record should be dynamically maintained:

```
Does the client have an active DHCP lease?
        |
        v
Does DNS contain the expected record?
        |
        v
Does the record contain the current leased address?
```

If the client has a valid lease but the expected DNS record is missing or outdated, the DHCP/DNS update process should be investigated.

Useful checks include:

```
Is dynamic DNS enabled?

Who is expected to register the record?

Is the DHCP server authorized to update DNS?

Does the DNS zone accept the expected dynamic updates?

Does the hostname match the expected client?

Is the DNS record owned or controlled by another updater?

Are stale records being cleaned up correctly?
```

## DHCP and DNS Are Separate Services

DHCP and DNS can be integrated, but they should still be understood as separate services.

```
DHCP
        |
        v
Provides IP configuration
and manages leases
```

while:

```
DNS
        |
        v
Provides name resolution
```

A DHCP lease can work while DNS is broken.

Similarly, a valid DNS record can exist even when DHCP is unavailable.

Troubleshooting should identify which service is actually failing instead of treating DHCP and DNS as one system.

## Example End-to-End Process

Consider:

```
Client:
PC-01

DHCP Server:
10.0.10.20

DNS Server:
10.0.10.10
```

A simplified integrated process can look like:

```
1. PC-01 requests DHCP configuration.

PC-01
   |
   v
DHCP Server


2. DHCP assigns an address.

PC-01
10.0.20.105


3. DHCP provides DNS configuration.

DNS server:
10.0.10.10

Domain:
corp.example.com


4. DNS information is registered or updated.

PC-01.corp.example.com
        |
        v
10.0.20.105


5. Other systems can resolve the client by name.

PC-01.corp.example.com
        |
        v
10.0.20.105


6. If the DHCP address later changes, DNS can be updated.

PC-01
10.0.20.143

        |
        v

PC-01.corp.example.com
-> 10.0.20.143


7. When the lease/device disappears, stale DNS information should eventually be removed.
```

The exact registration and cleanup responsibilities depend on the environment.

## Key Takeaways

- DHCP and DNS are separate services but are often closely integrated.
- DHCP can provide DNS server addresses to clients through DHCP Option 6.
- DHCP can provide domain information through DHCP Option 15.
- Distributing DNS configuration through DHCP allows administrators to manage client DNS settings centrally.
- Dynamic DNS allows DNS records to follow changes in DHCP-assigned addresses.
- Stale DNS information can direct hostname-based communication toward an old or incorrect address.
- Accurate DNS is particularly important in domain environments and other systems that depend heavily on hostname-based communication.
- Dynamic DNS updates can be performed by DHCP servers, authorized clients, or other managed systems depending on the environment.
- DNS updates should be controlled so untrusted systems cannot arbitrarily modify important records.
- A records provide hostname-to-IPv4 address resolution.
- PTR records provide reverse IP-to-name resolution.
- PTR records are not required for ordinary workstation hostname-to-IP communication but can be useful for administration, logging, monitoring, and other services.
- DHCP can participate in creating, updating, and removing DNS information associated with leases.
- DNS aging and scavenging can also be used to remove stale dynamically registered records.
- DHCP lease expiration does not universally mean that the related DNS record is immediately deleted.
- A DHCP reservation and a DNS record are separate concepts.
- A reserved device can still benefit from DNS because the device or address can change while the logical hostname remains consistent.
- If IP communication works but hostname resolution fails, DNS should be investigated before assuming that DHCP address allocation is broken.
- For dynamically registered clients, a valid DHCP lease combined with a missing or incorrect DNS record can indicate a problem with the DHCP/DNS update process.