# DNS Records

## Overview

DNS records are the individual pieces of information stored inside DNS zones.

They describe different kinds of data associated with DNS names.

The general purpose of DNS and the idea that DNS stores more than simple name-to-IP mappings are introduced in [DNS Fundamentals](../01-dns-fundamentals/dns-fundamentals.md).

The process used to retrieve records during DNS resolution is covered in [DNS Resolution Process](../02-dns-resolution-process/dns-resolution-process.md).

Delegation, zones, zone apexes, and authoritative responsibility are covered in [DNS Hierarchy](../03-dns-hierarchy/dns-hierarchy.md).

This topic focuses on the most important DNS record types commonly encountered in real environments.

The main records covered here are:

```
A
AAAA
CNAME
MX
NS
SOA
PTR
TXT
SRV
CAA
```

---

## DNS Records as Structured Data

A DNS name can have one or more records associated with it.

For example:

```
www.example.com
```

might contain:

```
A
→ 203.0.113.20

AAAA
→ 2001:db8::20
```

while:

```
example.com
```

might contain:

```
MX
TXT
CAA
NS
SOA
```

Different record types serve different purposes.

A useful model is:

```
DNS Name
   |
   ├── A
   ├── AAAA
   ├── MX
   ├── TXT
   ├── CAA
   └── other record types
```

---

## A Record

An **A record** maps a DNS name to an IPv4 address.

For example:

```
www.example.com
→ A
→ 203.0.113.20
```

This allows a resolver to obtain an IPv4 address for the requested name.

Conceptually:

```
www.example.com
        |
        v
A record
        |
        v
203.0.113.20
```

The meaning of the label itself is not enforced by DNS.

For example:

```
www.example.com
```

is commonly used for a web service, but DNS only treats it as a name associated with a record.

### Multiple A Records

A DNS name can have multiple A records.

For example:

```
www.example.com

A → 203.0.113.20
A → 203.0.113.21
A → 203.0.113.22
```

This can be used to make multiple IPv4 endpoints available under the same DNS name.

The exact client behavior depends on the resolver, application, and service design.

---

## AAAA Record

An **AAAA record** maps a DNS name to an IPv6 address.

For example:

```
www.example.com
→ AAAA
→ 2001:db8:100::20
```

The purpose is similar to an A record, but for IPv6.

A simple rule is:

```
A
→ IPv4

AAAA
→ IPv6
```

A name can have both record types at the same time.

For example:

```
www.example.com

A
→ 203.0.113.20

AAAA
→ 2001:db8:100::20
```

This is common in dual-stack environments where the same service is reachable through both IPv4 and IPv6.

IPv4 and IPv6 addressing are covered in more detail in:

- [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md)
- [IPv6 Basics](../../01-fundamentals/06-ipv6-basics/ipv6-basics.md)

---

## CNAME Record

A **CNAME record** creates an alias from one DNS name to another DNS name.

It does not directly point to an IP address.

For example:

```
www.example.com
→ CNAME
→ web01.example.net
```

The resolver must then resolve:

```
web01.example.net
```

which may eventually return an A or AAAA record.

Conceptually:

```
www.example.com
      |
      v
CNAME
      |
      v
web01.example.net
      |
      v
A / AAAA
      |
      v
IP address
```

A useful rule is:

```
CNAME
→ alias to another DNS name
```

This is useful when multiple names should refer to another canonical name without duplicating address records.

### CNAME and Other Records

A name that contains a CNAME generally cannot also contain normal DNS data such as:

```
A
AAAA
MX
TXT
```

at the same owner name.

For example, this is problematic:

```
www.example.com
CNAME → web01.example.net

www.example.com
A → 203.0.113.20
```

The reason is that a CNAME means:

> This name is an alias; use the target name for the actual DNS data.

CNAME behavior during resolution is also discussed in [DNS Resolution Process](../02-dns-resolution-process/dns-resolution-process.md).

