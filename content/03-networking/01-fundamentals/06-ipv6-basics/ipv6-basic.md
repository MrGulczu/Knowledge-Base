# IPv6 Basics

## Overview

IPv6 is the current version of the Internet Protocol designed to provide a much larger addressing space than IPv4.

One of the main reasons for introducing IPv6 was the limited size of the IPv4 address space.

IPv4 uses:

```
32 bits
```

IPv6 uses:

```
128 bits
```

This creates an enormous increase in the number of available addresses.

IPv6 also introduces several differences in how networks operate.

For example:

- IPv6 addresses are written using hexadecimal notation, 
- IPv6 does not use broadcast,
- normal IPv6 LANs generally use `/64` prefixes,
- hosts can automatically configure addresses using SLAAC,
- Neighbor Discovery replaces ARP functionality,
- routers do not fragment IPv6 packets in transit,
- the IPv6 base header does not contain an IPv4-style header checksum.

IPv6 has not completely replaced IPv4.

Many environments still primarily use IPv4, while others operate both IPv4 and IPv6 simultaneously.

Understanding IPv6 is therefore important even when working in networks where IPv4 remains dominant.

---

## Why IPv6 Exists

IPv4 provides:

```
2^32
```

possible addresses.

This represents approximately 4.3 billion addresses.

Although this originally appeared to provide a very large addressing space, the rapid growth of:

- Internet-connected computers,
- smartphones,
- cloud infrastructure,
- servers,
- IoT devices,
- home networks,
- enterprise networks

created much greater demand for IP addresses.

Technologies such as:

```
Private IPv4 addressing
NAT
PAT
```

helped extend the useful life of IPv4.

However, they did not increase the actual IPv4 address space.

IPv6 solves this problem by using 128-bit addresses.

The theoretical IPv6 address space contains:

```
2^128
```

possible addresses.

The IPv6 addressing philosophy is therefore different from IPv4.

Instead of attempting to conserve every individual address, networks can use large, predictable address blocks and focus more on logical organization and subnet design.

---

## IPv6 Address Structure

An IPv6 address contains:

```
128 bits
```

The address is divided into:

```
8 groups
```

with each group containing:

```
16 bits
```

Therefore:

```
8 × 16 bits = 128 bits
```

An IPv6 address may look like:

```
2001:0db8:1234:5678:abcd:0000:0000:0001
```

Each group contains four hexadecimal digits.

Hexadecimal uses:

```
0 1 2 3 4 5 6 7 8 9 A B C D E F
```

Each hexadecimal digit represents four bits.

Therefore:

```
4 hexadecimal digits × 4 bits = 16 bits
```

IPv6 addresses are usually more difficult for humans to read and memorize than IPv4 addresses.

However, hexadecimal allows the much larger 128-bit address to be represented much more compactly than binary notation.

---

## Shortening IPv6 Addresses

IPv6 addresses can be shortened using two important rules.

### Removing Leading Zeros

Leading zeros inside each 16-bit block can be removed.

For example:

```
0db8 → db8
0001 → 1
0010 → 10
0000 → 0
```

Only leading zeros can be removed.

For example:

```
1000
```

cannot become:

```
1
```

because the remaining zeros are part of the actual value.

### Compressing Zero Blocks

A sequence of consecutive blocks containing only zeros can be replaced with:

```
::
```

For example:

```
2001:0db8:0000:0000:0000:0000:0000:0010
```

can be shortened to:

```
2001:db8::10
```

The double colon can only be used once inside an IPv6 address.

Otherwise, it would be impossible to determine how many zero blocks each `::` represents.

Consider:

```
2001:0db8:0000:0000:0001:0000:0000:0010
```

It could be written as:

```
2001:db8::1:0:0:10
```

or:

```
2001:db8:0:0:1::10
```

Both represent the same address.

When producing a standardized textual representation, the longest sequence of zero blocks is normally compressed.

If multiple equal-length sequences exist, the first one is normally selected.

---

## IPv6 Prefix Length

IPv6 uses CIDR prefix notation similarly to IPv4.

For example:

```
2001:db8:1234:10::/64
```

The `/64` means that the first 64 bits belong to the network prefix.

The remaining 64 bits belong to the interface portion.

Conceptually:

```
|----------- 64 bits -----------|----------- 64 bits -----------|
|        Network Prefix         |       Interface Identifier    |
```

