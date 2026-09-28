# DNS Zones

## Overview

A DNS zone is an administrative container for authoritative DNS data.

It represents the portion of the DNS namespace managed by a particular set of authoritative DNS servers.

A DNS zone can contain records such as:

```
SOA
NS
A
AAAA
CNAME
MX
TXT
SRV
CAA
PTR
```

and many other DNS record types.

For the distinction between domains, subdomains, delegation, and authoritative boundaries, see [DNS Hierarchy](../03-dns-hierarchy/dns-hierarchy.md).

For the individual DNS record types stored inside zones, see [DNS Records](../04-dns-records/dns-records.md).

---

## Domain vs Zone

A **domain** and a **zone** are related, but they are not the same thing.

A domain represents a logical part of the hierarchical DNS namespace.

For example:

```
example.com
```

represents a branch of the DNS hierarchy.

A zone represents the authoritative DNS data managed for some portion of that namespace.

A simple distinction is:

```
Domain
→ hierarchical namespace

Zone
→ administrative container for authoritative DNS data
```

For example:

```
example.com
|
├── www.example.com
├── mail.example.com
└── sales.example.com
```

could all be managed inside one zone:

```
example.com zone
```

In that case, the authoritative DNS servers for `example.com` also store the records for:

```
www.example.com
mail.example.com
sales.example.com
```

However, part of the namespace can be delegated into another zone.

For example:

```
example.com zone
        |
        | delegation
        v
sales.example.com zone
```

`Sales.example.com` is still logically part of the `example.com` namespace, but it now forms a separate administrative and authoritative DNS zone.

---

## Forward Lookup Zones

A **forward lookup zone** contains DNS records that are queried using DNS names.

The most common examples are A and AAAA records.

For example:

```
server01.example.com
→ A
→ 192.0.2.10
```

or:

```
server01.example.com
→ AAAA
→ 2001:db8::10
```

However, forward lookup zones are not limited to name-to-IP mappings.

They can also contain records such as:

```
CNAME
MX
NS
SOA
TXT
SRV
CAA
```

A simplified forward zone might contain:

```
example.com zone

server01   → A     → 192.0.2.10
www        → CNAME → server01.example.com
mail       → A     → 192.0.2.20
@          → MX    → mail.example.com
@          → TXT   → verification information
```

The important concept is:

```
Forward lookup zone
→ queried by DNS name
```

---

## Reverse Lookup Zones

A **reverse lookup zone** is used to map an IP address back to a DNS name.

For example:

```
192.0.2.10
→ PTR
→ server01.example.com
```

This is the opposite direction of a normal forward lookup.

Forward lookup:

```
server01.example.com
        |
        v
A
        |
        v
192.0.2.10
```

Reverse lookup:

```
192.0.2.10
        |
        v
PTR
        |
        v
server01.example.com
```

Reverse DNS uses special namespaces.

For IPv4:

```
in-addr.arpa
```

For IPv6:

```
ip6.arpa
```

An IPv4 address such as:

```
192.0.2.10
```

is represented in reverse DNS using a reversed address structure:

```
10.2.0.192.in-addr.arpa
```

The reason is that DNS hierarchy is processed from right to left.

Reverse DNS is covered in more detail later in [Reverse DNS](../08-reverse-dns/reverse-dns.md).

---

## Zone Apex

The **zone apex** is the top name of a particular zone.

For example, for the zone:

```
example.com
```

the apex is:

```
example.com
```

If:

```
sales.example.com
```

is delegated into its own zone, then:

```
sales.example.com
```

becomes the apex of that child zone.

The zone apex normally contains important authoritative records such as:

```
SOA
NS
```

and may also contain records such as:

```
A
AAAA
MX
TXT
CAA
```

The concept of the zone apex is introduced in [DNS Hierarchy](../03-dns-hierarchy/dns-hierarchy.md).

---

## Primary Authoritative Zones

A **primary zone** contains the writable source of authoritative DNS data.

Administrative changes are normally made to the primary copy.

For example:

```
Primary DNS server
|
└── example.com zone
    ├── SOA
    ├── NS
    ├── A
    ├── MX
    └── TXT
```

If an administrator adds:

```
app.example.com
→ A
→ 192.0.2.40
```

the change is made to the writable zone data.

The primary zone then acts as the source from which secondary authoritative servers can synchronize their copies.

A useful rule is:

```
Primary zone
→ writable source of authoritative zone data
```

---

## Secondary Authoritative Zones

A **secondary zone** contains a synchronized copy of a zone obtained from another authoritative DNS server.

