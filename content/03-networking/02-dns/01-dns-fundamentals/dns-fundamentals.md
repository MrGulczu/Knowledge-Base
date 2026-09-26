# DNS Fundamentals

DNS is one of the core services used in modern computer networks. It allows devices and applications to use human-readable names instead of relying only on numeric IP addresses.

DNS stands for **Domain Name System**.

This topic builds on several concepts already covered in Networking Fundamentals. When a concept is only needed as background, this article links back to the existing explanation instead of repeating it in full.

Useful prerequisite topics:

- [Network Models](../../01-fundamentals/01-network-models/network-models.md)
- [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md)
- [IPv6 Basics](../../01-fundamentals/06-ipv6-basics/ipv6-basics.md)
- [TCP and UDP](../../01-fundamentals/09-tcp-udp/tcp-udp.md)
- [Ports and Sockets](../../01-fundamentals/10-ports-sockets/ports-sockets.md)

A common example is accessing a website such as:

```
google.com
```

Instead of requiring the user to remember the server's IPv4 or IPv6 address, the system can use DNS to obtain the information associated with the name.

At its simplest, DNS can be thought of as:

```
Name
  |
  v
DNS
  |
  v
IP address
```

However, DNS is much more than a simple name-to-IP translation service. It is a distributed naming system that can store many different types of information about names, services, and domains.

---

## Why DNS Exists

Computers communicate across IP networks using IP addresses.

IP addressing itself is covered in detail in [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md), with IPv6 expanded further in [IPv6 Basics](../../01-fundamentals/06-ipv6-basics/ipv6-basics.md).

For example:

```
192.168.10.50
```

or:

```
2001:db8::50
```

These addresses work well for computers, but they are difficult for people to remember and manage.

It is much easier to remember:

```
server01.example.com
```

than:

```
192.168.10.50
```

DNS provides the mechanism that allows applications and users to work with names while the underlying network continues to use IP addresses.

A simplified example looks like this:

```
User enters:

server01.example.com

        |
        v

Workstation asks DNS

        |
        v

DNS returns:

192.168.10.50

        |
        v

Workstation can communicate
with the destination
```

DNS therefore creates a layer of abstraction between names used by people and applications and the IP addresses used for network communication.

---

## DNS Is Not the Network Connection

DNS resolution and IP connectivity are two different things.

The underlying IP addressing, local-versus-remote decision, and default gateway behavior are already covered in [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md).

Consider the following workstation:

```
IP address:      192.168.10.25
Default gateway: 192.168.10.1
DNS server:      192.168.10.10
```

If the workstation can access the network and the Internet but cannot resolve:

```
www.example.com
```

the underlying IP connectivity may still be working correctly.

For example:

```
Workstation
192.168.10.25
     |
     v
Default Gateway
192.168.10.1
     |
     v
Internet
```

can still be fully operational even if DNS resolution fails.

If the destination IP address is already known, the workstation can still attempt to communicate with it directly.

This leads to an important troubleshooting principle:

> **DNS failure does not automatically mean network connectivity failure.**

For example, a device may successfully communicate with an IP address while failing to access the same destination by name.

There is one important limitation to remember. Some applications and services rely on the domain name itself.

For example, many websites use HTTPS certificates issued for names such as:

```
example.com
```

and multiple websites may also share the same IP address.

Because of this, accessing a service directly by IP address may not always produce the same result as accessing it by its DNS name.

---

## DNS Client and Resolver

DNS is an application-layer service. The relationship between application, transport, network, and link-layer communication is covered in [Network Models](../../01-fundamentals/01-network-models/network-models.md).

A workstation does not normally contain all DNS information itself.

Instead, the operating system provides a component responsible for name resolution. This component is commonly called a **DNS resolver** or **stub resolver**.

Applications can request name resolution through the operating system.

For example:

```
Browser
   |
   v
Operating System
DNS Resolver
   |
   v
DNS Server
```

If an application wants to access:

```
server01.example.com
```

the local resolver can send a DNS query to one of the DNS servers configured on the machine.

For example:

```
Workstation
192.168.10.25

Configured DNS server:
192.168.10.10
```

The resolver can ask:

```
What is the IP address associated with
server01.example.com?
```