A typical IPv6 network prefix may itself contain multiple logical parts.

For example, an organization may receive:

```
2001:db8:1234::/48
```

and use the next 16 bits for internal subnet identification.

One possible subnet could be:

```
2001:db8:1234:0010::/64
```

Conceptually:

```
2001:db8:1234 | 0010 | Interface Identifier
 Global Prefix  Subnet ID
```

The prefix assigned to an organization does not always have to be `/48`.

Depending on the environment and provider, prefixes such as `/56` may also be allocated.

---

## IPv6 Address Types

IPv6 uses several important types of addresses.

### Global Unicast

A Global Unicast Address is used for normal one-to-one communication and can be globally routable.

Global Unicast addressing is roughly comparable in purpose to public IPv4 addressing.

Globally allocated IPv6 unicast addresses are primarily associated with:

```
2000::/3
```

For example:

```
2001:db8:1234:10::25
```

The prefix:

```
2001:db8::/32
```

is reserved specifically for documentation and examples.

It should not be used as real production Internet addressing.

### Link-Local

Link-local addresses are used for communication on the local network link.

They use:

```
fe80::/10
```

A link-local address is not normally routed between different networks.

It can be used by multiple devices communicating inside the same local network or VLAN.

IPv6 interfaces commonly generate a link-local address automatically.

This can happen even when the same interface also has a Global Unicast or Unique Local address.

Link-local addresses are important for several IPv6 mechanisms, including communication with local routers and Neighbor Discovery.

### Unique Local Address

Unique Local Addresses, or ULAs, are intended primarily for internal IPv6 communication.

The ULA range is:

```
fc00::/7
```

Locally generated ULAs normally use:

```
fd00::/8
```

Unique Local Addresses are not intended to be routed across the public Internet.

Their purpose can be loosely compared with RFC1918 private IPv4 addressing.

However, IPv6 addressing should not simply be treated as a direct copy of IPv4 private addressing.

### Multicast

Multicast represents communication from one sender to a selected group of receivers.

IPv6 multicast addresses use:

```
ff00::/8
```

IPv6 uses multicast extensively for protocol operations.

One important difference from IPv4 is that IPv6 does not use broadcast.

Functions that may use broadcast in IPv4 are often implemented using multicast or other mechanisms in IPv6.

### Anycast

Anycast allows the same unicast address to exist on multiple systems or interfaces.

Routing determines which instance receives the traffic.

Conceptually:

```
             Server A
            /
Client ---- Routing
            \
             Server B
```

Both servers may use the same destination address.

The routing topology determines which server receives the connection.

Anycast does not use a special visual address format.

It uses normal unicast addressing.

---

## IPv6 Does Not Use Broadcast

IPv4 supports broadcast communication.

For example, an IPv4 subnet normally contains a broadcast address representing all hosts inside the subnet.

IPv6 does not use broadcast.

Instead, IPv6 uses multicast where communication with groups of systems is required.

This also means that IPv6 subnetting does not reserve a final address as a broadcast address.

The traditional IPv4 structure:

```
Network Address
Host Range
Broadcast Address
```

does not apply to IPv6 in the same way.

---

## Important Special IPv6 Addresses

### Unspecified Address

The unspecified IPv6 address is:

```
::/128
```

Usually written simply as:

```
::
```

It represents the absence of a specific IPv6 address.

It may appear temporarily while a system is configuring its networking.

### Loopback

The IPv6 loopback address is:

```
::1/128
```

It serves a similar purpose to:

```
127.0.0.1
```

in IPv4.

Traffic sent to:

```
::1
```

remains inside the local system.

---

## IPv6 Address Configuration

IPv6 addresses can be configured using several methods.

Common approaches include:

```
Static configuration
SLAAC
DHCPv6
```

These methods can also be used together depending on the network design.

An IPv6 interface may have several IPv6 addresses simultaneously.

For example:

```
Link-Local:
fe80::...

Global Unicast:
2001:db8:1234:10:abcd:1234:5678:90ef

Temporary / Privacy Address:
2001:db8:1234:10:8d32:6af1:92ce:4810

Unique Local Address: 
fd12:3456:789a:10::25
```

Having multiple IPv6 addresses on one interface is normal.

---

## SLAAC

SLAAC stands for:

```
Stateless Address Autoconfiguration
```

