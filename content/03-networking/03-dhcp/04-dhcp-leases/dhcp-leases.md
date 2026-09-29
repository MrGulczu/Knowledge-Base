# DHCP Leases

## Overview

The initial DHCP address allocation process was covered in [DHCP DORA Process](../03-dora-process/dora-process.md). After the DHCP server acknowledges a client's request, the client does not permanently own the assigned address. Instead, the address is normally provided for a limited period called a **DHCP lease**.

A lease allows the DHCP server to temporarily assign an address to a client while retaining the ability to return that address to the available pool later.

## Why DHCP Uses Leases

Consider a DHCP pool:

```
192.168.10.100 - 192.168.10.200
```

A laptop receives:

```
IP address: 192.168.10.105
Lease time: 8 hours
```

The laptop may later leave the network, be decommissioned, or never connect again.

If DHCP assignments were permanent, every temporary device could permanently consume an address. Eventually, the DHCP server could run out of addresses even though many of the devices that originally received them were no longer present.

A lease prevents this by limiting how long an assignment remains valid.

Conceptually:

```
192.168.10.105
Assigned to Client A
        |
        v
Valid for a defined lease period
        |
        v
Renewed or eventually returned to the pool
```

## Lease Lifetime

The **lease lifetime** defines how long a client is allowed to use an address before the lease must be renewed or expires.

For example:

```
Address:    192.168.10.105
Lease time: 8 hours
```

The client can continue using the address while the lease remains valid.

DHCP does not normally wait until the final moment before attempting to extend the lease. Renewal begins earlier using lease timers.

## T1 - Renewal

The first important timer is **T1**, also called the renewal timer.

By default, T1 is commonly set to **50% of the lease lifetime** unless different values are provided.

For an 8-hour lease:

```
0h                      4h                      8h
|-----------------------|-----------------------|
Lease begins            T1                  Expiration
                        50%
```

At T1, the client still has a valid address and knows which DHCP server originally granted the lease.

The client therefore normally sends a **unicast DHCPREQUEST** directly to the original DHCP server.

```
Client                                  DHCP Server
192.168.10.105
   |                                         |
   | DHCPREQUEST                             |
   | "Can I continue using this address?"    |
   |---------------------------------------->|
   |                                         |
   |                                 DHCPACK |
   | "Yes. The lease is renewed."            |
   |<----------------------------------------|
```

The use of DHCPREQUEST and DHCPACK during the initial address allocation process was introduced in [DHCP DORA Process](../03-dora-process/dora-process.md).

If the server approves the renewal, the client continues using the same address and the lease timing is refreshed according to the configuration returned by the server.

From the user's perspective, this normally happens without any visible interruption.

## Why T1 Uses Unicast

During the initial DHCP process, broadcast communication is important because the client does not yet have usable IPv4 configuration and does not initially know which DHCP server will provide it.

At T1, the situation is different.

The client already has:

```
A valid IP address
A valid lease
Knowledge of the DHCP server that granted the lease
```

It can therefore contact the original DHCP server directly using unicast.

The differences between broadcast and unicast traffic were covered earlier in [Traffic Types](../../01-fundamentals/11-traffic-types/traffic-types.md).

## When the Original DHCP Server Does Not Respond

Failure to contact the DHCP server at T1 does not immediately cause the client to lose network connectivity.

The existing lease is still valid.

For example:

```
Client
192.168.10.105
     |
     | Unicast DHCPREQUEST
     v
Original DHCP Server
     X
No response
```

The client can continue using `192.168.10.105` while the lease remains valid and continues trying to renew it.

This is an important operational characteristic of DHCP. A temporary DHCP server outage does not necessarily disconnect clients that already hold valid leases.

## T2 - Rebinding

If renewal with the original DHCP server continues to fail, the client eventually reaches the second important timer: **T2**, also called the rebinding timer.

By default, T2 is commonly set to **87.5% of the lease lifetime** unless different values are provided.

For an 8-hour lease:

```
0h                        4h             7h              8h
|-------------------------|--------------|---------------|
Lease starts              T1             T2          Expiration
                          50%            87.5%
```

At T2, the client can no longer rely only on the original DHCP server.

It sends a **broadcast DHCPREQUEST**, allowing another suitable DHCP server to respond.

```
T1 - Renewal

Client
   |
   | Unicast DHCPREQUEST
   v
Original DHCP Server
   X
No response


T2 - Rebinding

Client
   |
   | Broadcast DHCPREQUEST
   v
Available DHCP Servers
```

This gives the DHCP infrastructure another opportunity to extend the client's lease before it expires.

## Renewal and Rebinding Timeline

The lease lifecycle can be represented as:

```
Lease starts
     |
     | Client uses leased configuration
     |
     v
T1 - Renewal
     |
     | Unicast DHCPREQUEST
     | to original DHCP server
     |
     +---- DHCPACK ----> Lease renewed
     |
     | No successful renewal
     v
T2 - Rebinding
     |
     | Broadcast DHCPREQUEST
     | to available DHCP servers
     |
     +---- DHCPACK ----> Lease renewed
     |
     | No successful renewal
     v
Lease expiration
```

