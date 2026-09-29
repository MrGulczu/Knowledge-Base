# DHCP Reservations and Exclusions

## Overview

DHCP normally assigns an available address from a configured pool, but not every device should necessarily receive a different address over time.

Some devices benefit from having a predictable IP address while still receiving their network configuration from DHCP. This is where **DHCP reservations** are useful.

DHCP can also prevent particular addresses from being dynamically allocated through **exclusions**.

Address pools and exclusions were introduced in [DHCP Scopes and Pools](../05-scopes-and-pools/scopes-and-pools.md). This topic focuses on the difference between reservations, exclusions, and static addressing, as well as when each approach is useful.

## DHCP Reservations

A DHCP reservation associates a particular DHCP client with a specific IP address.

For example:

```
DHCP reservation:

Printer -> 192.168.10.25
```

The printer remains configured as a DHCP client.

When it requests network configuration, the DHCP server recognizes the client and assigns the reserved address instead of selecting an arbitrary available address from the normal dynamic pool.

Conceptually:

```
Normal client
        |
        | DHCP request
        v
DHCP Server
        |
        v
Available address from pool


Reserved client
        |
        | DHCP request
        v
DHCP Server
        |
        | Reservation found
        v
Specific reserved address
```

A reservation therefore provides predictable addressing while retaining DHCP-based configuration.

## Reservations Do Not Replace DHCP

A reservation does not configure the address statically on the endpoint.

The device still uses DHCP.

For example:

```
Printer
DHCP enabled
    |
    | DHCP request
    v
DHCP Server
    |
    | Reservation found
    v
192.168.10.25
```

If the printer restarts, it does not simply assume that it owns `.25` permanently. It continues participating in the normal DHCP process.

The reservation changes **which address DHCP assigns to the client**.

The normal DHCP exchange was covered in [DHCP DORA Process](../03-dora-process/dora-process.md).

## Static Addressing vs DHCP Reservation

Both static addressing and DHCP reservations can provide a device with a predictable IP address, but they work differently.

### Static Addressing

With static addressing, the configuration is stored directly on the device.

For example:

```
Printer configuration:

IP address: 192.168.10.35
Mask:       255.255.255.0
Gateway:    192.168.10.1
DNS:        192.168.10.10
```

DHCP does not assign this configuration.

The administrator must ensure that DHCP does not dynamically allocate `192.168.10.35` to another client.

### DHCP Reservation

With a reservation, the endpoint remains a DHCP client.

```
Printer:

DHCP enabled
```

The DHCP server contains:

```
Reservation:

Printer -> 192.168.10.35
```

The server can then provide the reserved address together with the appropriate DHCP options.

```
IP address: 192.168.10.35
Gateway:    192.168.10.1
DNS:        192.168.10.10
```

DHCP options were covered in [DHCP Options](../06-dhcp-options/dhcp-options.md).

## Why Use Reservations?

Reservations combine two useful properties:

```
Predictable IP address
        +
Centralized DHCP configuration
```

A device can consistently receive the same address while still receiving its network parameters through DHCP.

This can simplify administration when settings such as DNS servers, gateways, domain information, or other DHCP options need to change.

Instead of manually modifying the endpoint, the administrator can update the appropriate DHCP configuration.

## How DHCP Identifies a Reserved Client

The DHCP server needs a way to determine which client should receive a reserved address.

A common example is associating the reservation with the client's MAC address:

```
Client:
AA:BB:CC:11:22:33

Reserved address:
192.168.10.25
```

When the DHCP server recognizes the expected client, it can provide the reserved address.

However, DHCP reservations should not universally be described as MAC-address reservations.

DHCP can also use client identification mechanisms such as the **Client Identifier**, carried in DHCPv4 Option 61. The exact reservation and identification behavior depends on the DHCP server and client implementation.

Option 61 and other DHCP options are listed in the [DHCP Options Reference](../06-dhcp-options/appendix/dhcp-options-reference.md).

## What Happens if the Client Identifier Changes?

A reservation depends on the DHCP server recognizing the expected client.

Suppose a reservation is associated with:

```
AA:BB:CC:11:22:33 -> 192.168.10.25
```

The network adapter is later replaced and the device now uses:

```
AA:BB:CC:44:55:66
```

If the reservation depends on the old MAC address, the DHCP server may no longer recognize the device as the reserved client.

