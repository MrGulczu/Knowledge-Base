# DHCP Summary

## Overview

DHCP allows network administrators to centrally provide IP configuration to clients instead of manually configuring every device.

Throughout this section we covered:

```
DHCP fundamentals
Scopes and address pools
Lease operation
DHCP options
Reservations and exclusions
DHCP relay
DHCP redundancy
DHCP and DNS
DHCPv6
DHCP security
DHCP troubleshooting
```

This summary combines those concepts into one realistic example environment and follows a workstation from the moment it connects to the network through normal operation, lease renewal, redundancy, security, and troubleshooting.

---

## Example Environment

Consider an organization with several separated networks:

```
VLAN 10 - Servers
10.0.10.0/24

VLAN 20 - Employees
10.0.20.0/24

VLAN 30 - Printers
10.0.30.0/24

VLAN 40 - Guest Wi-Fi
172.20.50.0/24
```

The guest network deliberately uses a different addressing pattern from the internal networks.

This can make administrative separation clearer, but the different addressing pattern should not be treated as a security mechanism by itself.

Actual guest isolation should be enforced through mechanisms such as:

```
VLAN separation
Firewall policy
Access controls
Client isolation where required
```

---

## Central Infrastructure

The organization uses centralized infrastructure in VLAN 10.

```
                     VLAN 10 - Servers
                      10.0.10.0/24

                  DHCP-01       DHCP-02
                 10.0.10.20    10.0.10.21
                      \           /
                       \         /
                        Core / L3
                       /    |     \
                      /     |      \
                     v      v       v

VLAN 20          VLAN 30          VLAN 40
Employees        Printers         Guest Wi-Fi
10.0.20.0/24     10.0.30.0/24    172.20.50.0/24
```

Both DHCP servers use static addresses.

For redundancy:

```
DHCP-01
10.0.10.20
Hypervisor A

DHCP-02
10.0.10.21
Hypervisor B
```

The DHCP servers should not both depend on the same physical server if the objective is to survive a host failure.

Where practical, other shared dependencies should also be considered:

```
Power
Switching
Storage
Virtualization infrastructure
Network paths
```

---

## Why Centralize DHCP?

Instead of deploying a separate DHCP server in every VLAN, the organization keeps the DHCP service centrally.

The gateways for the client networks provide DHCP relay functionality.

Conceptually:

```
Employee Client
      |
      v
VLAN 20
      |
      v
Gateway / DHCP Relay
10.0.20.1
      |
      v
DHCP-01 / DHCP-02
VLAN 10
```

The same model can be used for the printer and guest networks.

This allows the organization to centrally manage:

```
Scopes
Pools
Reservations
Lease information
DHCP options
Redundancy
Monitoring
```

---

## Employee VLAN

The Employee VLAN uses:

```
Network:
10.0.20.0/24

Default Gateway:
10.0.20.1
```

The subnet is divided logically:

```
10.0.20.1
Default gateway

10.0.20.2 - 10.0.20.49
Infrastructure / static assignments

10.0.20.50 - 10.0.20.230
Dynamic DHCP pool

10.0.20.231 - 10.0.20.254
Future / static use
```

Not every usable address needs to belong to the dynamic DHCP pool.

Leaving space outside the pool can simplify static assignments and future changes.

---

## Employee DHCP Options

The employee scope provides more than just an IP address.

Example configuration:

```
IP address:
Selected from 10.0.20.50 - 10.0.20.230

Subnet mask:
255.255.255.0

Default gateway:
10.0.20.1

DNS servers:
10.0.10.10
10.0.10.11

Domain:
corp.example

NTP server:
10.0.10.30

Lease time:
8 hours
```

DHCP can provide only the options that have actually been configured.

Additional DHCP options may be required for specific environments, for example services related to network boot or other specialized clients.

See [DHCP Options](../06-dhcp-options/dhcp-options.md) and the DHCP options reference in the appendix.

---

## Printer VLAN

Printers benefit from predictable addressing.

For the example environment:

```
VLAN 30 - Printers
10.0.30.0/24

Gateway:
10.0.30.1
```

Instead of manually configuring every printer, DHCP reservations can provide predictable addresses while retaining centralized DHCP configuration.

Example:

```
Printer-Finance

MAC:
AA:BB:CC:11:22:33

Reserved address:
10.0.30.10
```