SLAAC allows a host to generate its own IPv6 address using information received from the local network.

A simplified process is:

```
Interface becomes active
        ↓
Create Link-Local address
        ↓
Receive or request Router Advertisement
        ↓
Learn IPv6 prefix
        ↓
Generate Interface Identifier
        ↓
Construct IPv6 address
        ↓
Perform Duplicate Address Detection
        ↓
Use the address
```

Routers periodically send Router Advertisement messages.

A host can also send a Router Solicitation to request information instead of waiting for the next advertisement.

For example, the router may advertise:

```
2001:db8:1234:10::/64
```

The host can combine that prefix with an Interface Identifier.

For example:

```
Prefix:
2001:db8:1234:10::/64

Interface Identifier:
abcd:1234:5678:90ef
```

Result:

```
2001:db8:1234:10:abcd:1234:5678:90ef
```

Historically, an Interface Identifier could be generated using mechanisms related to the MAC address, such as modified EUI-64.

Modern operating systems commonly use other mechanisms, including stable or temporary privacy-oriented identifiers.

An IPv6 address should therefore not automatically be assumed to contain the device MAC address.

---

## Duplicate Address Detection

Before using a newly generated IPv6 address, the system can perform Duplicate Address Detection, or DAD.

Its purpose is to determine whether another system on the local network is already using the same IPv6 address.

Conceptually:

```
Generate candidate address
        ↓
Check for duplicate
        ↓
No duplicate found
        ↓
Address becomes usable
```

Duplicate Address Detection is part of IPv6 Neighbor Discovery.

The detailed mechanism is covered in the dedicated ARP and NDP topic.

---

## DHCPv6

IPv6 can also use DHCP.

The IPv6 version is called DHCPv6.

Two important models exist.

### Stateful DHCPv6

A DHCPv6 server assigns IPv6 addresses and tracks those assignments.

Conceptually:

```
Client
   ↓
DHCPv6 Server
   ↓
IPv6 Address
```

### Stateless DHCPv6

With stateless DHCPv6, the host may create its own IPv6 address using SLAAC.

DHCPv6 then provides additional information.

For example:

```
DNS configuration
```

One important difference from typical IPv4 DHCP operation is that the IPv6 default router is normally learned from Router Advertisements rather than DHCPv6.

---

## IPv6 Default Gateway

IPv6 still uses routers when communication needs to leave the local network.

The concept of a default gateway therefore still exists.

However, IPv6 hosts normally learn their default router from Router Advertisement messages.

The router's link-local address can be used as the next hop.

For example:

```
Host:
2001:db8:1234:10::25/64

Default Router:
fe80::1
```

This means that both globally routable and link-local addresses can play important roles on the same IPv6 interface.

---

## IPv6 Subnetting

IPv6 uses prefixes and subnetting just like IPv4, but the design philosophy is different.

In IPv4, subnet size is often selected based on the required number of hosts.

For example:

```
/24
→ 254 traditional usable hosts

/27
→ 30 traditional usable hosts
```

IPv6 normally does not use this approach for standard LANs.

A normal IPv6 LAN generally uses:

```
/64
```

even if the subnet contains only a small number of devices.

---

## Dividing a /48 Into /64 Networks

Consider an organization that receives:

```
2001:db8:1234::/48
```

Normal internal LANs use:

```
/64
```

The organization therefore has:

```
64 - 48 = 16 bits
```

available for subnet identification.

This creates:

```
2^16
```

possible subnets.

Therefore:

```
65,536
```

separate `/64` networks can be created.

For example:

```
2001:db8:1234:0001::/64
2001:db8:1234:0002::/64
2001:db8:1234:0003::/64
...
2001:db8:1234:ffff::/64
```

This provides a very large number of networks while keeping each LAN on a standard `/64`.

---

## Why IPv6 LANs Normally Use /64

A `/64` leaves:

```
64 bits
```

for the interface portion.

This represents:

```
2^64
```

possible interface identifiers.

From an IPv4 perspective this may appear extremely wasteful.

However, IPv6 is not normally designed around conserving individual host addresses.

Instead, administrators focus more on:

- subnet organization,
- hierarchical addressing,
- predictable prefixes,
- route summarization,
- operational simplicity.

The `/64` boundary is also important for normal IPv6 mechanisms such as SLAAC.

For this reason, creating:

```
/80
/96
/120
```

