# DHCP Redundancy

## Overview

DHCP is often a critical network service. A failure of the only DHCP server may not disconnect every client immediately, but it can prevent new clients from obtaining configuration and eventually affect existing clients when their leases can no longer be renewed.

DHCP redundancy reduces this dependency by providing more than one DHCP server capable of serving the network.

However, redundancy requires more than simply installing two independent DHCP servers. The servers must be designed so they do not create conflicting address allocations, and the surrounding network must allow clients to reach the redundant service.

The DHCP lease lifecycle was covered in [DHCP Leases](../04-dhcp-leases/dhcp-leases.md), while communication with DHCP servers across routed networks was covered in [DHCP Relay](../08-dhcp-relay/dhcp-relay.md).

## What Happens When a DHCP Server Fails?

Consider a network with one centralized DHCP server:

```
                 DHCP Server
                  10.0.10.10
                       |
              -------------------
              |        |        |
           VLAN 20  VLAN 30  VLAN 40
```

If the DHCP server becomes unavailable, different clients are affected differently.

### Clients With Valid Leases

A client that already has a valid DHCP lease does not normally lose its address immediately.

```
DHCP Server
   OFFLINE
      |
      v
Existing client
Valid lease
      |
      v
Continues using its current configuration
```

The client can continue using the leased configuration while the lease remains valid.

### New Clients

A new client still needs to obtain DHCP configuration.

If no DHCP server is available:

```
New client
    |
    | DHCPDISCOVER
    v
No DHCP server response
```

The client cannot successfully obtain a lease from that unavailable DHCP infrastructure.

### Clients Renewing Existing Leases

Existing DHCP clients attempt to renew their leases before expiration.

If the original DHCP server is unavailable, the client can continue using the lease while it remains valid and eventually enters the rebinding stage.

If another appropriate DHCP server is available, it may be able to renew the lease.

If no server can renew the lease before it expires, the client must stop using the expired leased address and begin address acquisition again.

The renewal, rebinding, and expiration process was covered in [DHCP Leases](../04-dhcp-leases/dhcp-leases.md).

## DHCP Failure May Not Be Immediately Visible

A DHCP server failure can initially appear less serious than failures of some other network services.

Existing clients may continue operating normally because they already have valid leases.

Meanwhile:

```
Existing clients
Valid leases
      |
      v
Still working


New clients
      |
      v
Cannot obtain DHCP configuration
```

As more leases require renewal, the impact can increase.

This is one reason DHCP redundancy is important even when a short DHCP outage does not immediately disconnect every device.

## Why Two Independent Servers Can Be Dangerous

A simple approach might be to configure two DHCP servers with the same pool:

```
DHCP Server 1
Pool:
10.0.20.50 - 10.0.20.200


DHCP Server 2
Pool:
10.0.20.50 - 10.0.20.200
```

If the servers operate independently and do not share lease information, they do not know which addresses the other server has already allocated.

For example:

```
DHCP Server 1

10.0.20.75 -> Laptop A
```

Server 2 may still believe:

```
10.0.20.75 -> Available
```

and could allocate:

```
10.0.20.75 -> Laptop B
```

The result is an IP address conflict.

```
Laptop A
10.0.20.75

Laptop B
10.0.20.75

       |
       v

IP address conflict
```

DHCP redundancy therefore needs a mechanism that prevents overlapping allocation decisions.

## Split-Scope Redundancy

One traditional approach is to divide the available DHCP address space between independent servers.

For example:

```
Network:
10.0.20.0/24

DHCP Server 1:
10.0.20.50 - 10.0.20.169

DHCP Server 2:
10.0.20.170 - 10.0.20.220
```

The servers allocate from different ranges.

Because the pools do not overlap, Server 1 cannot normally allocate an address belonging to Server 2's range, and vice versa.

## Split-Scope Example

Conceptually:

```
               10.0.20.0/24
                     |
          -------------------------
          |                       |
          v                       v
   DHCP Server 1            DHCP Server 2

10.0.20.50-.169          10.0.20.170-.220
```

