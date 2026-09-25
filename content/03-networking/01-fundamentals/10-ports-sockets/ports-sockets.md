# Ports and Sockets

## Overview

Ports and sockets allow operating systems to deliver network traffic to the correct application.

IP addresses identify hosts or interfaces on a network.

Port numbers identify application endpoints on those hosts.

Sockets combine addressing and transport information so the operating system can distinguish one communication endpoint or connection from another.

For example:

```
192.168.1.10:52341 / TCP
```

contains:

```
IP Address:
192.168.1.10

Port:
52341

Transport Protocol:
TCP
```

Ports and sockets are therefore an important link between the network layer and applications.

---

## What Is a Port?

A port is a 16-bit number used by TCP and UDP to identify application endpoints.

The valid range is:

```
0–65535
```

A port number by itself does not identify a complete connection.

For example:

```
443
```

is commonly associated with HTTPS, but the number alone does not guarantee which application is using it.

The transport protocol also matters.

For example:

```
443/TCP
→ commonly HTTPS over TCP
```

while:

```
443/UDP
→ commonly QUIC / HTTP/3
```

An administrator can also configure software to use non-standard port numbers.

A port should therefore be understood as a numeric endpoint identifier rather than a guaranteed description of an application.

---

## Why Ports Are Needed

An IP address identifies the host, but a host can run many network services at the same time.

For example:

```
192.168.1.20
```

could provide:

```
22/TCP
→ SSH

80/TCP
→ HTTP

443/TCP
→ HTTPS
```

The destination port tells the operating system which application should receive the traffic.

Conceptually:

```
IP Address
→ which host?

Port Number
→ which application endpoint?
```

---

## Port Number Ranges

Port numbers are commonly divided into three ranges.

### Well-Known Ports

```
0–1023
```

These are commonly associated with widely used protocols and services.

Examples include:

```
22/TCP
→ SSH

53/TCP and 53/UDP
→ DNS

80/TCP
→ HTTP

443/TCP
→ HTTPS
```

These are conventions and standard assignments.

They do not guarantee which software is actually running on the port.

### Registered Ports

```
1024–49151
```

These ports can be registered for specific applications, products, or services.

Registration helps avoid conflicts and provides a conventional port number, but software can still be configured to use other ports.

### Dynamic / Private Ports

The IANA Dynamic / Private port range is:

```
49152–65535
```

Unlike Well-Known and Registered ports, this range is not intended for standardized permanent service assignments.

These ports are commonly used for:

- temporary client connections,
- private applications,
- dynamically selected application endpoints,
- services that do not require an officially registered port.

The term **Dynamic / Private** describes the port's position in the IANA port-number ranges.

It does not necessarily describe how long the port is used.

For example:

```
55000/TCP
```

could be permanently configured as a listening port for a private application.

In that case, the port is still inside the Dynamic / Private range, but it is not an ephemeral port.

---

## Ephemeral Ports

An ephemeral port is a temporary local port selected by the operating system when an application creates a network connection.

For example:

```
192.168.1.10:53144
        →
203.0.113.20:443
TCP
```

Here:

```
53144
```

is being used as an ephemeral source port.

The operating system temporarily allocates it so returning traffic can be associated with the correct local socket.

When the connection is no longer needed, the port can eventually become available for reuse.

The term **ephemeral** describes how the port is being used:

```
Ephemeral
→ temporary OS-assigned port
```

It does not define one universal numeric range.

Operating systems maintain their own configurable ephemeral-port ranges.

These ranges may match the IANA Dynamic / Private range:

```
49152–65535
```

but they do not have to.

Therefore:

```
Dynamic / Private
→ IANA port-number classification

Ephemeral
→ temporary operating-system allocation
```

The two concepts frequently overlap, which is why the terms are often used interchangeably in everyday networking discussions.

However, they are not technically identical.

---

### Dynamic vs Ephemeral Example

Consider port:

```
55000
```

If the operating system temporarily assigns it to a browser connection:

```
192.168.1.10:55000 → 203.0.113.20:443
```

then it is both:

```
Dynamic / Private
→ because 55000 is inside 49152–65535

Ephemeral
→ because it was temporarily allocated
```

But if an administrator configures an application to permanently listen on:

```
0.0.0.0:55000
```

then it is:

```
Dynamic / Private
→ yes

Ephemeral
→ no
```

This distinction is important:

```
Dynamic / Private
→ describes the numeric range

Ephemeral
→ describes temporary usage
```

---

## Source and Destination Ports

A transport-layer communication normally contains both:

```
Source Port
Destination Port
```

For a typical web connection:

```
192.168.1.10:53144
        →
203.0.113.20:443
TCP
```

the client uses:

```
Source Port:
53144
```

and the server uses:

```
Destination Port:
443
```

The source port is usually selected temporarily by the client operating system.

The destination port normally identifies the service the client wants to reach.

---

## Ports Do Not Map Directly to Browser Tabs

