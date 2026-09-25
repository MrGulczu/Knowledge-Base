# MTU and Fragmentation

## Overview

MTU and fragmentation determine how large IP packets can be when they travel across a network.

These concepts are especially important when working with Ethernet, VPNs, IPsec, GRE, PPPoE, IPv4, IPv6, Path MTU Discovery, and jumbo frames.

A connection can appear healthy for small packets while failing for larger packets if MTU handling is incorrect.

---

## What Is MTU?

MTU stands for:

```
Maximum Transmission Unit
```

For an Ethernet interface configured with:

```
MTU 1500
```

the value normally describes the maximum size of the Layer 3 packet carried inside the Ethernet frame.

```
Ethernet Frame
┌──────────────────────┐
│ Ethernet Header      │
├──────────────────────┤
│ IP Packet            │  ← maximum 1500 bytes
│   IP Header          │
│   TCP/UDP Header     │
│   Application Data   │
├──────────────────────┤
│ FCS                  │
└──────────────────────┘
```

The complete Ethernet frame is larger than 1500 bytes because the Ethernet header and FCS are outside the IP MTU calculation.

So:

```
Ethernet MTU 1500
→ maximum IP packet carried as Ethernet payload = 1500 bytes
```

---

## Headers Reduce Available Payload

The MTU includes the IP packet headers, so the amount of actual application data that fits inside the packet is smaller.

For a typical IPv4 TCP packet:

```
Ethernet MTU:
1500 bytes

IPv4 Header:
20 bytes

TCP Header:
20 bytes

TCP Data:
1460 bytes
```

Therefore:

```
1500 - 20 - 20 = 1460 bytes
```

This value becomes important when discussing TCP MSS.

---

## IPv4 Fragmentation

IPv4 routers can fragment packets in transit when a packet is too large for the outgoing interface MTU, provided fragmentation is allowed.

For example:

```
Original IPv4 Packet:
3400 bytes

Outgoing Link MTU:
1500 bytes
```

Assuming a standard 20-byte IPv4 header:

```
Maximum fragment:
1500 bytes

IPv4 header:
20 bytes

Maximum fragment payload:
1480 bytes
```

The packet could be fragmented approximately as:

```
Fragment 1:
20-byte IPv4 header
+ 1480 bytes payload
= 1500 bytes

Fragment 2:
20-byte IPv4 header
+ 1480 bytes payload
= 1500 bytes

Fragment 3:
20-byte IPv4 header
+ 420 bytes payload
= 440 bytes
```

The total amount transmitted becomes slightly larger because every fragment carries its own IPv4 header.

IPv4 uses a Fragment Offset field so the destination can reconstruct the original packet. Fragment offsets are expressed in units of 8 bytes, so the payload size of every fragment except the last normally needs to be divisible by 8.

---

## Don't Fragment Flag

IPv4 includes the:

```
DF
Don't Fragment
```

flag.

```
DF = 0
→ router may fragment the IPv4 packet

DF = 1
→ router must not fragment it
```

If a packet is too large for the next link and DF is set:

```
Oversized IPv4 packet
        ↓
DF = 1
        ↓
Router cannot fragment
        ↓
Packet dropped
        ↓
ICMP Fragmentation Needed
```

The DF flag is an important part of Path MTU Discovery.

It is not primarily a security mechanism and does not protect against man-in-the-middle attacks.

---

## IPv4 Fragment Reassembly

Routers may fragment IPv4 packets, but they do not normally reassemble them in transit.

Reassembly occurs at the final destination:

```
Original IPv4 Packet
        ↓
Router fragments packet
        ↓
Fragment 1
Fragment 2
Fragment 3
        ↓
Destination Host
        ↓
Reassembly
```

If one fragment is lost, the destination cannot reconstruct the original packet.

IP itself does not retransmit missing fragments.

For TCP:

```
Missing data
→ TCP detects loss
→ TCP retransmits required data
```