```
Printer-HR

MAC:
AA:BB:CC:44:55:66

Reserved address:
10.0.30.11
```

```
Printer-Reception

MAC:
AA:BB:CC:77:88:99

Reserved address:
10.0.30.12
```

The exact reservation range should depend on the actual number of printers and the organization's addressing plan.

Reserved addresses are not offered as ordinary dynamic leases to unrelated clients.

A reservation does not replace DHCP. The device still uses DHCP, but the DHCP server provides the intended address when the reservation matches the client.

See [DHCP Reservations and Exclusions](../07-reservations-and-exclusions/reservations-and-exclusions.md).

---

## Guest Wi-Fi

Guest devices behave differently from corporate infrastructure.

They frequently connect and disconnect, and predictable per-device addressing is normally unnecessary.

Our example uses:

```
VLAN 40 - Guest Wi-Fi

Network:
172.20.50.0/24

Gateway:
172.20.50.1

Dynamic pool:
172.20.50.10 - 172.20.50.254

Lease:
2 hours

Reservations:
None
```

A relatively large dynamic pool and shorter lease can be useful for a network with many temporary devices.

When a guest leaves, the lease eventually expires and the address can return to the available pool.

---

## A New Employee Workstation Connects

Now consider a new workstation connecting to an Employee VLAN access port.

Initially:

```
Workstation
No usable IPv4 configuration
      |
      v
Access Switch
VLAN 20
      |
      v
Gateway / DHCP Relay
10.0.20.1
      |
      v
DHCP-01 / DHCP-02
```

The workstation needs to discover DHCP service and obtain configuration.

---

## Step 1 - DHCPDISCOVER

The client begins the DHCPv4 DORA process by sending:

```
DHCPDISCOVER
```

Conceptually:

```
Workstation
     |
     | DHCPDISCOVER
     v
VLAN 20
```

At this point the client does not yet have its normal IPv4 configuration.

Because the DHCP servers are in VLAN 10, the client's local broadcast is not simply routed directly to them.

The DHCP relay handles this problem.

---

## Step 2 - DHCP Relay

The VLAN 20 gateway receives the DHCP communication and forwards it toward the centralized DHCP servers.

```
Client
VLAN 20
    |
    | DHCPDISCOVER
    v
Gateway / DHCP Relay
10.0.20.1
    |
    | Relayed DHCP request
    v
DHCP-01 / DHCP-02
```

For DHCPv4, the relay provides information that allows the server to determine the network from which the request originated.

An important field is:

```
giaddr
Gateway IP Address
```

In this example:

```
giaddr:
10.0.20.1
```

The DHCP server can use this information to associate the request with:

```
10.0.20.0/24
```

and therefore select the Employee VLAN scope rather than the Printer or Guest scope.

See [DHCP Relay](../08-dhcp-relay/dhcp-relay.md).

---

## Step 3 - DHCPOFFER

The DHCP server selects an available address from the Employee pool.

For example:

```
10.0.20.105
```

It sends a:

```
DHCPOFFER
```

containing the proposed configuration.

Conceptually:

```
DHCP Server
     |
     | DHCPOFFER
     |
     | 10.0.20.105
     | /24
     | Gateway 10.0.20.1
     | DNS 10.0.10.10
     | DNS 10.0.10.11
     | Domain corp.example
     | NTP 10.0.10.30
     v
Client
```

Depending on the DHCP design, more than one DHCP server may respond.

The client can therefore receive multiple offers and select one.

---

## Step 4 - DHCPREQUEST

After selecting an offer, the client sends:

```
DHCPREQUEST
```

During the initial allocation process, this communicates which offered address and DHCP server the client selected.

Conceptually:

```
Client
   |
   | DHCPREQUEST
   v
DHCP Infrastructure
```

Other DHCP servers that supplied offers can determine that their offers were not selected and make those addresses available again.

The selected server processes the request and determines whether the lease can be confirmed.

---

## Step 5 - DHCPACK

If the selected DHCP server accepts the request, it sends:

```
DHCPACK
```

Conceptually:

```
Selected DHCP Server
        |
        | DHCPACK
        v
Client
        |
        v
Configure interface
```

The workstation can now apply the DHCP configuration.

For example:

```
IPv4:
10.0.20.105

Prefix:
 /24

Gateway:
10.0.20.1

DNS:
10.0.10.10
10.0.10.11

Domain:
corp.example

NTP:
10.0.10.30

Lease:
8 hours
```