---

## MX Record

An **MX record** identifies which mail server or servers are responsible for receiving email for a domain.

For example, email sent to:

```
user@example.com
```

requires the sending mail system to discover where mail for:

```
example.com
```

should be delivered.

DNS might contain:

```
example.com
→ MX 10 mail1.example.com
→ MX 20 mail2.example.com
```

The number is a **preference value**.

Lower values are normally preferred.

For example:

```
10
→ preferred first

20
→ lower preference
```

The MX record points to a hostname, not directly to an IP address.

The sending system still needs to resolve:

```
mail1.example.com
```

using A and/or AAAA records.

Conceptually:

```
user@example.com
      |
      v
Query MX for example.com
      |
      v
mail1.example.com
      |
      v
A / AAAA
      |
      v
Mail server IP address
```

A useful rule is:

```
MX
→ identifies the mail server by name

A / AAAA
→ provides the address used to reach it
```

---

## NS Record

An **NS record** identifies authoritative DNS servers for a zone or delegated namespace.

For example:

```
example.com
→ NS ns1.example.net
→ NS ns2.example.net
```

This tells resolvers:

> These name servers are authoritative for `example.com`.

NS records are central to DNS delegation.

For example:

```
example.com
    |
    | NS delegation
    v
sales.example.com
```

The role of NS records in parent-child delegation is covered in detail in [DNS Hierarchy](../03-dns-hierarchy/dns-hierarchy.md).

Like MX records, NS records normally point to DNS names rather than directly to IP addresses.

The resolver must still obtain address information for the name server before contacting it.

---

## SOA Record

SOA stands for:

```
Start of Authority
```

The **SOA record** contains key administrative and synchronization information for a DNS zone.

It exists at the zone apex.

For example:

```
Zone:
example.com

Apex:
example.com
```

The SOA record can contain information such as:

```
Primary authoritative DNS server
Responsible administrator contact
Zone serial number
Refresh interval
Retry interval
Expire interval
Negative caching-related timing
```

A simplified conceptual example is:

```
example.com
→ SOA

Primary server:
ns1.example.com

Responsible contact:
hostmaster.example.com

Serial:
2026092701

Refresh:
3600

Retry:
600

Expire:
1209600
```

### Serial Number

The SOA serial number allows secondary authoritative DNS servers to determine whether the zone data has changed.

For example:

```
Primary DNS Server
Serial:
2026092701

Secondary DNS Server
Serial:
2026092601
```

The secondary can determine that newer zone data exists.

Conceptually:

```
Secondary detects newer serial
        |
        v
Zone data needs to be updated
```

The details of zone transfers and authoritative DNS replication are outside the scope of this record overview.

The important rule is:

```
SOA
→ administrative and synchronization
  information for a DNS zone
```

---

## PTR Record

A **PTR record** is used for reverse DNS resolution.

Instead of asking:

```
server01.example.com
→ which IP address?
```

a reverse lookup asks:

```
192.168.10.50
→ which DNS name?
```

Conceptually:

```
Forward lookup:

server01.example.com
        |
        v
A / AAAA
        |
        v
192.168.10.50
```

while:

```
Reverse lookup:

192.168.10.50
        |
        v
PTR
        |
        v
server01.example.com
```

For IPv4, reverse DNS uses:

```
in-addr.arpa
```

For IPv6, reverse DNS uses:

```
ip6.arpa
```

The complete reverse DNS structure is covered later in [Reverse DNS](../08-reverse-dns/reverse-dns.md).

A useful rule is:

```
PTR
→ reverse lookup
→ IP address to DNS name
```

---

## TXT Record

A **TXT record** stores text data associated with a DNS name.

TXT records are flexible and are widely used by external services and security mechanisms.

Common uses include:

- domain ownership verification,
- service verification tokens,
- SPF-related information,
- DKIM public-key information,
- DMARC policy information,
- other application-specific metadata.