LAN networks simply because only a small number of devices are present is generally not recommended.

For example:

```
Users
→ 2001:db8:1234:10::/64

Servers
→ 2001:db8:1234:20::/64

Printers
→ 2001:db8:1234:30::/64

Management
→ 2001:db8:1234:40::/64
```

Each receives an entire `/64`.

---

## Other IPv6 Prefix Sizes

A `/64` is normal for standard LANs, but other prefix lengths can be used for specific purposes.

For example:

```
/64
→ Normal LAN / VLAN
```

```
/127
→ Common on router-to-router point-to-point links
```

```
/128
→ One individual IPv6 address
→ Often used for loopbacks or host routes
```

The subnet size should therefore match the network's purpose rather than following the IPv4 practice of minimizing address consumption.

---

## NAT and IPv6

IPv6 does not normally require NAT for address conservation.

In IPv4, many internal devices commonly share a small number of public addresses.

For example:

```
IPv4

Private addresses
        ↓
NAT / PAT
        ↓
Public Internet
```

IPv6 provides enough globally unique addresses that this is generally unnecessary.

Conceptually:

```
IPv6

Globally unique addresses
        ↓
Routing
        ↓
Internet
```

IPv6 still provides internal addressing options such as Unique Local Addresses.

However, the normal IPv6 design does not require translating every internal address to another public address simply because globally routable addresses are scarce.

---

## NAT64

Address translation may still be required when IPv4 and IPv6 systems need to communicate.

One example is NAT64.

NAT64 can allow an IPv6-only client to communicate with an IPv4 service.

This is different from traditional IPv4 NAT/PAT used primarily to conserve public IPv4 addresses.

---

## NAT Is Not a Firewall

The absence of NAT does not mean the absence of security.

NAT and firewalling perform different functions.

A globally routable IPv6 address does not automatically mean that anyone on the Internet can connect to the device.

Reachability still depends on:

- routing,
- network firewall policy,
- host firewall configuration,
- security controls,
- whether a service is listening.

For example:

```
Internet
    |
Firewall
    |
IPv6 Client
```

The client can have a globally routable IPv6 address while the firewall still blocks unsolicited inbound traffic.

---

## IPv6 Security Considerations

IPv6 should not be considered automatically secure simply because it is newer than IPv4.

IPv6 security still depends on controls such as:

- firewalls,
- endpoint firewalls,
- access control lists,
- segmentation,
- monitoring,
- secure configuration.

One important practical risk appears when IPv6 is enabled but security controls are designed only for IPv4.

For example:

```
IPv4
→ properly filtered

IPv6
→ enabled but not properly filtered
```

This can create an unexpected network path that bypasses assumptions made by administrators.

IPv6 should therefore be included in:

- firewall policies,
- endpoint security,
- monitoring,
- vulnerability testing,
- network documentation.

IPv6 also supports IPsec, but IPv6 traffic is not automatically encrypted simply because IPv6 is being used.

---

## Dual Stack

IPv4 and IPv6 can operate on the same device at the same time.

This is called Dual Stack.

For example, a workstation may have:

```
IPv4:
192.168.10.25
```

and:

```
IPv6:
2001:db8:1234:10::25
```

simultaneously.

The system can then communicate with IPv4 and IPv6 destinations.

---

## DNS and Dual Stack

DNS can provide addresses for both IPv4 and IPv6.

An IPv4 address is normally stored in an A record.

An IPv6 address is normally stored in an AAAA record.

For example:

```
example.com
```

could return:

```
A
→ IPv4 address

AAAA
→ IPv6 address
```

If only IPv4 is available:

```
→ IPv4 is used
```

If only IPv6 is available:

```
→ IPv6 is used
```

If both are available:

```
→ the operating system applies address-selection rules
```

Modern systems generally prefer working IPv6 connectivity while still being able to use IPv4 when appropriate.

This allows organizations to gradually introduce IPv6 without immediately removing IPv4.

---

## Important IPv6 Protocol Differences

IPv6 is not simply IPv4 with larger addresses.

Several protocol behaviors are different.

### Hop Limit

IPv4 uses:

```
TTL
```

IPv6 uses:

```
Hop Limit
```

Each router that forwards the packet decreases the Hop Limit.

When it reaches zero, the packet is discarded.

### No IPv6 Header Checksum

The IPv6 base header does not contain the same header checksum used by IPv4.