The administrator would need to update the reservation with the correct client information.

## MAC Address Randomization

MAC address randomization can also affect reservations.

Modern endpoints, especially wireless clients, may use randomized MAC addresses for privacy.

If the DHCP reservation expects one MAC address but the client begins using another, the reservation may no longer match.

This can make a reservation appear to be malfunctioning even though the DHCP server is operating correctly.

When troubleshooting a reservation, verifying the identifier actually presented by the client is therefore important.

## Reserved Addresses and Other Clients

Suppose the DHCP configuration contains:

```
DHCP pool:
192.168.10.20 - 192.168.10.200

Reservation:
AA:BB:CC:11:22:33 -> 192.168.10.25
```

The reserved `.25` address is intended for that particular client.

Another ordinary DHCP client should not receive `.25` simply because the reserved device is currently offline.

The DHCP server keeps the reservation associated with the intended client.

## Reservation vs Exclusion

Reservations and exclusions can both prevent an address from being normally allocated to another dynamic client, but they serve different purposes.

### Reservation

A reservation associates an address with a particular DHCP client.

```
192.168.10.25
        |
        v
Reserved for a specific DHCP client
```

The intended client can request the address through DHCP.

### Exclusion

An exclusion tells DHCP not to dynamically allocate an address or range.

```
192.168.10.30 - 192.168.10.39
        |
        v
Excluded from dynamic allocation
```

An exclusion does not mean that DHCP should assign those addresses to a particular client.

The addresses are simply unavailable for normal dynamic allocation from that range.

## Example - Reservation and Exclusion Together

Consider:

```
DHCP pool:
192.168.10.20 - 192.168.10.200

Reservation:
AA:BB:CC:11:22:33 -> 192.168.10.25

Exclusion:
192.168.10.30 - 192.168.10.39
```

The reservation means:

```
192.168.10.25
Reserved for the identified DHCP client
```

The exclusion means:

```
192.168.10.30 - 192.168.10.39
Do not dynamically allocate these addresses
```

Although the result for unrelated dynamic clients can look similar, reservations and exclusions are different DHCP mechanisms and should not be treated as identical.

## Static Devices Inside a DHCP Range

Suppose a printer is manually configured with:

```
IP address:
192.168.10.35
```

while the DHCP pool is:

```
192.168.10.20 - 192.168.10.200
```

The printer does not communicate with DHCP to obtain its address.

If DHCP is unaware that `.35` is already being used, it could potentially attempt to allocate the same address to a DHCP client.

An exclusion can prevent this:

```
DHCP exclusion:
192.168.10.35
```

Now the address remains inside the overall configured range but is unavailable for normal dynamic allocation.

Exclusions were introduced in more detail in [DHCP Scopes and Pools](../05-scopes-and-pools/scopes-and-pools.md).

## Static Device vs Reserved Device

The distinction can be summarized as:

```
Static device

Device stores:
IP address
Subnet mask
Gateway
DNS
Other required configuration

DHCP:
Not responsible for assigning the configuration
```

Compared with:

```
Reserved device

Device:
DHCP enabled

DHCP server provides:
Reserved IP address
Subnet configuration
Gateway
DNS
Other configured DHCP options
```

If centralized DHCP management is desired, the device can be configured as a DHCP client and assigned a reservation.

If the device must operate independently of DHCP, static configuration may be more appropriate.

## Reservations and DHCP Options

Depending on the DHCP implementation, a reserved client may also receive client-specific DHCP options.

Conceptually:

```
Normal scope configuration:

Gateway: 192.168.10.1
DNS:     192.168.10.10


Reserved device:

Address: 192.168.10.25
Additional or overridden options
```

This allows an administrator to provide configuration required by a particular DHCP client without necessarily applying the same settings to every device in the scope.

The exact option inheritance and override behavior depends on the DHCP server implementation.

## Reservations and DHCP Leases

A reserved address is still provided through DHCP, so the client continues to participate in the normal lease lifecycle.

For example:

```
Reservation:

Printer -> 192.168.10.75

Lease duration:
8 hours
```

The printer should still attempt to renew its lease according to the normal DHCP lease process.

The reservation does not mean that lease timers stop applying.

## Lease vs Reservation

A lease and a reservation represent different things.

```
Lease
Temporary DHCP allocation state
```

while:

```
Reservation
Persistent DHCP server configuration
```