For UDP:

```
Missing fragment
→ packet cannot be reassembled
→ UDP does not retransmit automatically
```

This is one reason fragmentation is undesirable: losing one fragment makes the complete original packet unusable.

---

## IPv6 Fragmentation

IPv6 handles fragmentation differently from IPv4.

IPv6 routers do not fragment packets in transit like its in IPv4.

If a router receives a packet that is too large for the next link:

```
IPv6 packet too large
        ↓
Router drops packet
        ↓
ICMPv6 Packet Too Big
        ↓
Sender learns smaller MTU
        ↓
Sender adjusts packet size
```

Therefore:

```
IPv4
→ routers may fragment packets in transit

IPv6
→ routers do not fragment packets in transit
```

IPv6 still supports fragmentation, but only the source host performs it using the IPv6 Fragment extension header.

---

## Minimum IPv6 MTU

Every IPv6 link must support an MTU of at least:

```
1280 bytes
```

If an underlying link technology cannot directly carry 1280-byte IPv6 packets, it must provide fragmentation and reassembly below the IPv6 layer.

So:

```
IPv6 minimum link MTU
→ 1280 bytes

IPv6 routers
→ never fragment transit packets

IPv6 source
→ may fragment when necessary
```

---

## Path MTU

Path MTU is the smallest MTU supported by any link along the complete end-to-end path.

For example:

```
Host
MTU 1500
   |
Router
MTU 1500
   |
Tunnel
MTU 1400
   |
Destination
MTU 1500
```

The Path MTU is:

```
1400 bytes
```

because that is the smallest value along the path.

---

## Why Tunnels Affect MTU

Tunnels add extra headers around the original packet.

For example:

```
Original IP Packet
        ↓
IPsec Header
UDP Encapsulation
Outer IP Header
        ↓
Larger Encapsulated Packet
```

If the physical network supports an MTU of 1500, the packet inside the tunnel may need to be smaller so the complete encapsulated packet still fits.

This is why MTU problems commonly appear with:

```
IPsec
GRE
PPPoE
VPN tunnels
Overlay networks
```

The issue is not that tunnels are inherently problematic. The issue is the additional encapsulation overhead and whether the resulting Path MTU is handled correctly.

---

## Path MTU Discovery

Path MTU Discovery, or PMTUD, allows a host to learn the largest packet size that can travel through the full path without inappropriate fragmentation.

It normally happens while communication is taking place.

### IPv4 PMTUD

```
Sender transmits packet with DF set
        ↓
Router finds packet is too large
        ↓
Router cannot fragment
        ↓
Packet dropped
        ↓
ICMP Fragmentation Needed
        ↓
Sender reduces packet size
```

### IPv6 PMTUD

```
Sender transmits IPv6 packet
        ↓
Router finds packet is too large
        ↓
Router drops packet
        ↓
ICMPv6 Packet Too Big
        ↓
Sender reduces packet size
```

---

## ICMP and PMTUD

ICMP is not used only for ping.

It also carries important control information required for normal network operation.

For MTU handling, important examples include:

```
IPv4:
ICMP Destination Unreachable
Fragmentation Needed

IPv6:
ICMPv6 Packet Too Big
```

Blindly blocking all ICMP can break Path MTU Discovery.

A better approach is to allow ICMP message types required for normal network operation and restrict individual message types only when there is a reason to do so.

This is especially important for IPv6 because ICMPv6 is involved in several fundamental IPv6 mechanisms.

---

## PMTUD Black Hole

A Path MTU black hole can occur when oversized packets are dropped but the required ICMP messages are also blocked.

```
Small packets
→ pass

Large packets
→ exceed Path MTU
→ dropped

Required ICMP
→ blocked

Sender never learns correct MTU
```

This can create confusing symptoms:

```
Ping works
DNS works
VPN connects
SSH connects
```

while:

```
Web pages partially load
File transfers stall
TLS sessions time out
Large packets disappear
```

