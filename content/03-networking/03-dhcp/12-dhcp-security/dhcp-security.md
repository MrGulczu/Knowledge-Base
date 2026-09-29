# DHCP Security

## Overview

DHCP is designed to provide network configuration to clients that may not yet have a usable IP configuration.

This makes DHCP convenient, but it also creates security concerns. A client looking for DHCP service does not automatically know which responding DHCP server belongs to the organization.

An unauthorized DHCP server can therefore attempt to provide malicious network configuration, while an attacker can also attempt to exhaust the legitimate DHCP address pool.

This topic focuses on:

```
Rogue DHCP servers
Malicious DHCP configuration
DHCP starvation
DHCP snooping
Trusted and untrusted interfaces
DHCP snooping binding tables
DHCP rate limiting
Static-address limitations
```

The normal DHCP message exchange was covered in [DHCP DORA Process](../03-dora-process/dora-process.md), while DHCP options were covered in [DHCP Options](../06-dhcp-options/dhcp-options.md).

---

## Rogue DHCP Servers

A **rogue DHCP server** is an unauthorized DHCP server operating on a network.

It may be connected intentionally by an attacker or accidentally by a user.

For example, an employee might connect a small router to an office network while its built-in DHCP server is still enabled.

Consider:

```
Employee VLAN
10.0.20.0/24

Legitimate DHCP Server
        |
        +--> Gateway: 10.0.20.1
        +--> DNS:     10.0.10.10
```

A rogue device is then connected:

```
                    DHCPDISCOVER
                         |
                         v
                       Client
                      /      \
                     /        \
                    v          v
          Legitimate DHCP   Rogue DHCP
              Server           Server
```

Both servers may attempt to provide DHCP configuration to the client.

---

## Why a Rogue DHCP Server Is Dangerous

If the client accepts configuration from the rogue server, the attacker may control important parts of the client's network configuration.

For example:

```
IP address:      10.0.20.150
Default gateway: 10.0.20.200
DNS server:      10.0.20.200
```

If `10.0.20.200` belongs to an attacker-controlled device, the client may begin sending traffic through infrastructure controlled by the attacker.

---

## Malicious Default Gateway

Controlling the client's default gateway can allow an attacker to place an attacker-controlled system in the path of traffic leaving the local network.

Conceptually:

```
Client
10.0.20.150
    |
    | Default gateway:
    | 10.0.20.200
    v
Attacker
10.0.20.200
    |
    | Forwards traffic
    v
Real Gateway
10.0.20.1
    |
    v
Internet / Other Networks
```

This can create an opportunity for a **Man-in-the-Middle (MITM)** attack.

Depending on the protocols and protections used by the traffic, an attacker positioned in the communication path may attempt to:

```
Observe traffic
Redirect traffic
Manipulate traffic
Disrupt communication
```

Higher-layer protections such as authenticated encryption can still protect the content and integrity of properly secured communication.

---

## Malicious DNS Configuration

A rogue DHCP server can also provide an attacker-controlled DNS server.

For example:

```
Client
   |
   | DNS query
   v
Attacker-controlled DNS
   |
   | Attacker-selected answer
   v
Client
```

This can allow the attacker to attempt to redirect hostname-based communication toward addresses selected by the attacker.

The relationship between DHCP and DNS was covered in [DHCP and DNS](../10-dhcp-and-dns/dhcp-and-dns.md).

---

## Rogue DHCP as Denial of Service

A rogue DHCP server does not need to perform a sophisticated interception attack to cause damage.

It can simply provide unusable network configuration.

For example:

```
Incorrect gateway
Incorrect DNS server
Incorrect subnet information
Unusable IP configuration
```

The client may successfully complete DHCP but still be unable to communicate correctly.

This is an important troubleshooting distinction:

```
DHCP lease received
```

does not automatically mean:

```
DHCP configuration is legitimate or correct
```

---

## Why Can a Rogue Server Participate?

During normal DHCP discovery, a client searches for available DHCP service.

Conceptually:

```
Client
   |
   | DHCPDISCOVER
   v
   +--------------------+
   |                    |
   v                    v
Legitimate DHCP      Rogue DHCP
   |                    |
   | DHCPOFFER          | DHCPOFFER
   +---------> Client <-+
```

Basic DHCPv4 does not inherently prove to the client that a responding DHCP server is the organization's authorized server.

Client offer-selection behavior can vary, so it should not be assumed that every client always chooses the first offer received.

The important security problem is that an unauthorized server can participate in the exchange and potentially have its configuration selected.

This is why protection is commonly implemented in the network infrastructure.

---

## DHCP Snooping

**DHCP snooping** is a Layer 2 security feature available on many managed switches.

