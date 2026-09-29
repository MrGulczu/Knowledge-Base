# DHCP Scopes and Pools

## Overview

A DHCP server does not assign addresses arbitrarily from an entire IP network. Administrators define which addresses are available for dynamic allocation and which network configuration should be provided to clients.

The basic role of the DHCP server and its address pools was introduced in [DHCP Components](../02-dhcp-components/dhcp-components.md). This topic focuses on how scopes, pools, ranges, exclusions, and capacity planning are used to organize DHCP address allocation.

## Scope, Pool, and Range Terminology

The terms **scope**, **pool**, and **range** are sometimes used differently depending on the DHCP implementation.

In general, they describe the configuration that determines which addresses DHCP can allocate within a particular network.

For example:

```
Network:         192.168.10.0/24
Default gateway: 192.168.10.1
DHCP range:      192.168.10.50 - 192.168.10.200
```

Some implementations may refer to the overall configuration for `192.168.10.0/24` as a **scope**, while the addresses from `.50` to `.200` form the dynamic address pool or range inside that scope.

The exact terminology can vary between platforms, but the underlying purpose remains the same, defining which network is being served and which addresses can be allocated dynamically.

## The DHCP Pool Does Not Need to Use the Entire Subnet

A subnet normally contains addresses that should not be dynamically assigned to clients.

Consider:

```
Network: 192.168.10.0/24

192.168.10.0       Network address
192.168.10.1       Default gateway

192.168.10.2
     ...
192.168.10.49      Static or infrastructure space

192.168.10.50
     ...
192.168.10.200     DHCP pool

192.168.10.201
     ...
192.168.10.254     Static or infrastructure space

192.168.10.255     Broadcast address
```

The DHCP server is allowed to dynamically allocate only the configured pool:

```
192.168.10.50 - 192.168.10.200
```

Addresses outside the pool remain available for other purposes, such as routers, firewalls, switches, access points, servers, printers, management interfaces, or future infrastructure.

Network addresses, broadcast addresses, usable host addresses, and subnet boundaries were covered earlier in [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md) and [IPv4 Subnetting](../../01-fundamentals/05-ipv4-subnetting/ipv4-subnetting.md).

## Addresses Outside the Pool

An address outside the DHCP pool is simply not available for normal dynamic allocation.

For example:

```
Network:    192.168.10.0/24
DHCP pool:  192.168.10.50 - 192.168.10.200
```

The address:

```
192.168.10.48
```

is part of the subnet but outside the DHCP pool.

An administrator could therefore use it for a statically configured device without DHCP attempting to allocate that address from the configured dynamic range.

## DHCP Exclusions

Sometimes an address must remain unavailable for dynamic allocation even though it falls inside the configured DHCP range.

Consider:

```
DHCP range:
192.168.10.50 - 192.168.10.200

Existing equipment:
192.168.10.75  Printer
192.168.10.76  Printer
192.168.10.77  Access Point
```

If DHCP were allowed to dynamically assign `.75`, `.76`, or `.77`, an address conflict could occur.

Instead of redesigning the entire range, the administrator can exclude those addresses.

```
DHCP range:
192.168.10.50 - 192.168.10.200

Exclusion:
192.168.10.75 - 192.168.10.77
```

The effective dynamic allocation becomes:

```
192.168.10.50 - 192.168.10.74

192.168.10.75 - 192.168.10.77
Excluded

192.168.10.78 - 192.168.10.200
```

An address outside the pool was never available to DHCP, while an excluded address is located inside the configured range but is deliberately prevented from being dynamically allocated.

## Multiple Networks Require Separate DHCP Configuration

A centralized DHCP server commonly supports multiple networks.

For example:

```
VLAN 10 - Servers
10.0.10.0/24

VLAN 20 - Employees
10.0.20.0/24

VLAN 30 - Wi-Fi
10.0.30.0/24

VLAN 40 - Guests
10.0.40.0/24
```

Each subnet requires an appropriate DHCP scope or pool.

For example:

```
VLAN 20 - Employees
Network:   10.0.20.0/24
DHCP pool: 10.0.20.50 - 10.0.20.200

VLAN 30 - Wi-Fi
Network:   10.0.30.0/24
DHCP pool: 10.0.30.50 - 10.0.30.230

VLAN 40 - Guests
Network:   10.0.40.0/24
DHCP pool: 10.0.40.10 - 10.0.40.250
```

A client connected to `10.0.20.0/24` must receive configuration appropriate for that network.

If the client incorrectly received an address such as `10.0.40.50/24`, it would be configured for a different IP subnet and would generally be unable to communicate correctly on the employee network.

The relationship between IP addresses and local subnet boundaries was covered in [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md).

## Selecting the Correct Scope

When one DHCP server supports many networks, it must determine which scope should be used for each client request.

For example:

```
Request from VLAN 20
        |
        v
DHCP Server
        |
        v
10.0.20.0/24 scope
        |
        v
Address from 10.0.20.x
```

While:

```
Request from VLAN 40
        |
        v
DHCP Server
        |
        v
10.0.40.0/24 scope
        |
        v
Address from 10.0.40.x
```

When the DHCP server is located in another network, a DHCP relay can provide the server with information that helps identify the originating client network. The role of the relay was introduced in [DHCP Components](../02-dhcp-components/dhcp-components.md) and will be covered in greater detail in the dedicated DHCP relay topic.

## Scope-Specific Network Configuration

Separate scopes are useful for more than selecting the correct IP address.

Different networks can require different network parameters.