A common mistake is to assume that every browser tab receives its own source port.

This is not necessarily true.

A source port belongs to a network socket or connection.

The operating system tracks which process owns that socket.

The application then decides which request, tab, stream, or internal task should receive the returned data.

Modern protocols can also reuse a single connection for multiple requests.

For example, HTTP/2 and HTTP/3 support multiplexing.

Therefore:

```
one browser tab
≠
one source port
```

---

## What Is a Socket?

A socket is a software endpoint used for network communication.

A socket is associated with information such as:

```
IP Address
Port Number
Transport Protocol
```

For example:

```
192.168.1.10:52341 / TCP
```

describes a TCP endpoint on the local machine.

Sockets allow the operating system to connect network traffic with the correct application.

---

## Listening Socket

A listening socket waits for new incoming TCP connections.

For example:

```
10.0.0.10:443
LISTEN
```

means a local process is waiting for TCP connections on:

```
IP Address:
10.0.0.10

Port:
443
```

The listening socket remains available while multiple clients connect.

---

## Established Socket

After a client connects to a listening TCP socket, the operating system creates connection state for that specific session.

For example:

```
192.168.1.10:51001
        →
10.0.0.10:443
ESTABLISHED
```

Conceptually:

```
Listening Socket
10.0.0.10:443
        |
        | accepts connection
        v
Established Connection
192.168.1.10:51001 → 10.0.0.10:443
```

The listening socket can continue accepting new clients while existing connections remain active.

---

## TCP Connection Tuple

A TCP connection can be identified using four addressing values:

```
Source IP
Source Port
Destination IP
Destination Port
```

This is commonly called a:

```
4-Tuple
```

For example:

```
192.168.1.10:51001
        →
10.0.0.10:443
```

When the transport protocol is included:

```
Protocol
Source IP
Source Port
Destination IP
Destination Port
```

this is commonly described as a:

```
5-Tuple
```

The transport protocol matters because TCP and UDP have separate port spaces.

---

## Multiple Connections to the Same Service

A server can accept many simultaneous connections on a single listening port.

For example:

```
Listening Socket:
10.0.0.10:443
```

may have established connections such as:

```
192.168.1.10:51001 → 10.0.0.10:443
192.168.1.11:52002 → 10.0.0.10:443
192.168.1.12:53003 → 10.0.0.10:443
```

The server distinguishes the connections using the combination of source and destination addresses and ports.

The same client can also create several connections to the same server by using different source ports.

---

## Binding to an Address

A service can choose which local addresses it listens on.

```
0.0.0.0:443
```

means the service is bound to all local IPv4 interfaces.

```
127.0.0.1:443
```

means the service is bound only to the IPv4 loopback interface.

```
192.168.1.20:443
```

means the service is bound specifically to that local IPv4 address.

These bindings describe where the service listens locally, not which remote clients are allowed to connect.

Firewall policy and routing still control remote reachability.

---

## IPv6 Binding

The IPv6 unspecified address is:

```
::
```

A service shown as:

```
[::]:80
```

is generally bound to all local IPv6 interfaces.

This is similar in concept to:

```
0.0.0.0:80
```

for IPv4.

Conceptually:

```
0.0.0.0:80
→ all local IPv4 interfaces

[::]:80
→ all local IPv6 interfaces
```

If both listeners exist, the service is clearly listening for both IPv4 and IPv6 traffic, subject to routing and firewall rules.

One important nuance is that some operating systems can allow an IPv6 socket bound to `::` to also accept IPv4-mapped connections, depending on socket configuration.

Therefore, dual-stack behavior should be verified rather than assumed.

---

## Listening Does Not Mean Reachable

A service can be listening locally while still being unreachable from another machine.

For example:

```
0.0.0.0:22
LISTEN
```

shows that the application is waiting for TCP connections.

However, access may still be prevented by:

- host firewall rules,
- network firewalls,
- ACLs,
- routing,
- NAT configuration,
- security policy.

A useful distinction is:

```
Listening
→ application is ready locally

Reachable
→ the network path and security policy also allow access
```

---

## Open, Closed, and Filtered Ports

From a scanner's perspective:

```
Open
→ a service appears to be listening
```

```
Closed
→ the host is reachable, but no service appears to be listening
```

```
Filtered
→ filtering prevents the scanner from reliably determining whether the port is open or closed
```

For TCP, a closed port may respond to a SYN with:

```
RST
```

A filtered port may produce no useful response because a firewall or ACL silently drops the probe.

Scan results are always relative to the machine performing the scan.

A different client with different firewall permissions may see a different result.

---

## TCP vs UDP Socket Behavior

TCP has explicit connection states such as:

```
LISTEN
ESTABLISHED
TIME_WAIT
CLOSE_WAIT
```

UDP is connectionless and does not use the same TCP state machine.

A UDP socket can be bound to a local address and port, but there is no TCP-style:

```
LISTEN → ESTABLISHED
```

---