It allows the switch to distinguish between interfaces from which DHCP server messages are expected and interfaces where DHCP server messages should not be accepted.

Interfaces are commonly classified as:

```
Trusted

or

Untrusted
```

The exact commands and implementation depend on the switch vendor.

---

## Trusted Interfaces

A trusted DHCP snooping interface is an interface from which legitimate DHCP server-side messages are expected.

For example:

```
DHCP Server
     |
     |
Switch
     |
Trusted interface
```

A trusted interface may be directly connected to the DHCP server or may be an uplink forming part of the legitimate path toward DHCP infrastructure.

---

## Untrusted Interfaces

Ordinary client-facing access interfaces are normally treated as untrusted for DHCP server traffic.

For example:

```
Managed Switch
   /    |    \
  /     |     \
PC1    PC2    PC3
```

Conceptually:

```
PC access ports
      |
      v
Untrusted
```

This does not mean normal DHCP client communication is prohibited.

Clients still need to send messages such as:

```
DHCPDISCOVER
DHCPREQUEST
```

The trusted/untrusted distinction primarily controls where DHCP **server-side** messages are permitted to enter the switched network.

---

## Blocking Rogue DHCP Server Messages

Suppose an unauthorized router is connected to an untrusted employee port.

```
Rogue DHCP Server
        |
        | DHCPOFFER
        | DHCPACK
        v
Untrusted Switch Port
        |
        X
```

DHCP snooping can drop DHCP server messages arriving from the untrusted interface.

Compare this with the legitimate path:

```
Legitimate DHCP Server
        |
        | DHCPOFFER
        | DHCPACK
        v
Trusted Interface
        |
        v
Allowed
```

This prevents ordinary access ports from being used as unauthorized DHCP server locations.

---

## DHCP Snooping Does Not Necessarily Disable the Port

It is important to distinguish between:

```
Dropping unauthorized DHCP messages
```

and:

```
Physically or administratively disabling
the entire switch interface
```

DHCP snooping primarily filters inappropriate DHCP server traffic received on untrusted interfaces.

A switch may support additional enforcement actions depending on the platform and configuration, but DHCP snooping should not automatically be described as shutting down every interface that sends an invalid DHCP message.

---

## Trusted Ports Across Multiple Switches

Consider:

```
                DHCP Server
                     |
                  Switch A
                     |
                   Trunk
                     |
                  Switch B
                 /   |   \
                /    |    \
              PC1   PC2   PC3
```

DHCP snooping policy is enforced on the managed switches.

The legitimate path carrying DHCP server responses must be treated appropriately.

Conceptually:

```
                DHCP Server
                     |
                  Switch A
                 [TRUSTED]
                     |
                   Trunk
                     |
                 [TRUSTED]
                  Switch B
                 /   |   \
                /    |    \
          UNTRUSTED UNTRUSTED UNTRUSTED
             |        |        |
            PC1      PC2      PC3
```

The exact trusted-interface configuration depends on the topology and vendor implementation.

The main principle is:

```
Legitimate DHCP server/uplink path
        |
        v
Trusted where required


Ordinary client access ports
        |
        v
Untrusted
```

Administrators should avoid marking unnecessary access interfaces as trusted.

---

## DHCP Snooping Binding Table

DHCP snooping can also learn information from successful DHCP assignments.

The switch can build a **DHCP snooping binding table**.

For example:

```
MAC Address          IP Address       VLAN    Interface
AA:AA:AA:11:22:33    10.0.20.105      20      Gi1/0/15
BB:BB:BB:44:55:66    10.0.20.106      20      Gi1/0/16
```

This provides a relationship between:

```
MAC address
     +
IP address
     +
VLAN
     +
Switch interface
```

For example:

```
10.0.20.105
belongs to
AA:AA:AA:11:22:33
on VLAN 20
through Gi1/0/15
```

This information can be useful to additional Layer 2 security mechanisms.

---

## DHCP Snooping and Dynamic ARP Inspection

One important feature that can use DHCP snooping bindings is **Dynamic ARP Inspection (DAI)**.

Suppose the binding table contains:

```
IP:
10.0.20.105

MAC:
AA:AA:AA:11:22:33

VLAN:
20

Interface:
Gi1/0/15
```

Another device then claims that the same IP belongs to:

```
MAC:
CC:CC:CC:77:88:99
```

The DHCP snooping database provides information that a supporting security mechanism can use to determine that the claim does not match the learned DHCP binding.

Conceptually:

```
Claimed mapping
        |
        v
Compare with trusted binding information
        |
        v
Valid or invalid?
```

Dynamic ARP Inspection is a separate Layer 2 security mechanism, so its detailed operation belongs in a switching or security topic rather than DHCP itself.