The normal DORA process is therefore:

```
DISCOVER
   |
   v
OFFER
   |
   v
REQUEST
   |
   v
ACK
```

See [DHCP DORA Process](../03-dora-process/dora-process.md).

---

## DHCP and DNS

In an integrated environment, DHCP can also participate in dynamic DNS updates.

For example:

```
Employee Laptop
      |
      v
DHCP Lease
10.0.20.105
      |
      v
DHCP / DNS integration
      |
      v
DNS record
```

This helps keep hostname-to-address information aligned with dynamically allocated addresses.

If a lease is removed or expires, the DHCP/DNS integration can also participate in removing outdated records, depending on the configured environment.

DNS remains particularly important in domain environments because systems frequently communicate using names rather than manually entered IP addresses.

See [DHCP and DNS](../10-dhcp-and-dns/dhcp-and-dns.md).

---

## Lease Operation

The workstation does not own `10.0.20.105` permanently.

It receives the address for a defined lease period.

In our example:

```
Lease:
8 hours
```

The client should attempt to renew the lease before it expires.

---

## T1 - Renewal

At approximately 50% of the lease lifetime, the client normally enters the renewal phase.

For an 8-hour lease:

```
Lease begins
     |
     | approximately 4 hours
     v
T1
```

The client normally sends a unicast `DHCPREQUEST` to the DHCP server that granted the lease.

```
Client
10.0.20.105
    |
    | Unicast DHCPREQUEST
    v
DHCP Server
    |
    | DHCPACK
    v
Client
```

If successful, the lease is renewed and the client continues using the address.

The client does not normally wait for the full lease to expire before attempting renewal.

---

## T2 - Rebinding

If the original DHCP server does not respond during renewal, the client can continue using its address while the lease remains valid.

Later, at T2, the client enters rebinding.

Conceptually:

```
Original DHCP Server
        X
   No response
        |
        v
Client continues using
valid lease
        |
        v
T2
        |
        v
Attempt to reach
available DHCP service
```

This is particularly important in redundant DHCP environments.

---

## DHCP Redundancy

Our environment contains:

```
DHCP-01
10.0.10.20
Hypervisor A

DHCP-02
10.0.10.21
Hypervisor B
```

The servers use a properly configured redundancy mechanism and maintain the required lease information.

The relay can forward DHCP communication toward both servers where appropriate.

Now suppose:

```
DHCP-01
    X
Offline

DHCP-02
    |
    v
Online
```

The effect depends on the client's state.

---

## Existing Clients During a DHCP Failure

A client with a valid lease does not immediately lose its IP address just because DHCP-01 becomes unavailable.

```
Client
10.0.20.105

Lease:
Still valid
     |
     v
Continues normal operation
```

DHCP availability becomes important when the client needs to obtain or renew configuration.

---

## Renewing Clients During a DHCP Failure

A client may initially attempt renewal with the DHCP server that originally granted the lease.

If DHCP-01 is unavailable:

```
Client
   |
   | DHCPREQUEST
   v
DHCP-01
   X
```

The client continues using its still-valid lease.

If necessary, it eventually enters rebinding and can communicate with another available DHCP server.

With properly configured redundancy, DHCP-02 has the information required to continue providing DHCP service according to the redundancy design.

---

## New Clients During a DHCP Failure

A new client can still obtain configuration from the surviving DHCP infrastructure.

```
New Client
    |
    | DHCPDISCOVER
    v
Relay
    |
    +----> DHCP-01
    |         X
    |
    +----> DHCP-02
              |
              | DHCPOFFER
              v
           Client
```

This is the purpose of DHCP redundancy: failure of one DHCP server should not automatically remove DHCP service for the organization.

See [DHCP Redundancy](../09-dhcp-redundancy/dhcp-redundancy.md).

---

## DHCP Security

Now suppose someone connects a personal router to an Employee VLAN access port.

```
Employee VLAN 20

Managed Switch
      |
      +--> Corporate Workstation
      |
      +--> Personal Router
              |
              v
        DHCP Server Enabled
```

Without protection, the unauthorized router could attempt to provide DHCP configuration to corporate clients.

It could potentially provide:

```
Incorrect IP configuration
Malicious default gateway
Malicious DNS server
```

This creates risks including loss of connectivity and opportunities for traffic interception or redirection.