The connection itself may establish successfully and only fail when larger packets begin to flow.

---

## Maximum Segment Size

MSS stands for:

```
Maximum Segment Size
```

MSS is a TCP concept.

It describes the maximum amount of TCP payload a host advertises it is willing to receive in a single TCP segment.

MSS does not include the IP or TCP headers.

For typical IPv4 over Ethernet:

```
MTU:
1500 bytes

IPv4 Header:
20 bytes

TCP Header:
20 bytes

MSS:
1460 bytes
```

Therefore:

```
1500 - 20 - 20 = 1460
```

For basic IPv6:

```
MTU:
1500 bytes

IPv6 Header:
40 bytes

TCP Header:
20 bytes

TCP Payload:
1440 bytes
```

Additional IPv6 extension headers or TCP options can reduce the available payload further.

---

## MTU vs MSS

The important distinction is:

```
MTU
→ maximum Layer 3 packet size

MSS
→ maximum TCP payload advertised for one segment
```

Hosts advertise MSS during the TCP handshake:

```
Client
→ SYN, MSS 1460

Server
→ SYN-ACK, MSS 1460
```

Each side can advertise a different MSS.

---

## TCP MSS Clamping

TCP MSS clamping is commonly used on routers and VPN gateways when a tunnel reduces the usable Path MTU.

The device modifies the MSS value advertised during the TCP handshake.

For example:

```
Original SYN:
MSS 1460

        ↓
VPN Gateway

Modified SYN:
MSS 1360
```

The remote endpoint then sends smaller TCP segments.

```
MSS Clamping
→ modifies advertised TCP MSS
→ endpoints use smaller TCP payloads
→ helps avoid oversized packets
```

MSS clamping can be especially useful when VPN encapsulation reduces the effective MTU or when PMTUD is unreliable.

A major limitation is:

```
MSS Clamping
→ TCP only
```

It does not directly solve MTU problems for UDP, ICMP, or other protocols.

---

## Jumbo Frames

Standard Ethernet commonly uses:

```
MTU 1500
```

Jumbo frames use an MTU larger than the standard Ethernet MTU.

A common value is approximately:

```
9000 bytes
```

Jumbo frames can reduce per-packet processing overhead when transferring large amounts of data.

They are often used in controlled environments such as:

```
Storage networks
Virtualization clusters
Datacenter networks
High-throughput server links
```

---

## Jumbo Frame Compatibility

Jumbo frames require compatible configuration across the relevant Layer 2 path.

For example:

```
Host
MTU 9000
   |
Switch
MTU 9000
   |
Switch
MTU 1500
   |
Destination
```

The smaller MTU in the middle can create connectivity problems.

The problem is not that jumbo frames are inherently unreliable.

The problem is inconsistent MTU configuration.

---

## Typical MTU Problem Symptoms

MTU problems can produce unusual and inconsistent behavior.

Examples include:

```
VPN connects but applications fail
Small pings work
Large pings fail
Websites partially load
TLS sessions stall
File transfers freeze
Some applications work while others fail
Large packets disappear
```

These symptoms often occur because smaller packets fit within the available Path MTU while larger packets do not.

---

## Testing MTU with Ping on Linux

On Linux, IPv4 MTU testing can be performed with:

```
ping -M do -s 1472 8.8.8.8
```

Here:

```
-M do
→ do not allow fragmentation

-s 1472
→ ICMP payload size
```

For a standard 1500-byte IPv4 MTU:

```
1472 bytes ICMP payload
+ 8 bytes ICMP header
+ 20 bytes IPv4 header
= 1500 bytes
```

If the test fails, try smaller values:

```
ping -M do -s 1400 8.8.8.8
ping -M do -s 1350 8.8.8.8
ping -M do -s 1300 8.8.8.8
```

Then increase the size gradually to find the largest successful value.

---

## Testing MTU with Ping on Windows