The relationship is important because the DHCP snooping database can provide the trusted address-binding information used by DAI.

---

## DHCP Starvation

Another DHCP-related threat is **DHCP starvation**.

In this attack, an attacker attempts to consume the available DHCP address pool by generating large numbers of DHCP requests using many client identities.

Conceptually:

```
Attacker
   |
   +--> Client identity A -> requests address
   +--> Client identity B -> requests address
   +--> Client identity C -> requests address
   +--> Client identity D -> requests address
   +--> ...
```

If enough leases are consumed:

```
DHCP Pool

10.0.20.50 - 10.0.20.200

Available addresses:
0
```

The legitimate DHCP server no longer has a free address to provide to a new client.

DHCP pools and address availability were covered in [DHCP Scopes and Pools](../05-scopes-and-pools/scopes-and-pools.md).

---

## Effect of DHCP Starvation

A legitimate client connects:

```
Legitimate Client
       |
       | DHCPDISCOVER
       v
DHCP Server
       |
       X
No available address
```

The client cannot obtain a normal DHCP lease from the exhausted pool.

The primary result is therefore a **denial of service** against new DHCP clients.

Existing clients with valid leases may continue using their addresses, so DHCP starvation does not necessarily disconnect every device immediately.

This resembles some aspects of a DHCP server outage, which was discussed in [DHCP Redundancy](../09-dhcp-redundancy/dhcp-redundancy.md).

---

## Rogue DHCP and Starvation Together

A DHCP starvation attack and a rogue DHCP server can potentially complement each other.

Conceptually:

```
Legitimate DHCP pool
        |
        v
Exhausted
        |
        v
Legitimate server cannot
provide new addresses
```

while:

```
Rogue DHCP server
        |
        v
Attempts to provide
unauthorized configuration
```

The security goal is therefore both to protect legitimate DHCP availability and to prevent unauthorized DHCP servers from supplying configuration.

---

## DHCP Rate Limiting

Managed switches can provide **DHCP rate limiting** on untrusted interfaces.

The purpose is to prevent a single access port from generating an excessive amount of DHCP traffic.

Normal behavior might look like:

```
Employee PC
     |
     | Normal DHCP traffic
     v
Access Port
     |
     v
Allowed
```

Abnormal behavior might look like:

```
Suspicious Client
     |
     | DHCP message
     | DHCP message
     | DHCP message
     | DHCP message
     | ...
     v
Access Port
     |
     v
Configured DHCP rate exceeded
```

The switch can then enforce the configured protection.

---

## Rate-Limit Enforcement

The exact action taken when a DHCP rate limit is exceeded depends on the switch implementation and configuration.

Possible behavior can include:

```
Dropping excessive DHCP traffic

Restricting the interface

Placing the interface into an
error-disabled or similar state
```

Therefore, it should not be assumed that every switch automatically disables the physical interface when the configured DHCP rate is exceeded.

Administrators should verify the behavior of the specific platform.

---

## DHCP Snooping and Static IP Addresses

DHCP snooping learns dynamic bindings by observing DHCP exchanges.

A device using a manually configured static IP address does not obtain its address through DHCP.

For example:

```
Server-01

Static IP:
10.0.20.50

MAC:
AA:BB:CC:11:22:33
```

The switch can still learn the source MAC address through normal Ethernet switching.

Its MAC address table may contain:

```
MAC Address:

AA:BB:CC:11:22:33

Interface:

Gi1/0/15
```

However, this is not the same as a dynamically learned DHCP snooping binding.

---

## MAC Table vs DHCP Snooping Binding

The normal MAC address table primarily associates:

```
MAC address
     |
     v
Switch interface
```

A DHCP snooping binding can associate:

```
MAC address
     +
IP address
     +
VLAN
     +
Switch interface
```

A statically addressed device that never participates in DHCP does not automatically create this dynamic DHCP snooping binding.

This matters when another security feature depends on the DHCP snooping database.

---

## Static Devices and Dependent Security Features

If Dynamic ARP Inspection or another mechanism relies on DHCP snooping bindings, statically addressed systems require consideration.

Depending on the platform and design, administrators may need mechanisms such as:

```
Static bindings

ARP ACLs

Other vendor-specific configuration
```

to allow legitimate statically addressed systems while maintaining the intended security controls.

The exact method depends on the switching platform.

---

## Security Example

Consider an employee VLAN protected by a managed switch:

```
Employee VLAN
      |
      v
Managed Switch
      |
      +--> Legitimate clients
      |
      +--> Attacker-controlled router
```

DHCP snooping is enabled.