If a reserved device disappears and its current lease eventually expires, the reservation can remain configured on the DHCP server.

The reserved address remains associated with the intended client rather than simply becoming an ordinary dynamic address for unrelated clients.

Lease duration, renewal, rebinding, and expiration were covered in [DHCP Leases](../04-dhcp-leases/dhcp-leases.md).

## Choosing Which Devices Should Use Reservations

Not every device requires a predictable address.

Normal endpoint devices usually only need a valid network configuration.

For example:

```
Ordinary DHCP:

Employee workstations
Employee laptops
IP phones
Guest devices
```

These devices can normally receive any appropriate address from their DHCP pool.

Other devices often benefit from predictable addressing:

```
Possible DHCP reservations:

Printers
Access points
Other managed infrastructure
```

Predictable addresses can make management, monitoring, access rules, and troubleshooting easier.

## Servers

Servers can use either static addressing or DHCP reservations depending on the environment and operational requirements.

For example:

```
Server
  |
  +-- Static configuration

or

  +-- DHCP reservation
```

A DHCP reservation provides centralized configuration while retaining a predictable address.

Static configuration makes the server's IP configuration independent of DHCP availability.

The appropriate choice depends on the role of the server and the network design.

## Infrastructure Required for DHCP

Some infrastructure should not depend on the DHCP service that it helps provide or reach.

Examples can include:

```
DHCP servers
Routers
Firewalls
Core network infrastructure
```

These devices are commonly configured with static addresses.

For example, if the router providing the default gateway or the DHCP server itself depended on the same DHCP service to obtain essential addressing, failure or startup dependencies could complicate network recovery.

This does not mean that every infrastructure device must always use static addressing. The correct approach depends on the environment, but service dependencies should be considered when deciding between static addressing and DHCP reservations.

## Practical Addressing Example

An organization might use the following approach:

```
Employee laptops
        |
        v
Ordinary DHCP


IP phones
        |
        v
Ordinary DHCP


Guest devices
        |
        v
Ordinary DHCP


Printers
        |
        v
DHCP reservations


Access points
        |
        v
DHCP reservations


Servers
        |
        v
Static addresses
or
DHCP reservations


Routers / Firewalls / DHCP infrastructure
        |
        v
Typically static addresses
```

There is no universal rule requiring every organization to use exactly this design.

The important goal is to choose an addressing method appropriate for the device's role, management requirements, and dependencies.

## Troubleshooting Reservations

If a device does not receive its expected reserved address, useful checks include:

```
Is the device actually configured for DHCP?

Is the reservation enabled?

Is the reservation configured for the correct scope?

Does the reservation contain the correct client identifier?

Did the device's MAC address change?

Is MAC randomization enabled?

Does the client already have another valid lease?

Is another device already using the reserved address?

Are the expected DHCP options configured correctly?
```

A reservation problem does not necessarily mean DHCP itself is unavailable.

The client may successfully communicate with the DHCP server but fail to match the reservation because the server sees a different client identifier than expected.

## Key Takeaways

- A DHCP reservation associates a particular DHCP client with a specific IP address.
- Reservations provide predictable addressing while allowing the endpoint to remain a DHCP client.
- A reservation does not replace DHCP or configure the address statically on the endpoint.
- Reserved clients still participate in the normal DHCP and lease processes.
- A reservation can commonly be associated with a MAC address, but DHCP client identification is not universally limited to MAC addresses.
- DHCPv4 Option 61 can provide a Client Identifier.
- Changes to a client's identifier, including network adapter replacement or MAC randomization, can prevent an existing reservation from matching.
- A reserved address should not normally be dynamically allocated to unrelated clients.
- A reservation and an exclusion are different mechanisms.
- A reservation assigns an address to a particular DHCP client.
- An exclusion prevents DHCP from dynamically allocating an address or range.
- A statically configured device does not receive DHCP options simply because a reservation exists for the same address.
- If centralized DHCP configuration is desired, the endpoint should use DHCP and can be assigned a reservation.
- A lease is temporary allocation state, while a reservation is persistent DHCP server configuration.
- Employee endpoints, IP phones, and guest devices can commonly use ordinary dynamic DHCP.
- Printers, access points, and other managed devices can benefit from DHCP reservations.
- Servers may use either static addressing or reservations depending on their role and environment.
- Critical infrastructure dependencies should be considered before making a device dependent on DHCP.