The advantage is simplicity.

The servers do not need to share every lease state to avoid overlapping allocations because their normal allocation ranges are already separated.

## Split-Scope Capacity

The disadvantage is that each server normally controls only part of the available address space.

The split does not need to be exactly 50/50.

For example:

```
Server 1
Approximately 80% of available addresses

Server 2
Approximately 20% of available addresses
```

This is commonly described conceptually as an **80/20 split**.

If the server controlling the larger portion becomes unavailable, the remaining server initially has access only to its own smaller allocation range.

The total subnet may contain sufficient addresses, but the surviving independent server cannot automatically treat every address belonging to the failed server as available.

This makes split-scope simpler but less flexible than coordinated DHCP failover.

## Coordinated DHCP Failover

A more advanced redundancy design allows DHCP servers to exchange lease-state information.

Conceptually:

```
             Lease information
        <----------------------->
DHCP Server 1                 DHCP Server 2
     |                              |
     +---------- Clients -----------+
```

Suppose Server 1 assigns:

```
10.0.20.75 -> Laptop A
```

The lease state is communicated to its failover partner.

Server 2 can therefore know:

```
10.0.20.75
Active lease
Do not allocate to another client
```

This allows the servers to coordinate their use of the DHCP address space.

## Benefits of Lease-State Synchronization

Coordinated failover provides the second server with information about existing leases.

Conceptually:

```
DHCP Server 1                 DHCP Server 2

10.0.20.75 -> Laptop A  <-->  10.0.20.75 -> Laptop A
10.0.20.76 -> Laptop B  <-->  10.0.20.76 -> Laptop B
10.0.20.77 -> Available <-->  10.0.20.77 -> Available
```

If one server becomes unavailable, the surviving server has information about lease state rather than operating completely independently.

This can provide better utilization of the available address space than a simple split-scope design.

The exact allocation behavior depends on the DHCP failover implementation and operating mode. Coordinated servers should not be interpreted as two servers freely allocating every address simultaneously without restrictions.

## Load-Balanced Redundancy

One redundancy model allows both DHCP servers to actively participate in serving clients.

```
             Clients
            /       \
           v         v
    DHCP Server 1   DHCP Server 2
       ACTIVE          ACTIVE
          \             /
           \           /
            Lease state
          synchronization
```

This can distribute DHCP processing between the servers.

Advantages can include:

```
Workload distribution
Service availability
Both servers actively used
Reduced dependency on one active server
```

If one server fails, the surviving server can continue serving clients according to the failover mechanism.

## Active/Standby Redundancy

Another design uses one server primarily for normal DHCP service while the second waits to take over when required.

```
              Clients
                 |
                 v
          DHCP Server 1
              ACTIVE
                 |
        Lease synchronization
                 |
          DHCP Server 2
             STANDBY
```

This model focuses primarily on service availability rather than distributing the normal DHCP workload.

The standby server maintains the information required to take over when the active server becomes unavailable, according to the implementation's failover behavior.

## Vendor Terminology

Different DHCP implementations use different terminology and failover mechanisms.

For example, one platform may describe modes as:

```
Load Balance
Hot Standby
```

while another implementation may use different terms or a different redundancy architecture.

The important concepts are:

```
Are both servers normally serving clients?

or

Is one server primarily waiting to take over?
```

The exact implementation should always be verified for the DHCP platform being deployed.

## Synchronization Failure

A redundancy system must also consider what happens when both DHCP servers remain operational but can no longer communicate with each other.

```
DHCP Server 1 <---- X ----> DHCP Server 2
                   |
                   v
          Synchronization lost
```

This situation is dangerous because each server may continue receiving client requests while no longer receiving current lease information from its partner.

Suppose Server 1 assigns:

```
10.0.20.75 -> Laptop A
```

but Server 2 never receives that information.

If Server 2 were allowed to freely allocate any address from the entire pool indefinitely, it could later assign:

```
10.0.20.75 -> Laptop B
```

creating an address conflict.

## Partner Failure vs Communication Failure

A DHCP server may not immediately know why its partner is unreachable.

From Server 1's perspective:

```
Partner not reachable
```

could mean:

```
DHCP Server 2 has failed
```

or:

```
DHCP Server 2 is still running
but communication between the servers has failed
```

These are very different situations.

If the partner is genuinely offline, allowing the surviving server to take over more responsibility may be desirable.

If the partner is still serving clients but synchronization is broken, allowing both servers unrestricted control of the same addresses could create conflicting leases.

A well-designed failover mechanism therefore needs safeguards for communication failures and inconsistent lease state.

The exact failover states and recovery procedures are implementation-specific.

## DHCP Redundancy and Relay

DHCP server redundancy must also be supported by the network path used by clients.

Consider:

```
VLAN 20
10.0.20.0/24
      |
      v
DHCP Relay
      |
      +--------> DHCP Server 1
      |          10.0.10.10
      |
      +--------> DHCP Server 2
                 10.0.10.11
```

The relay can forward client DHCP requests toward both redundant DHCP servers.

This allows clients on the remote network to reach the available DHCP service.

DHCP relay operation was covered in [DHCP Relay](../08-dhcp-relay/dhcp-relay.md).

## A Redundant Server Must Be Reachable

Suppose the DHCP servers are correctly synchronized:

```
DHCP Server 1
10.0.10.10

DHCP Server 2
10.0.10.11
```

but the relay is configured only with:

```
DHCP destination:
10.0.10.10
```

If Server 1 becomes unavailable, the existence of Server 2 does not automatically help those relayed clients if their DHCP requests are never forwarded to it.

A more complete design would allow the relay to reach the redundant DHCP infrastructure:

```
DHCP Relay
    |
    +----> 10.0.10.10
    |
    +----> 10.0.10.11
```

The exact relay behavior and server configuration depend on the network platform and DHCP redundancy design.

## Redundancy Must Be End-to-End

Reliable DHCP service depends on more than the DHCP servers themselves.

A complete design should consider:

```
DHCP server redundancy
        +
DHCP relay configuration
        +
Routing
        +
Firewall / ACL rules
        +
Network infrastructure
```

Two perfectly configured DHCP servers do not provide useful redundancy to a client network if the clients cannot reach the surviving server.

## Shared Infrastructure Can Defeat Redundancy

Two DHCP server instances do not necessarily provide strong redundancy if they depend on the same physical infrastructure.

Consider:

```
DHCP Server 1
DHCP Server 2
      |
      v
Same hypervisor
Same physical server
Same storage
Same power source
Same network connection
```

A single hypervisor failure could take both DHCP servers offline simultaneously.

The same applies to other shared infrastructure failures.

## Separating Failure Domains

For stronger redundancy, the servers can be placed on separate underlying infrastructure.

For example:

```
DHCP Server 1
    |
    v
Hypervisor 1
Power source / UPS 1
Network path 1


DHCP Server 2
    |
    v
Hypervisor 2
Power source / UPS 2
Network path 2
```

Depending on the required availability level, additional separation may include:

```
Separate physical hosts
Separate storage
Separate switches
Separate power sources
Separate racks
Separate physical locations
```

The required level of separation should match the importance of the service and the organization's acceptable downtime.

A small office and a large datacenter do not necessarily require the same redundancy architecture.

## DHCP Failover vs Infrastructure Redundancy

These concepts solve related but different problems.

```
DHCP failover
        |
        v
Protects against DHCP service
or DHCP server failure
```

while:

```
Infrastructure redundancy
        |
        v
Protects against failures of
the systems underneath DHCP
```

A properly configured DHCP failover pair running on the same failed physical host can still become unavailable.

Similarly, two servers on separate hosts can still create DHCP problems if their lease allocation is not coordinated correctly.

A resilient design should consider both.

## Redundancy and Address Pools