Conceptually:

```
Primary DNS server
example.com zone
        |
        | zone transfer
        v
Secondary DNS server
example.com zone copy
```

The secondary is still authoritative for the zone.

This is important.

A secondary server is not simply a recursive cache.

It can answer authoritative queries for the zone even though administrative changes are not normally made directly to its copy.

Secondary authoritative DNS servers provide benefits such as:

- redundancy,
- availability,
- geographic distribution,
- load distribution,
- resilience if one authoritative server becomes unavailable.

A useful distinction is:

```
Primary
→ writable source

Secondary
→ synchronized authoritative copy
```

---

## Authoritative vs Non-Authoritative Answers

An **authoritative answer** comes from a server that is authoritative for the requested zone.

For example:

```
Authoritative server for example.com
        |
        v
www.example.com
A → 203.0.113.20
```

A **non-authoritative answer** can come from a recursive resolver or cache.

For example:

```
Recursive resolver
        |
        v
cached answer

www.example.com
A → 203.0.113.20
```

The cached answer may still be completely valid.

The difference is where the answer comes from.

A useful rule is:

```
Authoritative
→ original authoritative source for the zone

Non-authoritative
→ answer obtained from another source, commonly cache
```

An important point is:

> Authoritative describes the source of the answer, not whether the answer is correct.

Caching and TTL behavior are introduced in [DNS Fundamentals](../01-dns-fundamentals/dns-fundamentals.md) and [DNS Resolution Process](../02-dns-resolution-process/dns-resolution-process.md).

---

## Zone Data Storage

DNS zone data can be stored in different ways depending on the DNS implementation.

A traditional implementation may use a **zone file**.

A zone file is a structured text representation of the records belonging to a zone.

A simplified example might look like:

```
example.com.        SOA   ...
example.com.        NS    ns1.example.com.
example.com.        MX    10 mail.example.com.

www.example.com.    A     203.0.113.20
mail.example.com.   A     203.0.113.25
```

However, DNS servers do not always store zone information in plain-text files.

Zone data may instead be stored in:

```
database
directory service
internal server database
distributed configuration backend
```

The storage mechanism is an implementation detail.

From a networking perspective, the important concept is that the authoritative server maintains the DNS records for the zone regardless of how they are physically stored.

---

## SOA and Zone Versioning

Each authoritative zone contains an SOA record.

Among other information, the SOA contains a **serial number**.

The serial acts as a version identifier for the zone.

For example:

```
Primary:
Serial 2026092705

Secondary:
Serial 2026092703
```

The secondary can determine that its copy is older than the primary copy.

Conceptually:

```
Secondary checks SOA
        |
        v
Compare serial numbers
        |
        ├── same version
        |      |
        |      v
        |   no update
        |
        └── newer primary version
               |
               v
        synchronize zone
```

Administrators often format serial numbers using a convention such as:

```
YYYYMMDDNN
```

For example:

```
2026092701
```

However, the DNS protocol treats the value as the zone serial rather than as a human-readable date.

The SOA record itself is covered in [DNS Records](../04-dns-records/dns-records.md).

---

## Zone Transfers

Secondary authoritative servers can synchronize zone data using **zone transfers**.

Two important mechanisms are:

```
AXFR
IXFR
```

---

## AXFR

**AXFR** is a full zone transfer.

The complete zone is transferred from one authoritative server to another.

Conceptually:

```
Primary
|
| AXFR
| complete zone
v
Secondary
```

If the zone contains:

```
SOA
NS
A
AAAA
MX
TXT
SRV
```

the full zone data is transferred.

AXFR is simple, but it can be inefficient for large zones when only a small amount of data has changed.

---

## IXFR

**IXFR** is an incremental zone transfer.

Instead of transferring the entire zone, it transfers differences between zone versions.

For example:

```
Primary serial:
2026092703

Secondary serial:
2026092701
```

The secondary can request the changes needed to move from its current version toward the newer zone version.

Conceptually:

```
Changes on primary:

+ new A record
- removed TXT record
~ changed MX record

        |
        | IXFR
        v

Secondary applies changes
```

The key distinction is:

```
AXFR
→ full zone transfer

IXFR
→ differences between zone versions
```

---

## Zone Transfer Security

Zone transfers expose authoritative DNS data.

For this reason, zone transfers should not automatically be available to arbitrary systems.

Authoritative DNS servers commonly restrict zone transfers to specific trusted secondary servers.

Depending on the environment, zone transfer security may include:

- source-address restrictions,
- authenticated DNS transactions,
- access-control rules,
- encrypted or protected transport mechanisms provided by the DNS platform.

Unrestricted zone transfers can expose useful infrastructure information to unauthorized users.

---

## SOA Refresh, Retry, and Expire

The SOA record includes timing information used by secondary DNS servers.

Three important values are:

```
Refresh
Retry
Expire
```

### Refresh

The refresh value indicates how long a secondary normally waits before checking whether a newer version of the zone exists.

Conceptually:

```
Secondary
   |
   | after refresh interval
   v
Check primary SOA serial
```

---

## Retry

If the refresh attempt fails, the retry value indicates how long the secondary waits before trying again.

For example:

```
Refresh attempt
        |
        v
Primary unreachable
        |
        v
Wait retry interval
        |
        v
Try again
```

---

## Expire

The expire value defines the maximum time the secondary can continue serving its current copy of the zone without successfully refreshing it.

For example:

```
Secondary has valid zone copy
        |
        v
Primary becomes unreachable
        |
        v
Refresh attempts fail
        |
        v
Retries continue
        |
        v
Expire interval exceeded
        |
        v
Secondary stops serving
the stale zone authoritatively
```

The important point is that the secondary should not serve an increasingly stale copy forever.

It does not necessarily mean that all zone records are immediately deleted from disk.

The important DNS behavior is that the stale copy is no longer considered valid authoritative zone data.

A useful summary is:

```
Refresh
→ when to check for updates

Retry
→ when to try again after failure

Expire
→ how long stale zone data may still be served
```

---

## Dynamic DNS Updates

DNS zone data does not always need to be changed manually.

**Dynamic DNS updates** allow authorized systems to automatically add, modify, or remove records.

For example, a client might receive a new address from DHCP:

```
client01.example.com
→ 192.0.2.25
```

The corresponding DNS record can then be updated automatically.

Conceptually:

```
Client receives address
        |
        v
Dynamic DNS update
        |
        v
Authoritative DNS server
        |
        v
Zone data updated
```

The update may be sent directly by the client.

For example:

```
Client
→ authoritative DNS server
```

or another system such as a DHCP server can update DNS on the client's behalf:

```
DHCP server
→ authoritative DNS server
```

The DHCP service may also update corresponding reverse DNS PTR records.

Dynamic update is a DNS capability.

It does not necessarily mean that a separate "DDNS server" exists.

### Dynamic Update Security

Allowing arbitrary systems to modify DNS records would create a major security problem.

Dynamic updates should therefore be restricted to authorized systems.

Depending on the DNS platform, this can involve:

- authenticated updates,
- access-control policies,
- trusted DHCP servers,
- client identity,
- zone-specific update permissions.

---

## Delegation and Child Zones

A subdomain does not automatically create a new DNS zone.

For example:

```
example.com
|
└── sales.example.com
```

can remain entirely inside the `example.com` zone.

In that case:

```
example.com zone
|
└── sales.example.com records
```

are managed by the same authoritative infrastructure.

However, administrators can decide that:

```
sales.example.com
```

should have its own authoritative zone.

Conceptually:

```
example.com zone
        |
        | delegation
        v
sales.example.com zone
```

The parent zone keeps the delegation information.

The child zone contains the authoritative data below that boundary.

Reasons for separating a child zone can include:

- different administrative teams,
- organizational boundaries,
- independent DNS infrastructure,
- different security requirements,
- scaling,
- operational separation.

The important rule is:

```
Subdomain
→ logical DNS namespace

Delegated child zone
→ separate administrative and authoritative boundary
```

Delegation is covered in more detail in [DNS Hierarchy](../03-dns-hierarchy/dns-hierarchy.md).

---

## Split DNS / Split-Horizon DNS

**Split DNS**, also called **split-horizon DNS**, allows the same DNS name to return different answers depending on the source or context of the query.

For example:

```
portal.example.com
```

might resolve differently for internal and external users.

Internal query:

```
portal.example.com
        |
        v
10.10.20.50
```

External query:

```
portal.example.com
        |
        v
203.0.113.50
```

Conceptually:

```
                    portal.example.com
                           |
               +-----------+-----------+
               |                       |
        Internal query            External query
               |                       |
               v                       v
         10.10.20.50              203.0.113.50
```

Split DNS can be implemented in several ways depending on the DNS platform.

Examples include:

- separate internal and public zones,
- DNS views,
- policy-based answers,
- separate authoritative infrastructure.

The implementation method is platform-specific.