For example:

```
VLAN 20 - Employees

Network: 10.0.20.0/24
Gateway: 10.0.20.1
DNS:     10.0.10.10
```

While:

```
VLAN 40 - Guests

Network: 10.0.40.0/24
Gateway: 10.0.40.1
DNS:     1.1.1.1
```

Each network requires an appropriate default gateway.

A client in `10.0.40.0/24` should not receive `10.0.20.1` as its normal default gateway because that address belongs to another subnet.

DHCP can also distribute different DNS servers and other parameters depending on the scope. These settings are covered in more detail in the DHCP options topic.

The role of the default gateway was covered earlier in [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md), while DNS itself was covered in [DNS Fundamentals](../../02-dns/01-dns-fundamentals/dns-fundamentals.md).

## Pool Exhaustion

A DHCP pool contains a limited number of addresses.

Consider:

```
Network:
192.168.10.0/24

DHCP pool:
192.168.10.100 - 192.168.10.150
```

If every address from `.100` through `.150` currently has an active lease, the pool has no available address for another client.

A new device may send DHCPDISCOVER, but the DHCP server cannot successfully offer a free address from that pool.

The client therefore cannot successfully obtain an IPv4 address from that DHCP pool until an address becomes available or the configuration is changed.

## Increasing the Pool

If the subnet still contains unused address space, an administrator may increase the DHCP pool.

For example:

```
Before:

192.168.10.100 - 192.168.10.150
```

could become:

```
After:

192.168.10.100 - 192.168.10.220
```

This provides additional addresses for dynamic clients.

However, the DHCP pool cannot simply grow beyond the boundaries of the subnet.

If the subnet itself no longer provides enough usable addresses, the network design may need to change. This could involve using a larger subnet or creating additional networks.

Subnet sizing was covered in [IPv4 Subnetting](../../01-fundamentals/05-ipv4-subnetting/ipv4-subnetting.md).

## Lease Duration and Pool Exhaustion

Pool utilization is also related to DHCP lease duration.

Consider a guest network where many devices connect temporarily.

If leases remain allocated for a long time after those devices disappear, the DHCP pool can become exhausted even though many previous clients are no longer actively using the network.

Shorter leases can allow addresses to return to the available pool sooner.

The relationship between lease duration, client turnover, and address availability was covered in [DHCP Leases](../04-dhcp-leases/dhcp-leases.md).

## DHCP Capacity Planning

The size of a DHCP pool should reflect the expected number of clients rather than simply using the same range size on every network.

Consider:

```
Employee VLAN

Network:       10.0.20.0/24
DHCP range:    10.0.20.50 - 10.0.20.200
Normal clients: approximately 80
```

Compared with:

```
Guest VLAN

Network:       10.0.40.0/24
DHCP range:    10.0.40.50 - 10.0.40.200
Normal clients: approximately 140
Peak clients:   approximately 220
```

The employee network has comfortable capacity within its existing pool.

The guest network is different. Its expected peak client count is larger than the configured DHCP range can support.

The guest pool should therefore be increased if sufficient addresses exist within the subnet. If the subnet cannot support the expected number of clients, the subnet design itself needs to be reconsidered.

DHCP capacity planning therefore depends on several related factors:

```
Subnet size
     +
DHCP pool size
     +
Expected client count
     +
Lease duration
```

## Enabling and Disabling Scopes

Many DHCP implementations allow a scope or pool to be enabled or disabled.

This gives administrators control over when DHCP begins serving a particular network.

For example, a new VLAN may still be under construction:

```
VLAN 50

Network:    10.0.50.0/24
DHCP range: 10.0.50.50 - 10.0.50.200

DHCP status:
DISABLED
```

An administrator can test the network using a manually configured address:

```
Test device:

IP address: 10.0.50.20
Gateway:    10.0.50.1
```

The administrator can verify the network before allowing DHCP clients to obtain configuration automatically.

Once the network and DHCP configuration are ready, the scope can be enabled.

## Why Disable a Scope?

Disabling a scope can be useful during:

- Initial network deployment
- Testing
- Maintenance
- Troubleshooting
- Configuration changes
- Correction of an incorrectly configured scope

If a scope contains incorrect settings and begins serving clients immediately, multiple devices could receive invalid network configuration.

Temporarily disabling the scope prevents additional dynamic allocations while the administrator investigates or prepares the network.

Disabling the DHCP scope does not necessarily disable the IP network itself. Devices with appropriate static configuration can still communicate normally if the underlying network infrastructure is operational.

## Key Takeaways

- A DHCP pool defines which addresses can be dynamically allocated to clients.
- The DHCP pool does not need to include every usable address in the subnet.
- Addresses outside the pool can be retained for static addressing and infrastructure.
- Exclusions prevent specific addresses inside a configured DHCP range from being dynamically allocated.
- Different subnets require separate DHCP configuration so clients receive addresses appropriate for their network.
- DHCP scopes can provide network-specific settings such as the appropriate default gateway and DNS servers.
- A centralized DHCP server can maintain scopes for many different networks.
- When every address in a pool has an active lease, new clients cannot successfully obtain an address from that pool.
- A pool can be increased only while usable address space remains available inside the subnet.
- DHCP pool size should be planned according to expected client count and peak usage.
- Lease duration affects how quickly unused addresses can return to the available pool.
- Scope capacity planning should consider subnet size, pool size, expected client count, and lease duration together.
- DHCP scopes can be disabled during testing, deployment, maintenance, or troubleshooting without necessarily disabling the underlying network.