A simple example is:

```
example.com
→ TXT
→ "verification-token"
```

Another example could be:

```
example.com
→ TXT
→ "v=spf1 ..."
```

TXT records are not limited to the zone apex.

Different systems can create TXT records beneath specific labels.

For example, email security mechanisms often use dedicated names under the domain.

A useful rule is:

```
TXT
→ arbitrary text data associated
  with a DNS name
```

---

## SRV Record

An **SRV record** is used for service discovery.

It can tell clients:

- which service is available,
- which transport protocol is used,
- which host provides the service,
- which port should be used,
- which server has higher priority,
- how traffic can be distributed between servers of equal priority.

The general format is conceptually:

```
_service._protocol.domain
```

For example:

```
_ldap._tcp.example.com
```

could describe an LDAP service over TCP.

A simplified SRV record might contain:

```
Priority: 10
Weight:   50
Port:     389
Target:   server01.example.com
```

Conceptually:

```
_ldap._tcp.example.com
        |
        v
SRV
        |
        ├── target host
        ├── port
        ├── priority
        └── weight
```

### Priority

Lower priority values are normally preferred.

For example:

```
Priority 10
→ preferred before

Priority 20
```

### Weight

Weight can help distribute requests between multiple targets that have the same priority.

SRV records are widely used in enterprise environments.

For example, Active Directory uses SRV records to help clients locate services such as:

- LDAP,
- Kerberos,
- domain controllers.

A useful rule is:

```
SRV
→ service discovery
→ host + port + priority + weight
```

---

## CAA Record

CAA stands for:

```
Certification Authority Authorization
```

A **CAA record** allows a domain owner to publish which Certificate Authorities are permitted to issue certificates for that DNS namespace.

For example:

```
example.com
CAA 0 issue "letsencrypt.org"
```

This indicates that Let's Encrypt is authorized to issue certificates for the domain according to the published CAA policy.

CAA records can include properties such as:

```
issue
```

which identifies Certificate Authorities allowed to issue normal certificates.

```
issuewild
```

which identifies Certificate Authorities allowed to issue wildcard certificates.

```
iodef
```

which can provide reporting or contact information related to CAA policy processing.

The important distinction is:

> CAA does not issue or install certificates.

It only publishes certificate-issuance policy through DNS.

A useful rule is:

```
CAA
→ certificate authority authorization policy
```

---

## Multiple Records for the Same DNS Name

A single DNS name can contain multiple records.

For example:

```
www.example.com

A
→ 203.0.113.20

A
→ 203.0.113.21

AAAA
→ 2001:db8::20
```

These records do not need to represent exactly the same address or protocol family.

Multiple records can coexist when DNS rules allow those record types to exist together.

---

## Record Sets / RRSets

Multiple records of the same type for the same owner name are commonly treated as a **record set**, also called an **RRset**.

For example:

```
www.example.com

A → 203.0.113.20
A → 203.0.113.21
A → 203.0.113.22
```

These A records form one A RRset for:

```
www.example.com
```

The same idea applies to other record types.

For example:

```
example.com

MX 10 mail1.example.com
MX 20 mail2.example.com
```

forms an MX record set.

---

## Compatible and Incompatible Record Types

Different record types can coexist at the same DNS name.

For example:

```
example.com

A
AAAA
MX
TXT
CAA
```

can all be associated with the same name.

However, some DNS record types have specific compatibility rules.

The most important example is CNAME.

If:

```
www.example.com
→ CNAME web01.example.net
```

then the name is acting as an alias.

It generally should not also contain unrelated normal data such as:

```
A
AAAA
MX
TXT
```

at that same owner name.

This is one of the most important practical rules to remember when designing DNS records.

---

## Record Type Relationships

Several DNS record types commonly depend on other DNS records.

### MX

```
example.com
→ MX mail.example.com
```

requires the target name to be resolvable:

```
mail.example.com
→ A / AAAA
```

### NS

```
example.com
→ NS ns1.example.net
```

requires the resolver to obtain an address for:

```
ns1.example.net
```

unless appropriate glue or cached information is already available.

### CNAME

```
www.example.com
→ CNAME web01.example.net
```

requires the resolver to continue resolving:

```
web01.example.net
```

until usable final data is obtained.

### SRV

```
_ldap._tcp.example.com
→ SRV server01.example.com
```

requires the target hostname to be resolved before the client can contact the service.

This illustrates an important DNS principle:

> Many records identify another DNS name rather than directly identifying an IP address.

---

## Common Record Summary

```
A
→ DNS name to IPv4 address

AAAA
→ DNS name to IPv6 address

CNAME
→ alias to another DNS name

MX
→ mail server for a domain

NS
→ authoritative DNS server

SOA
→ zone administrative and synchronization information

PTR
→ reverse lookup from IP address to DNS name

TXT
→ arbitrary text, verification, and policy information

SRV
→ service discovery including target host and port

CAA
→ certificate authority authorization policy
```

---

## Other DNS Record Types

DNS contains many additional record types.

Some examples include:

```
NAPTR
SVCB
HTTPS
TLSA
```

These records can support specialized functions such as:

- service discovery,
- modern application endpoint discovery,
- encrypted service metadata,
- certificate-related mechanisms,
- protocol-specific discovery.

They are not covered in detail here because the main purpose of this topic is to establish the DNS records most commonly encountered in general networking and systems administration.

Additional record types can be found in appendix to this article [DNS Records Types](../04-dns-records/appendix/dns-records-types.md).

---

## Practical Example

Consider:

```
example.com
```

A simplified DNS configuration might contain:

```
example.com
→ A 203.0.113.10

example.com
→ MX 10 mail.example.com

example.com
→ TXT "verification-data"

example.com
→ CAA 0 issue "letsencrypt.org"

www.example.com
→ CNAME web01.example.net

mail.example.com
→ A 203.0.113.25

_ldap._tcp.example.com
→ SRV 10 50 389 server01.example.com

server01.example.com
→ A 203.0.113.30
```

This example demonstrates that DNS is not simply:

```
name
→ IP address
```

Instead, DNS stores structured information used for:

- addressing,
- aliases,
- mail routing,
- service discovery,
- verification,
- certificate policy,
- authoritative delegation,
- reverse resolution.

---

## Key Takeaways

- DNS records are the individual pieces of data stored inside DNS zones.
- Different DNS record types represent different kinds of information.
- An A record maps a DNS name to an IPv4 address.
- An AAAA record maps a DNS name to an IPv6 address.
- A name can have both A and AAAA records.
- A name can also have multiple A or multiple AAAA records.
- A CNAME creates an alias from one DNS name to another DNS name.
- A CNAME does not directly point to an IP address.
- A CNAME generally cannot coexist with normal DNS data at the same owner name.
- An MX record identifies mail servers for a domain.
- Lower MX preference values are normally preferred.
- MX records point to hostnames, which must then be resolved to IP addresses.
- An NS record identifies authoritative DNS servers.
- NS records are central to DNS delegation.
- An SOA record contains administrative and synchronization information for a DNS zone.
- The SOA serial number helps secondary DNS servers detect zone changes.
- A PTR record is used for reverse DNS resolution.
- IPv4 reverse DNS uses `in-addr.arpa`.
- IPv6 reverse DNS uses `ip6.arpa`.
- TXT records store arbitrary text used for verification, policies, and application-specific information.
- SRV records support service discovery and can identify a target host, port, priority, and weight.
- CAA records publish which Certificate Authorities are authorized to issue certificates for a domain.
- One DNS name can have multiple compatible record types.
- Multiple records of the same type for the same name form an RRset.
- Many DNS records point to other DNS names rather than directly to IP addresses.