The legitimate DHCP infrastructure is reachable through trusted interfaces, while ordinary employee access ports are untrusted.

The attacker's device attempts two actions.

### Rogue DHCP Response

```
Attacker
   |
   | DHCPOFFER
   v
Untrusted Interface
   |
   X
DHCP server message dropped
```

The trusted/untrusted DHCP snooping policy prevents the unauthorized server response from being accepted through the client-facing interface.

### Excessive DHCP Requests

The same device generates an abnormal amount of DHCP client traffic:

```
Attacker
   |
   | Excessive DHCP requests
   v
Untrusted Interface
   |
   v
DHCP rate limit exceeded
```

The configured rate-limiting policy can then restrict or drop the excessive DHCP traffic according to the switch implementation.

These mechanisms address different behaviors:

```
Rogue DHCP server traffic
        |
        v
DHCP snooping
Trusted / untrusted interfaces


Excessive DHCP client traffic
        |
        v
DHCP rate limiting
```

---

## DHCP Security Troubleshooting

When investigating suspicious or unexpected DHCP behavior, useful checks include:

```
Which DHCP server provided the lease?

Is the DHCP server authorized?

What default gateway did the client receive?

What DNS servers did the client receive?

Are DHCP snooping policies enabled on the correct VLANs?

Which interfaces are trusted?

Are any client-facing interfaces incorrectly trusted?

Are legitimate server-facing or uplink interfaces trusted where required?

Is the DHCP snooping binding table being populated?

Is the DHCP pool unexpectedly exhausted?

Is one switch port generating excessive DHCP traffic?

Are DHCP rate limits configured?

Has an interface entered an error-disabled or restricted state?

Are static-address devices affected by security features that depend on DHCP snooping bindings?
```

If clients suddenly receive unexpected gateways or DNS servers, administrators should verify which DHCP server supplied the configuration rather than assuming that the legitimate DHCP server is misconfigured.

---

## Security Design Considerations

A practical DHCP security design can combine several controls:

```
Managed switches
        |
        +--> DHCP snooping
        |
        +--> Trusted DHCP infrastructure paths
        |
        +--> Untrusted client access ports
        |
        +--> DHCP rate limiting
        |
        +--> Binding-table monitoring
```

Additional Layer 2 security mechanisms can use the DHCP snooping database where appropriate.

The design should also account for:

```
Static-address systems
DHCP relay paths
Redundant DHCP servers
Switch uplinks
Trunk links
Network-management requirements
```

DHCP relay was covered in [DHCP Relay](../08-dhcp-relay/dhcp-relay.md), while redundant DHCP infrastructure was covered in [DHCP Redundancy](../09-dhcp-redundancy/dhcp-redundancy.md).

---

## Key Takeaways

- DHCP clients do not inherently know that every responding DHCP server is authorized by the organization.
- A rogue DHCP server can provide malicious or incorrect network configuration.
- A malicious default gateway can place attacker-controlled infrastructure in the path of client traffic.
- A malicious DNS server can attempt to redirect hostname-based communication.
- Rogue DHCP configuration can also cause a simple denial of service by providing unusable settings.
- Client DHCP offer-selection behavior can vary, so a rogue server should not be described as always winning simply because it responds first.
- DHCP snooping is a Layer 2 security feature available on many managed switches.
- DHCP snooping distinguishes trusted and untrusted interfaces for DHCP server traffic.
- Legitimate server-facing and required uplink paths can be configured as trusted.
- Ordinary client-facing access ports should normally remain untrusted for DHCP server messages.
- DHCP server messages received on untrusted interfaces can be dropped.
- DHCP snooping does not necessarily disable the entire physical port when an unauthorized DHCP message is detected.
- Normal DHCP client messages must still be allowed from untrusted client ports.
- DHCP snooping can build bindings containing IP address, MAC address, VLAN, and interface information.
- The DHCP snooping binding table can support other Layer 2 security mechanisms such as Dynamic ARP Inspection.
- DHCP starvation attempts to exhaust the available DHCP address pool.
- An exhausted DHCP pool prevents new legitimate clients from obtaining normal leases.
- Existing clients with valid leases may continue operating during a starvation attack.
- DHCP rate limiting can restrict excessive DHCP traffic from untrusted access ports.
- The exact response to exceeding a DHCP rate limit depends on the switch vendor and configuration.
- A statically addressed device does not automatically create a dynamic DHCP snooping binding.
- A switch may still learn the MAC address and physical interface of a static device through its normal MAC address table.
- Security mechanisms that depend on DHCP snooping bindings may require additional configuration for statically addressed systems.
- DHCP security should be designed together with relay configuration, redundancy, switching topology, and the organization's Layer 2 security controls.