Address-pool design remains important in redundant environments.

Administrators should consider:

```
Subnet size
DHCP pool size
Expected client count
Lease duration
Redundancy method
Failure capacity
```

For example, a split-scope design must ensure that the remaining server has sufficient address capacity during a failure.

A coordinated failover design must ensure that the failover mechanism and lease state remain healthy.

Address-pool planning was covered in [DHCP Scopes and Pools](../05-scopes-and-pools/scopes-and-pools.md).

## Practical Redundancy Example

Consider an organization with:

```
DHCP Server 1:
10.0.10.10

DHCP Server 2:
10.0.10.11

Client networks:
10.0.20.0/24
10.0.30.0/24
10.0.40.0/24
```

The DHCP servers maintain coordinated lease information.

The network relay configuration allows requests to reach both servers:

```
                  DHCP Server 1
                   10.0.10.10
                        ^
                        |
                        |
Client VLANs ---> DHCP Relay
                        |
                        |
                        v
                  DHCP Server 2
                   10.0.10.11
```

The DHCP servers are also placed on separate physical virtualization hosts.

```
DHCP Server 1
      |
Hypervisor 1


DHCP Server 2
      |
Hypervisor 2
```

This design addresses several different failure scenarios:

```
One DHCP service fails
        |
        v
Second server can continue


One hypervisor fails
        |
        v
Second DHCP server remains online


Remote client needs DHCP
        |
        v
Relay can reach redundant servers
```

The exact behavior during each failure still depends on the DHCP implementation and failover state.

## Redundancy Troubleshooting

When a redundant DHCP environment does not behave as expected, useful checks include:

```
Are both DHCP servers running?

Can the servers communicate with each other?

Is lease-state synchronization healthy?

Are the scopes configured for the intended redundancy mechanism?

Are address ranges overlapping incorrectly?

Can clients or relays reach both DHCP servers?

Are relay destinations configured correctly?

Are firewall and ACL rules permitting DHCP traffic?

Does the surviving server have sufficient address capacity?

Are both DHCP servers dependent on the same failed infrastructure?

What failover state does each DHCP server report?
```

A redundancy problem should not automatically be treated as a DHCP server software problem.

The failure can exist in:

```
DHCP configuration
Lease synchronization
Relay configuration
Routing
Firewall policy
Virtualization infrastructure
Physical network
Power infrastructure
```

## Key Takeaways

- A DHCP server failure does not immediately disconnect clients that still have valid leases.
- New clients cannot obtain leases when no appropriate DHCP server is available.
- Existing clients continue attempting lease renewal and rebinding before their leases expire.
- Two independent DHCP servers should not freely allocate from the same overlapping pool without coordination.
- Uncoordinated overlapping pools can result in duplicate IP address assignments.
- Split-scope redundancy divides the available address range between independent DHCP servers.
- Split-scope prevents normal allocation overlap but limits the address capacity available to each individual server.
- Split-scope designs do not need to use an exact 50/50 division.
- Coordinated DHCP failover allows servers to exchange lease-state information.
- Lease synchronization helps redundant servers avoid conflicting allocations.
- Load-balanced designs allow both DHCP servers to actively participate in serving clients.
- Active/standby designs use one server primarily for normal service while another waits to take over.
- Exact failover terminology and behavior depend on the DHCP implementation.
- Loss of synchronization is different from confirmed failure of a DHCP partner.
- Failover mechanisms must prevent both disconnected partners from independently allocating conflicting leases.
- DHCP redundancy must be supported by relay configuration, routing, firewall rules, and the surrounding network.
- A redundant DHCP server is not useful to remote clients if their relay cannot forward requests to it.
- Running two DHCP servers on the same physical host leaves a shared point of failure.
- Stronger redundancy places DHCP servers in separate failure domains where appropriate.
- DHCP failover protects the DHCP service, while infrastructure redundancy protects the systems underneath it.
- Redundancy design should match the organization's availability requirements and acceptable downtime.