The client continues using the existing address throughout T1 and T2 because the lease has not yet expired.

## Lease Expiration

If the client cannot successfully renew or rebind before the lease lifetime ends, the lease expires.

At that point, the client must stop using the leased address.

```
Lease active
     |
     v
T1 - Renewal
     |
     | No successful renewal
     v
T2 - Rebinding
     |
     | No successful renewal
     v
Lease expires
     |
     v
Client stops using the address
     |
     v
DHCP address acquisition begins again
```

The client then needs to obtain valid network configuration again using the DHCP acquisition process described in [DHCP DORA Process](../03-dora-process/dora-process.md).

The reason a client must use addressing appropriate for its current network was covered earlier in [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md).

## DHCPRELEASE

A client does not always need to wait for its lease to expire.

When a client intentionally gives up its DHCP configuration, it can send a **DHCPRELEASE** message to the DHCP server.

For example:

```
Client
192.168.10.105
     |
     | DHCPRELEASE
     v
DHCP Server
     |
     v
Address can be returned
to the available pool
```

This tells the server that the client no longer needs the leased address.

However, DHCPRELEASE cannot always be sent. A device may unexpectedly lose power, disconnect from the network, crash, or otherwise disappear before it can notify the server.

In that situation, the DHCP server cannot simply assume that the address is immediately available. The existing lease remains relevant until it expires or the server can otherwise safely make the address available again.

## Choosing a Lease Duration

There is no single lease duration that is appropriate for every network.

The lease duration should reflect the behavior of clients, the available address space, and operational requirements.

Consider two different environments:

```
Corporate office

- Mostly company-managed devices
- Devices regularly return to the same network
- Relatively stable client population
```

A longer lease, such as several hours or more, may be appropriate.

Compare this with:

```
Guest Wi-Fi

- Large number of temporary clients
- Devices frequently appear and disappear
- Many clients may never return
```

A shorter lease may be more appropriate because addresses used by temporary devices can return to the available pool sooner.

## Shorter Lease Trade-Offs

Short leases allow unused addresses to become available more quickly.

This is useful in networks with high client turnover or limited address space.

However, shorter leases also mean that clients must perform lease renewal more frequently.

Conceptually:

```
Short lease
    |
    +-- Addresses return to pool sooner
    |
    +-- Better for high client turnover
    |
    +-- More frequent DHCP renewal traffic
```

## Longer Lease Trade-Offs

Longer leases reduce how frequently clients need to renew their configuration.

They also allow clients with existing valid leases to continue operating for longer during a DHCP server outage.

However, an address assigned to a client that disappears may remain unavailable for a longer period.

```
Long lease
    |
    +-- Less frequent renewal
    |
    +-- Existing clients tolerate longer DHCP outages
    |
    +-- Unused addresses may remain allocated longer
```

Lease duration is therefore a balance rather than simply choosing the shortest or longest possible value.

## Example Lease Lifecycle

Consider the following client:

```
Address:       192.168.10.105
Lease length:  8 hours
```

Using the common default timer values:

```
0h              4h                       7h              8h
|---------------|------------------------|---------------|
Lease starts    T1                       T2          Expiration
                50%                      87.5%

                Renewal                  Rebinding
                Unicast                  Broadcast
                original server          available servers
```

If the original DHCP server responds successfully at T1:

```
T1
 |
 | DHCPREQUEST
 v
Original DHCP Server
 |
 | DHCPACK
 v
Lease renewed
```

If the original server does not respond:

```
T1
 |
 | Unicast attempts fail
 v
T2
 |
 | Broadcast DHCPREQUEST
 v
Another suitable DHCP server may respond
```

If no DHCP server successfully renews the lease:

```
T2
 |
 | No successful response
 v
Lease expiration
 |
 v
Stop using leased address
 |
 v
Begin DHCP acquisition again
```

## Key Takeaways

- DHCP addresses are normally assigned temporarily through leases rather than permanently.
- Leases allow addresses belonging to devices that leave the network to eventually return to the available pool.
- T1 is the renewal timer and is commonly 50% of the lease lifetime by default.
- At T1, the client normally attempts to renew directly with the original DHCP server using unicast.
- T2 is the rebinding timer and is commonly 87.5% of the lease lifetime by default.
- At T2, the client broadcasts its DHCPREQUEST so another suitable DHCP server can respond.
- A client can continue using its address while the lease remains valid, even if DHCP renewal attempts are temporarily failing.
- When the lease expires, the client must stop using the leased address and obtain valid configuration again.
- DHCPRELEASE allows a client to voluntarily return an address before the lease expires.
- Shorter leases can be useful for networks with high client turnover.
- Longer leases reduce renewal frequency and allow existing clients to operate longer during DHCP server outages.
- Lease duration should be selected according to the behavior and requirements of the network.