---

## DHCP Snooping

The managed switch uses DHCP snooping.

The legitimate path toward DHCP infrastructure is trusted, while normal employee access ports are untrusted.

```
Legitimate DHCP Infrastructure
            |
         TRUSTED
            |
        Managed Switch
         /        \
        /          \
 UNTRUSTED       UNTRUSTED
     |               |
Workstation      Personal Router
```

If the personal router sends server-side DHCP messages such as:

```
DHCPOFFER
DHCPACK
```

through an untrusted interface, DHCP snooping can drop those messages.

```
Personal Router
      |
      | DHCPOFFER
      v
Untrusted Port
      |
      X
Dropped
```

This does not mean the entire physical port must necessarily be disabled.

---

## Why Clients Still Work on Untrusted Ports

An untrusted DHCP snooping port does not mean ordinary DHCP clients cannot use DHCP.

Client-side messages still need to travel toward legitimate DHCP infrastructure.

```
Employee Workstation
      |
      | DHCPDISCOVER
      | DHCPREQUEST
      v
Untrusted Access Port
      |
      v
Allowed toward
DHCP infrastructure
```

The distinction is:

```
Client DHCP messages
        |
        v
Allowed from client access ports


Unauthorized server DHCP messages
        |
        X
Dropped on untrusted ports
```

DHCP snooping can also build a binding table containing relationships such as:

```
IP address
MAC address
VLAN
Switch interface
```

This information can support additional Layer 2 security mechanisms.

See [DHCP Security](../12-dhcp-security/dhcp-security.md).

---

## DHCP Starvation Protection

A malicious or malfunctioning device may generate excessive DHCP requests.

Conceptually:

```
Device
   |
   | DHCP request
   | DHCP request
   | DHCP request
   | DHCP request
   | ...
   v
Switch
```

DHCP rate limiting on untrusted interfaces can help prevent one access port from overwhelming DHCP infrastructure or contributing to DHCP pool exhaustion.

The exact enforcement behavior depends on the switch vendor and configuration.

---

## Troubleshooting Example

An employee reports:

```
"My network isn't working."
```

Their workstation shows:

```
IPv4:
169.254.87.24

Gateway:
None

DNS:
None
```

The APIPA address indicates that the workstation did not successfully obtain a normal DHCP lease.

However, before changing the DHCP server, check another workstation in the same VLAN.

---

## Compare Another Client

Another VLAN 20 workstation has:

```
IPv4:
10.0.20.108

Gateway:
10.0.20.1

DNS:
10.0.10.10
```

This is important evidence.

It shows that DHCP is successfully serving at least another device in the same VLAN.

Therefore:

```
Employee DHCP scope
        |
        v
Probably operational

DHCP relay
        |
        v
Probably operational

Path to DHCP infrastructure
        |
        v
Probably operational
```

The investigation should begin closer to the affected workstation.

---

## Troubleshoot the Affected Workstation

A useful sequence is:

```
Affected Workstation
        |
        v
NIC configured for DHCP?
        |
        v
Physical / Wi-Fi connection working?
        |
        v
Correct switch port or SSID?
        |
        v
Correct VLAN assignment?
        |
        v
Client sending DHCPDISCOVER?
```

If required, continue with:

```
Switch information
DHCP server logs
Packet captures
Security software
DHCP snooping state
```

The important principle is to follow the evidence instead of immediately restarting or reconfiguring shared infrastructure.

See [DHCP Troubleshooting](../13-dhcp-troubleshooting/dhcp-troubleshooting.md).

---

## End-to-End DHCP Flow

The complete employee workstation example can now be represented as:

```
Workstation connects
        |
        v
No IPv4 configuration
        |
        v
DHCPDISCOVER
        |
        v
Employee VLAN 20
        |
        v
DHCP Relay
10.0.20.1
        |
        | giaddr identifies
        | client network
        v
DHCP-01 / DHCP-02
        |
        v
Employee scope selected
        |
        v
Available address selected
10.0.20.105
        |
        v
DHCPOFFER
        |
        v
Client selects offer
        |
        v
DHCPREQUEST
        |
        v
DHCPACK
        |
        v
Client configures:
IP
Prefix
Gateway
DNS
Domain
NTP
Lease
        |
        v
Normal network operation
        |
        v
T1 renewal
        |
        v
Lease renewed
```