The DNS server then returns the appropriate information if it can obtain the answer.

A device sending DNS queries therefore acts as a **DNS client**.

---

## DNS Server

A DNS server runs a service capable of receiving and responding to DNS queries.

For example:

```
Workstation
192.168.10.25
     |
     | DNS query
     v
DNS Server
192.168.10.10
```

The DNS server does not need to contain every DNS record in the world.

Depending on its role, it may:

- answer directly from authoritative information it manages,
- answer from information stored in its cache,
- ask other DNS servers for information,
- forward the request to another resolver.

Different DNS servers can therefore perform different roles.

---

## Recursive Resolver

A **recursive resolver** works on behalf of a client.

If the recursive resolver does not already know the answer, it can communicate with other DNS servers to obtain it.

A simplified model looks like this:

```
Client
  |
  v
Recursive Resolver
  |
  | does not know the answer
  v
Other DNS Servers
  |
  v
Answer
  |
  v
Recursive Resolver
  |
  v
Client
```

The recursive resolver may also cache the answer so that future requests can be answered more quickly.

The detailed resolution process is covered in the dedicated DNS resolution topic.

---

## Authoritative DNS Server

An **authoritative DNS server** contains authoritative information for a DNS zone it is responsible for.

It can be treated as the source of truth for that particular portion of the DNS namespace.

For example:

```
Authoritative DNS Server
for example.com

example.com
├── www
├── mail
├── vpn
└── server01
```

An authoritative server for:

```
example.com
```

does not automatically contain authoritative information for:

```
google.com
```

or:

```
microsoft.com
```

The server is authoritative only for the namespace or zones it is responsible for.

A single DNS server can also perform multiple roles. In an internal network, the same DNS server may be authoritative for an internal zone while also providing recursive DNS resolution for clients.

---

## DNS Is a Distributed System

There is no single DNS server that contains every DNS record in the world.

DNS is a **distributed hierarchical system**.

Responsibility for different parts of the DNS namespace is delegated between many independent DNS servers.

A simplified hierarchy looks like this:

```
.
└── com.
    └── example.com.
        └── server01.example.com.
```

The DNS root represents the highest level of the hierarchy.

Different DNS systems are responsible for different levels or portions of the namespace.

For example:

```
DNS Root
│
├── .com
│   ├── example.com
│   ├── google.com
│   └── microsoft.com
│
├── .org
│   ├── wikipedia.org
│   ├── archive.org
│   └── python.org
│
├── .net
│   ├── battle.net
│   ├── php.net
│   └── reserachgate.net
│
└── ...
```

This distribution allows DNS to scale globally without requiring one central server to maintain every possible record.

The complete DNS hierarchy and delegation process are covered in a separate topic.

---

## Hostname

A **hostname** is the name assigned to a particular device or system.

For example:

```
server01
```

In home networks, device names may be automatically generated or fairly simple.

In organizations, administrators often use naming conventions that make systems easier to identify.

Examples might include:

```
srv-file-01
pc-fin-023
fw-waw-01
```

The exact naming convention depends on the organization.

A hostname by itself does not necessarily identify the complete location of the device within DNS.

---

## Domain Name

A **domain name** represents a namespace within DNS.

For example:

```
example.com
```

Names can exist underneath that domain:

```
www.example.com
mail.example.com
vpn.example.com
server01.example.com
```

A DNS domain should not be confused with an **Active Directory domain**.

Active Directory commonly uses DNS naming and depends heavily on DNS, but Active Directory and DNS are separate technologies.

For example:

```
corp.example.com
```

could be used as the name of an Active Directory domain, but the existence of a DNS name does not mean that every device using that name is joined to Active Directory.

---

## Fully Qualified Domain Name

FQDN stands for **Fully Qualified Domain Name**.

An FQDN represents the complete DNS name of a resource within the DNS hierarchy.

For example:

```
server01.example.com
```

can be broken into:

```
server01    hostname / label
example     domain label
com         top-level domain
```

The fully explicit DNS representation also includes the DNS root:

```
server01.example.com.
```

The final dot represents the root of the DNS hierarchy.

Modern systems usually omit the trailing dot when displaying or entering DNS names:

```
server01.example.com
```

but conceptually the root is still present.