## TIME_WAIT

After a TCP connection closes, one endpoint may remain in:

```
TIME_WAIT
```

for a period of time.

Conceptually:

```
ESTABLISHED
    ↓
Connection closes
    ↓
TIME_WAIT
    ↓
Timer expires
    ↓
Old connection state removed
```

One purpose of TIME_WAIT is to prevent delayed packets from an old connection from being confused with packets from a newer connection using the same addressing information.

It also allows the final acknowledgement to be retransmitted if needed.

Seeing many TIME_WAIT entries on a busy system does not automatically mean something is wrong.

---

## Ephemeral Port Exhaustion

A system has a finite range of ephemeral ports available for new outbound connections.

If the operating system cannot allocate another suitable source port for a new connection, connection creation can fail.

Possible causes include:

- unusually high connection volume,
- badly behaving applications,
- connection leaks,
- very large numbers of short-lived connections,
- denial-of-service conditions.

It is more accurate to think in terms of available connection tuples rather than just counting port numbers, because a local source port may sometimes be reused with a different destination depending on operating-system behavior and connection state.

---

## Common Port Examples

Some useful examples are:

```
22/TCP
→ SSH

53/TCP
→ DNS

53/UDP
→ DNS

67/UDP
→ DHCP Server

68/UDP
→ DHCP Client

80/TCP
→ HTTP

443/TCP
→ HTTPS

443/UDP
→ QUIC / HTTP/3
```

These are conventions.

The protocols themselves should be studied in dedicated articles rather than treating the port number as the protocol definition.

---

## Security Considerations

Open ports represent reachable application endpoints.

Every listening service potentially increases the attack surface of a system.

Useful security principles include:

- disable unnecessary services,
- bind services only to required interfaces,
- use firewalls to restrict access,
- avoid exposing administrative services unnecessarily,
- regularly review listening sockets,
- remember that a port number does not prove which application is running,
- monitor unexpected listeners,
- include both IPv4 and IPv6 when reviewing exposed services.

For example:

```
127.0.0.1:8080
```

is normally exposed only to the local host, while:

```
0.0.0.0:8080
```

makes the service listen on all local IPv4 interfaces.

---

## Troubleshooting

Useful questions include:

```
Is the application running?

Is it listening?

Which local IP address is it bound to?

Which port is it using?

Is it using TCP or UDP?

Is it bound only to localhost?

Is it bound to IPv4, IPv6, or both?

Is a firewall allowing the connection?

Is the port open, closed, or filtered from the client perspective?

Which process owns the socket?

Are many connections in TIME_WAIT?

Is ephemeral port exhaustion occurring?
```

---

## Linux Commands

Show listening TCP sockets:

```
ss -ltn
```

Show listening UDP sockets:

```
ss -lun
```

Show TCP connections:

```
ss -tan
```

Show UDP sockets:

```
ss -uan
```

Show listening sockets and owning processes:

```
ss -tulpn
```

Test a TCP port:

```
nc -vz <host> <port>
```

---

## Windows Commands

Show active connections and listening ports:

```
netstat -ano
```

Show TCP connections:

```
Get-NetTCPConnection
```

Show UDP endpoints:

```
Get-NetUDPEndpoint
```

Test a TCP port:

```
Test-NetConnection <host> -Port <port>
```

---

## Key Takeaways

A port is a 16-bit number in the range:

```
0–65535
```

The main IANA port ranges are:

```
0–1023
→ Well-Known

1024–49151
→ Registered

49152–65535
→ Dynamic / Private
```

Dynamic / Private describes the numeric IANA range.

Ephemeral describes a temporary port selected by the operating system for a connection.

The two often overlap, but they are not technically the same thing.

Ports identify application endpoints.

IP addresses identify hosts or interfaces.

A socket combines information such as:

```
IP Address
Port
Transport Protocol
```

A listening TCP socket waits for new incoming connections.

An established TCP connection is identified by both endpoints.

A TCP 4-tuple contains:

```
Source IP
Source Port
Destination IP
Destination Port
```

Including the transport protocol creates the common 5-tuple concept.

A server can handle many simultaneous connections on one listening port because each connection has a different tuple.

Binding matters:

```
0.0.0.0
→ all local IPv4 interfaces

127.0.0.1
→ IPv4 loopback only

Specific local IP
→ only that local address

::
→ generally all local IPv6 interfaces
```

A listening service is not automatically reachable from another system because routing and firewalls still apply.

From a scanner's perspective:

```
Open
→ service appears to be listening

Closed
→ host reachable, but no service listening

Filtered
→ filtering prevents reliable determination
```

Ephemeral ports are temporary source ports used for client connections.

After TCP connections close, some state may remain temporarily in TIME_WAIT.

If the operating system cannot allocate another suitable ephemeral source port or connection tuple, new outbound connections may fail.

Understanding ports and sockets makes it much easier to interpret firewall rules, packet captures, `ss`, `netstat`, and port scans.