This means routers do not need to recalculate an IP header checksum every time they decrease the Hop Limit.

### Fragmentation

IPv6 routers do not fragment packets while forwarding them.

If a packet is too large for the network path, the sending system is expected to adjust the packet size.

This makes mechanisms such as Path MTU Discovery important in IPv6.

IPv6 fragmentation is covered in more detail in the dedicated MTU and Fragmentation topic.

---

## Neighbor Discovery

IPv6 does not use ARP.

Instead, IPv6 uses Neighbor Discovery Protocol, or NDP.

Neighbor Discovery uses ICMPv6 and provides several important functions.

These include:

- neighbor address resolution,
- router discovery,
- prefix discovery,
- Duplicate Address Detection,
- neighbor reachability information.

The detailed behavior of NDP is covered in the dedicated ARP and NDP topic.

---

## ICMPv6

ICMPv6 is an important part of normal IPv6 operation.

It is not used only for ping.

ICMPv6 is involved in mechanisms such as:

- Neighbor Discovery,
- Router Solicitation,
- Router Advertisement,
- Duplicate Address Detection,
- error reporting,
- Path MTU Discovery.  

Blocking all ICMPv6 traffic can therefore break normal IPv6 operation.

ICMP and ICMPv6 are covered in the dedicated ICMP topic.

---

## IPv4 and IPv6 Comparison

A simplified comparison looks like:

```
IPv4
→ 32-bit addresses
→ decimal notation
→ broadcast supported
→ ARP
→ TTL
→ IPv4 header checksum
→ routers may fragment packets
→ NAT/PAT commonly used
→ subnet size often based on number of hosts
```

```
IPv6
→ 128-bit addresses
→ hexadecimal notation
→ no broadcast
→ Neighbor Discovery
→ Hop Limit
→ no base-header checksum
→ routers do not fragment transit packets
→ NAT normally unnecessary for address conservation
→ /64 normally used for LANs
```

IPv4 and IPv6 can operate simultaneously using dual stack.

---

## IPv6 Troubleshooting

Useful questions when troubleshooting IPv6 include:

```
Does the interface have a Link-Local address?

Does the interface have the expected Global Unicast or ULA address?

Is the prefix length correct?

Did the host receive Router Advertisements?

Does the host have a default route?

Is SLAAC working?

Is DHCPv6 expected?

Did Duplicate Address Detection succeed?

Can the local router be reached?

Does DNS return an AAAA record?

Is IPv6 permitted by the firewall?

Is ICMPv6 being blocked?

Does IPv4 work while IPv6 fails?

Does IPv6 work while IPv4 fails?
```

On Linux, useful commands include:

```
ip -6 addr
ip -6 route
ping -6 <destination>
```

On Windows:

```
ipconfig
route print -6
ping -6 <destination>
```

Testing IPv4 and IPv6 independently is especially important in dual-stack environments.

One protocol may work correctly while the other is misconfigured.

---

## Key Takeaways

IPv6 was introduced primarily to overcome the limited IPv4 address space.

IPv6 uses 128-bit addresses instead of IPv4's 32-bit addresses.

IPv6 addresses are written using hexadecimal and divided into eight 16-bit groups.

Leading zeros can be removed and one sequence of zero blocks can be compressed using `::`.

Important IPv6 address types include:

```
Global Unicast
→ globally routable communication

Link-Local
→ communication on the local link

Unique Local
→ internal IPv6 communication

Multicast
→ communication with a selected group

Anycast
→ same address on multiple nodes
→ routing selects one destination
```

IPv6 does not use broadcast.

Normal IPv6 LANs generally use `/64`.

An organization receiving `/48` has 16 bits available for subnet identification before reaching `/64`.

This provides:

```
2^16 = 65,536
```

possible `/64` networks.

IPv6 addresses can be configured using static configuration, SLAAC, or DHCPv6.

Router Advertisements are especially important for IPv6 because they provide information used for address configuration and default-router discovery.

IPv6 does not normally require NAT for address conservation.

However, firewalling remains essential.

A globally routable IPv6 address does not automatically make a device reachable from the Internet.

IPv4 and IPv6 can coexist using dual stack.

Most importantly, IPv6 should not be treated as simply a larger version of IPv4.

It introduces a different addressing philosophy and several different protocol behaviors that administrators need to understand when designing, securing, and troubleshooting modern networks.