---

## DNS Labels

Individual parts of a DNS name separated by dots are called **labels**.

For example:

```
www.sales.example.com.
```

contains the labels:

```
www
sales
example
com
```

The DNS hierarchy becomes more specific from right to left:

```
.                       DNS root
└── com                 Top-Level Domain
    └── example         example.com
        └── sales       sales.example.com
            └── www     www.sales.example.com
```

This gives the complete name:

```
www.sales.example.com.
```

In normal use it is usually written without the final root dot:

```
www.sales.example.com
```

---

## FQDN Does Not Always Mean a Physical Machine

An FQDN does not have to refer directly to one physical computer.

Names such as:

```
mail.example.com
vpn.example.com
portal.example.com
```

may represent:

- servers,
- services,
- load balancers,
- aliases,
- virtual services,
- other network resources.

DNS should therefore be treated as a naming system for network resources rather than only a mechanism for naming physical machines.

---

## DNS Stores More Than IP Addresses

One of the most common DNS functions is resolving a name to an IP address.

For example:

```
server01.example.com
        |
        v
192.168.10.50
```

However, DNS can store many different types of information.

DNS can be used for information related to:

- IPv4 addresses,
- IPv6 addresses,
- mail servers,
- aliases,
- authoritative name servers,
- service locations,
- text-based information,
- reverse mappings,
- verification and policy information.

These different types of DNS information are represented by different DNS record types.

DNS record types are covered in detail in a dedicated topic.

---

## How a Client Learns Which DNS Server to Use

A DNS resolver must know which DNS server or servers it should query.

This information can be configured in several ways.

### Static Configuration

An administrator can configure DNS servers manually.

For example:

```
IP address:      192.168.10.25
Default gateway: 192.168.10.1

DNS servers:
192.168.10.10
192.168.10.11
```

Static DNS configuration is common on infrastructure systems, servers, and devices where predictable configuration is important.

### DHCP

Clients can also receive DNS server information automatically through DHCP.

Static and dynamic IP configuration, including the basic role of DHCP and the network settings it can provide, is already introduced in [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md).

For example:

```
DHCP Server
    |
    | IP address
    | subnet information
    | default gateway
    | DNS servers
    v
Workstation
```

This allows workstations to receive the required network configuration automatically when connecting to the network.

Other technologies, such as VPN software or device-management policies, can also influence DNS resolver configuration.

---

## Multiple DNS Servers

A client is often configured with more than one DNS server.

For example:

```
DNS servers:

192.168.10.10
192.168.10.11
```

The primary purpose is **redundancy**.

If one DNS server is unavailable, the resolver may be able to use another configured server.

For example:

```
192.168.10.10   unavailable
192.168.10.11   available
```

This helps prevent one DNS server from becoming a single point of failure.

It is important not to assume that every operating system follows a simple rule of:

```
Always use DNS server 1
and only use DNS server 2
when DNS server 1 fails
```

The exact resolver behavior depends on the operating system and implementation.

A client may retry, change server preference, or select another configured DNS server based on previous responses and failures.

---

## Local DNS Cache

A workstation does not necessarily need to contact a DNS server every time an application uses the same name.

DNS responses can be stored temporarily in a **local DNS cache**.

For example, the workstation may previously have learned:

```
server01.example.com
        |
        v
192.168.10.50
```

A later request may be answered directly from the local cache:

```
Application
    |
    v
Local DNS Resolver
    |
    | cached answer available
    v
192.168.10.50
```

This reduces unnecessary DNS queries and can improve performance.

Cached information is not normally stored forever.

DNS information is associated with a **TTL — Time To Live**, which determines how long the information may remain cached.

For example:

```
server01.example.com
192.168.10.50

TTL: 3600 seconds
```

A TTL of:

```
3600 seconds
```

equals:

```
1 hour
```

The details of DNS caching, TTL behavior, negative caching, and DNS propagation are covered in a dedicated topic.

---

## Hosts File

A system can also contain manually configured local name mappings.

These are typically stored in a **hosts file**.

For example:

```
192.168.10.50    server01.example.com
```

This is different from the DNS cache.

A DNS cache contains information learned dynamically from DNS responses and is normally limited by TTL.