If the original DHCP server becomes unavailable:

```
T1 renewal fails
        |
        v
Client keeps valid lease
        |
        v
T2 rebinding
        |
        v
Available redundant DHCP server
```

---

## DHCPv4 and DHCPv6

The example above primarily describes DHCPv4.

IPv6 configuration works differently.

An IPv6 host may use:

```
SLAAC

SLAAC + Stateless DHCPv6

Stateful DHCPv6
```

Router Advertisements remain important because IPv6 hosts learn default-router information through RA rather than through a DHCPv6 default-gateway option.

A dual-stack client can therefore use:

```
IPv4
DHCPv4

and simultaneously

IPv6
SLAAC / DHCPv6
```

The two configuration processes are independent.

See [DHCPv6](../11-dhcpv6/dhcpv6.md).

---

## Choosing the Appropriate DHCP Method

Different device types can use different addressing approaches.

A practical example is:

```
Employee workstations
        |
        v
Dynamic DHCP


IP phones
        |
        v
Dynamic DHCP
+ required DHCP options


Guest devices
        |
        v
Dynamic DHCP
Shorter leases


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
Static addressing
or reservations
depending on design
```

The correct choice depends on how predictable the address must be and whether centralized DHCP configuration is useful for the device.

---

## Final Troubleshooting Model

When DHCP fails, follow the communication path.

```
1. Is the client configured for DHCP?

2. Is the client connected to the correct network?

3. Are other clients in the same VLAN working?

4. Is the switch port / SSID in the correct VLAN?

5. Does the client send DHCPDISCOVER?

6. Does the relay receive the request?

7. Does the DHCP server receive the request?

8. Does the correct scope exist?

9. Is the scope enabled?

10. Are addresses available?

11. Does the server generate an offer?

12. Does the offer return through the relay?

13. Does the client receive and accept it?

14. Are the supplied gateway, DNS and other options correct?
```

A particularly useful question is:

```
What is the last point where I can prove
that the expected DHCP message exists?
```

This helps identify where troubleshooting should continue.

---

## Key Takeaways

- DHCP centrally provides IP configuration to network clients.
- A DHCP scope represents configuration for a particular network.
- A dynamic pool does not need to consume every usable address in the subnet.
- Addresses can be left outside the dynamic pool for infrastructure, static assignments, and future requirements.
- DHCP can provide gateway, DNS, domain, NTP, and other configured options in addition to the IP address.
- Different networks can use different lease times and pool designs according to their purpose.
- Guest networks often benefit from larger dynamic pools and shorter leases.
- DHCP reservations provide predictable addresses while the client continues using DHCP.
- Reservations are useful for devices such as printers and access points when predictable addressing is required.
- Centralized DHCP servers can serve multiple routed networks through DHCP relay.
- DHCPv4 relay information allows the server to determine which client network and scope should be used.
- DHCPv4 normally follows DISCOVER, OFFER, REQUEST, and ACK.
- A client may receive offers from multiple DHCP servers and select one.
- DHCPACK confirms that the client can apply the leased configuration.
- DHCP leases are temporary and should be renewed before expiration.
- At T1, the client normally attempts unicast renewal with the server that granted the lease.
- If renewal fails, the client continues using its valid lease and can later enter T2 rebinding.
- Redundant DHCP servers should avoid unnecessary shared failure points.
- Existing clients do not immediately lose valid leases when a DHCP server fails.
- A surviving redundant DHCP server can continue serving new clients and participate in renewal/rebinding according to the redundancy design.
- DHCP and DNS integration can help keep dynamic hostname records aligned with DHCP address assignments.
- Rogue DHCP servers can provide malicious gateways, DNS servers, or otherwise incorrect network configuration.
- DHCP snooping can prevent DHCP server messages from being accepted through untrusted client-facing interfaces.
- Untrusted DHCP snooping ports still allow legitimate client-side DHCP communication.
- DHCP rate limiting can help protect against excessive DHCP traffic.
- An APIPA address indicates that a Windows client did not successfully obtain a normal DHCP lease.
- If another client in the same VLAN receives DHCP correctly, begin troubleshooting closer to the affected client and access network.
- DHCP troubleshooting should follow the actual packet path and use logs or captures to determine where communication stops.
- DHCPv4 and DHCPv6 are separate configuration processes and can use different mechanisms on the same dual-stack client.