From a networking perspective, the important concept is:

> The same DNS name can intentionally return different DNS records depending on where or how the query is received.

---

## Internal and Public Zone Data

An organization can maintain DNS data that is intended only for internal systems.

For example:

```
db01.example.com
→ 10.10.30.20
```

may only be useful inside the organization.

Public authoritative DNS might instead contain only internet-facing names such as:

```
www.example.com
mail.example.com
vpn.example.com
```

Separating public and internal DNS information can help:

- avoid exposing internal addressing,
- simplify internal service discovery,
- support split DNS,
- separate administrative responsibilities,
- apply different security policies.

Internal and public DNS design is covered is [Internal and Public DNS](../09-internal-and-public-dns/internal-and-public-dns.md).

---

## Zone Design Considerations

DNS zones should be divided based on administrative and operational requirements rather than simply because subdomains exist.

A small environment may use:

```
example.com
```

as one zone containing all records.

A larger organization might use:

```
example.com
|
├── corp.example.com
├── dev.example.com
├── sales.example.com
└── europe.example.com
```

with some or all of those subdomains delegated into separate zones.

A separate zone can be useful when it requires:

- independent administration,
- different authoritative servers,
- separate lifecycle management,
- different security controls,
- different update policies,
- operational independence.

However, creating too many zones increases administrative complexity.

The design should balance:

```
simplicity
vs
administrative separation
```

---

## Example Zone Design

Consider the domain:

```
example.com
```

A small environment might use:

```
example.com zone
|
├── www.example.com
├── mail.example.com
├── vpn.example.com
├── server01.example.com
└── sales.example.com
```

Everything is managed by the same authoritative DNS infrastructure.

A larger environment might instead use:

```
example.com zone
|
├── www.example.com
├── mail.example.com
|
├── delegation
|   |
|   v
|   sales.example.com zone
|
└── delegation
    |
    v
    dev.example.com zone
```

Each delegated child zone can then have its own:

```
SOA
NS
records
administrators
authoritative servers
update policies
```

while still remaining part of the same overall DNS namespace.

---

## Zone Lifecycle Example

A simplified secondary-zone synchronization process can look like:

```
Primary zone exists
        |
        v
Secondary receives zone
through AXFR
        |
        v
Secondary serves authoritative answers
        |
        v
Refresh interval reached
        |
        v
Secondary checks SOA serial
        |
        +-------------------------+
        |                         |
        v                         v
Serial unchanged             Serial changed
        |                         |
        v                         v
No update                    Request IXFR
                                  |
                                  v
                           Apply zone changes
                                  |
                                  v
                         Continue serving zone
```

If the primary cannot be contacted:

```
Refresh fails
      |
      v
Wait Retry interval
      |
      v
Try again
      |
      v
Repeated failures
      |
      v
Expire interval reached
      |
      v
Stop serving stale zone
authoritatively
```

---

## Key Takeaways

- A domain is part of the hierarchical DNS namespace.
- A zone is an administrative container for authoritative DNS data.
- A domain and a zone do not always cover exactly the same part of the namespace.
- A forward lookup zone is queried using DNS names.
- Forward zones can contain much more than A and AAAA records.
- A reverse lookup zone maps IP addresses back to DNS names using PTR records.
- IPv4 reverse DNS uses `in-addr.arpa`.
- IPv6 reverse DNS uses `ip6.arpa`.
- The zone apex is the top name of a particular DNS zone.
- A primary zone contains the writable source of authoritative DNS data.
- A secondary zone contains a synchronized authoritative copy of the zone.
- Secondary servers are authoritative, not merely caches.
- Authoritative describes the source of an answer, not whether the answer is correct.
- Zone data may be stored in text files, databases, directory services, or other backends.
- The SOA serial number acts as a version identifier for the zone.
- AXFR transfers the complete zone.
- IXFR transfers differences between zone versions.
- Zone transfers should be restricted to authorized systems.
- SOA Refresh determines when a secondary checks for updates.
- SOA Retry determines when another attempt is made after a failed refresh.
- SOA Expire determines how long a secondary may continue serving stale zone data without successful synchronization.
- Dynamic DNS allows authorized systems to automatically add, modify, or remove DNS records.
- Dynamic updates can be performed by clients or by systems such as DHCP servers.
- A subdomain does not automatically require a separate DNS zone.
- Delegation creates a separate authoritative and administrative boundary.
- Split DNS can return different answers for the same DNS name depending on query source or context.
- DNS zone design should balance simplicity with administrative, security, and operational separation.