A hosts file contains manually configured entries that remain until they are changed or removed.

A simplified name-resolution process can therefore include several local sources before a DNS server is contacted.

For example:

```
Application
    |
    v
Local Resolver
    |
    ├── DNS cache
    ├── hosts file
    |
    └── DNS server
```

The exact lookup order depends on the operating system and resolver configuration.

---

## DNS Ports and Transport Protocols

The general behavior of the two transport protocols is covered in [TCP and UDP](../../01-fundamentals/09-tcp-udp/tcp-udp.md), while the purpose of transport-layer ports is covered in [Ports and Sockets](../../01-fundamentals/10-ports-sockets/ports-sockets.md).

For DNS specifically, traditional DNS uses **port 53**.

DNS can use both:

```
UDP/53
TCP/53
```

### UDP

Most standard DNS queries and responses use:

```
UDP port 53
```

UDP has relatively low overhead and works well for normal DNS transactions.

A simplified example is:

```
Client
  |
  | UDP/53
  v
DNS Server
```

### TCP

DNS can also use:

```
TCP port 53
```

TCP is used when DNS communication requires reliable transport or when a response cannot be handled normally over UDP.

TCP is also used for some DNS server-to-server operations.

The older simplified statement that "DNS queries use UDP and only zone transfers use TCP" is not completely accurate. Normal DNS queries can also use TCP when required.

More advanced DNS transport behavior is covered later.

---

## DNS Resolution Can Be Local or Remote

Not every name-resolution request results in a DNS query being sent across the network.

The operating system may already have enough information locally.

For example:

```
Application
    |
    v
Local Resolver
    |
    ├── cached answer available
    |       |
    |       v
    |    return answer
    |
    └── no local answer
            |
            v
        DNS Server
```

This means observing an application use a DNS name does not automatically mean a new DNS packet must appear on the network.

---

## Basic DNS Communication Example

Consider the following network:

```
Workstation
192.168.10.25

DNS servers:
192.168.10.10
192.168.10.11

Default gateway:
192.168.10.1
```

The user wants to access:

```
server01.example.com
```

A simplified process might look like this:

```
1. Application requests server01.example.com.

2. Local resolver checks whether it already
   has usable information.

3. If not, the resolver sends a DNS query
   to a configured DNS server.

4. The DNS server attempts to provide
   the requested information.

5. The answer is returned to the workstation.

6. The resolver may cache the DNS response.

7. The application receives the resolved
   information.

8. The workstation can then attempt
   communication with the destination.
```

The complete process can involve recursive resolvers, root servers, TLD servers, authoritative servers, delegation, caching, and multiple DNS record types.

Those mechanisms are covered in the following DNS topics.

---

## Key Takeaways

The most important concepts from DNS fundamentals are:

- DNS stands for **Domain Name System**.
- DNS provides a distributed naming system used by devices and applications.
- Resolving names to IP addresses is one of the most common DNS functions.
- DNS can store many other types of information besides IP addresses.
- A **DNS resolver** sends DNS queries on behalf of applications.
- A **DNS server** receives and answers DNS queries.
- A **recursive resolver** obtains answers on behalf of clients.
- An **authoritative DNS server** is the source of truth for the DNS data it is responsible for.
- DNS is distributed and hierarchical rather than stored in one global database.
- A **hostname** identifies a device or system name.
- A **domain name** represents a namespace within DNS.
- An **FQDN** represents the complete DNS name of a resource.
- DNS names consist of labels separated by dots.
- The DNS hierarchy becomes more specific from right to left.
- The final dot in an absolute DNS name represents the DNS root.
- DNS traditionally uses **UDP/53** and **TCP/53**; see [TCP and UDP](../../01-fundamentals/09-tcp-udp/tcp-udp.md) and [Ports and Sockets](../../01-fundamentals/10-ports-sockets/ports-sockets.md) for the underlying transport concepts.
- Multiple DNS servers can provide redundancy.
- DNS answers can be cached temporarily.
- DNS cache entries are normally limited by TTL.
- Hosts files and DNS caches are different local name-resolution mechanisms.
- DNS server addresses can be configured manually or provided automatically, commonly through DHCP.
- DNS failure does not automatically mean that the underlying IP network is unavailable.