On Windows:

```
ping 8.8.8.8 -f -l 1472
```

Here:

```
-f
→ set Don't Fragment

-l
→ ICMP payload size
```

The value given to ping represents the ICMP payload, not the complete IP packet.

---

## Why 1472 Is Used for MTU 1500

For IPv4:

```
ICMP Payload:
1472 bytes

ICMP Header:
8 bytes

IPv4 Header:
20 bytes
```

Total:

```
1472 + 8 + 20 = 1500 bytes
```

A ping payload of 1500 bytes would result in:

```
1500
+ 8
+ 20
= 1528 bytes
```

which exceeds a standard 1500-byte MTU.

Therefore:

```
ping payload size
≠
complete IP packet size
```

A failed large ping does not automatically prove an MTU problem because ICMP Echo itself may be filtered.

The test is more useful when small DF pings succeed and larger DF pings consistently fail.

---

## Security and Operational Considerations

Fragmentation and MTU behavior should be understood when designing:

```
Firewalls
VPNs
IPsec tunnels
WAN links
Overlay networks
Cloud networking
```

Important practices include:

- do not blindly block all ICMP,
- account for tunnel encapsulation overhead,
- avoid unnecessary fragmentation,
- use PMTUD correctly,
- use MSS clamping where appropriate for TCP,
- keep MTU settings consistent,
- validate jumbo-frame support across the complete path,
- remember that IPv4 and IPv6 fragmentation behavior differs.

MTU problems are often mistaken for firewall, application, VPN, TLS, or routing problems because connectivity may fail only for larger packets.

---

## Troubleshooting

Useful questions include:

```
What MTU is configured on the local interface?

What is the smallest MTU along the path?

Is a VPN or tunnel adding encapsulation overhead?

Is IPv4 fragmentation allowed?

Is the DF flag set?

Is ICMP Fragmentation Needed being blocked?

Is ICMPv6 Packet Too Big being blocked?

Does the problem affect only larger packets?

Do small DF pings work?

What is the maximum successful ping payload?

Is TCP MSS appropriate for the Path MTU?

Is MSS clamping configured?

Are jumbo frames enabled consistently?

Are fragments being dropped by a firewall?
```

Packet captures can help identify:

```
IPv4 fragmentation
DF flag
ICMP Fragmentation Needed
ICMPv6 Packet Too Big
TCP MSS values
Repeated TCP retransmissions
```

---

## Key Takeaways

MTU describes the maximum Layer 3 packet size that can be carried by a link.

For standard Ethernet:

```
MTU 1500
```

normally means a maximum 1500-byte IP packet.

IPv4 routers may fragment packets in transit when:

```
DF = 0
```

If:

```
DF = 1
```

an oversized packet is dropped instead.

IPv4 fragments are reassembled only by the final destination.

If one fragment is lost, the original IP packet cannot be reconstructed.

IPv6 routers never fragment transit packets.

IPv6 fragmentation can only be performed by the source host.

The minimum IPv6 link MTU is:

```
1280 bytes
```

Path MTU is the smallest MTU along the complete communication path.

PMTUD uses ICMP or ICMPv6 feedback to help a sender discover the usable packet size.

Blocking required ICMP messages can create a Path MTU black hole.

MSS describes TCP payload size, not complete packet size.

For typical IPv4 over Ethernet:

```
MTU 1500
→ MSS 1460
```

TCP MSS clamping can help when tunnels reduce the usable Path MTU.

Jumbo frames commonly use an MTU around:

```
9000 bytes
```

but require consistent support across the relevant path.

A common IPv4 MTU test for a 1500-byte path is:

```
ping -M do -s 1472 <destination>
```

because:

```
1472 payload
+ 8 ICMP
+ 20 IPv4
= 1500 bytes
```

Understanding MTU, fragmentation, PMTUD, and MSS is especially important when troubleshooting VPNs, tunnels, partially working connections, and large packet transfers.