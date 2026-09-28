# Reverse DNS

## Overview

Reverse DNS is used to map an IP address back to a DNS name.

Normal forward DNS typically answers questions such as:

```
server01.example.com
→ which IP address?
```

Reverse DNS answers the opposite question:

```
192.0.2.10
→ which DNS name?
```

The record type used for reverse DNS is:

```
PTR
```

For example:

```
192.0.2.10
→ PTR
→ server01.example.com
```

PTR records were introduced in [DNS Records](../04-dns-records/dns-records.md).

Reverse lookup zones were introduced in [DNS Zones](../05-dns-zones/dns-zones.md).

This topic focuses on how reverse DNS is structured, delegated, managed, and used in practice.

---

## Forward DNS vs Reverse DNS

Forward and reverse DNS perform opposite mappings.

Forward DNS:

```
server01.example.com
        |
        v
A / AAAA
        |
        v
192.0.2.10
```

Reverse DNS:

```
192.0.2.10
        |
        v
PTR
        |
        v
server01.example.com
```

A useful rule is:

```
Forward DNS
→ name to address

Reverse DNS
→ address to name
```

The two systems are related, but they are maintained independently.

---

## PTR Records

A PTR record maps a reverse-DNS name to another DNS name.

For example:

```
10.2.0.192.in-addr.arpa
→ PTR
→ server01.example.com
```

This allows a resolver to start with:

```
192.0.2.10
```

and obtain:

```
server01.example.com
```

The PTR record normally points to a DNS hostname.

It does not automatically create or verify a corresponding A or AAAA record.

---

## IPv4 Reverse DNS

IPv4 reverse DNS uses the namespace:

```
in-addr.arpa
```

IPv4 addresses are represented in reverse order.

For example:

```
192.0.2.10
```

becomes:

```
10.2.0.192.in-addr.arpa
```

A PTR record can then exist at that name:

```
10.2.0.192.in-addr.arpa
→ PTR
→ server01.example.com
```

---

## Why IPv4 Octets Are Reversed

DNS hierarchy is read from **right to left**.

For example:

```
www.sales.example.com
```

can be viewed hierarchically as:

```
.
└── com
    └── example
        └── sales
            └── www
```

IPv4 addressing is normally written from the broader network portion toward the more specific host portion.

For example:

```
192.0.2.10
```

To fit IPv4 addressing into the DNS hierarchy, the octets are reversed.

Conceptually:

```
192.0.2.10
        |
        v
10.2.0.192.in-addr.arpa
```

The hierarchy then becomes:

```
arpa
 |
 v
in-addr
 |
 v
192
 |
 v
0
 |
 v
2
 |
 v
10
```

This makes reverse-zone delegation possible using the DNS hierarchy.

---

## IPv4 Reverse Zones

A `/24` IPv4 network fits naturally into the reverse DNS structure.

For example:

```
192.0.2.0/24
```

corresponds naturally to:

```
2.0.192.in-addr.arpa
```

Individual addresses exist underneath that zone.

For example:

```
10.2.0.192.in-addr.arpa
→ PTR
→ server01.example.com
```

and:

```
20.2.0.192.in-addr.arpa
→ PTR
→ mail.example.com
```

Conceptually:

```
2.0.192.in-addr.arpa
|
├── 10 → server01.example.com
├── 20 → mail.example.com
└── 30 → firewall01.example.com
```

---

## IPv6 Reverse DNS

IPv6 reverse DNS uses:

```
ip6.arpa
```

Unlike IPv4, IPv6 reverse DNS does not reverse whole octets or complete hextets.

It operates on individual hexadecimal digits.

Each hexadecimal digit represents:

```
4 bits
```

This is called a:

```
nibble
```

So IPv6 reverse DNS is often described as **nibble-based reverse DNS**.

---

## IPv6 Reverse Lookup Example

Consider:

```
2001:db8::1
```

First, the IPv6 address is expanded fully:

```
2001:0db8:0000:0000:0000:0000:0000:0001
```

Each hexadecimal digit is then treated individually.

Conceptually:

```
2 0 0 1 0 d b 8 0 0 0 0 ... 0 0 0 1
```

The digits are reversed and separated by dots under:

```
ip6.arpa
```

The resulting reverse-DNS name therefore begins conceptually as:

```
1.0.0.0....8.b.d.0.1.0.0.2.ip6.arpa
```

The complete representation is long because every hexadecimal digit becomes a DNS label.

The important rule is:

```
IPv4 reverse DNS
→ reverse octets

IPv6 reverse DNS
→ reverse hexadecimal nibbles
```

Using nibbles allows IPv6 reverse zones to be delegated along 4-bit boundaries.

---

## Forward and Reverse DNS Are Independent

Creating a forward record does not automatically create a reverse record.

For example:

```
mail.example.com
→ A
→ 203.0.113.25
```

can exist without:

```
203.0.113.25
→ PTR
→ mail.example.com
```

The opposite is also possible.

A PTR record can exist while the corresponding forward record is missing or points somewhere else.

So:

```
A / AAAA
→ forward DNS

PTR
→ reverse DNS
```

are maintained independently.

---

## Who Controls Reverse DNS?

Forward DNS is normally controlled by whoever manages the domain.

For example, the administrator of:

```
example.com
```

can create:

```
www.example.com
→ A
→ 203.0.113.25
```

Reverse DNS is different.

Authority over reverse DNS normally follows control or delegation of the IP address block.

For example, if:

```
203.0.113.25
```

belongs to an address block controlled by an ISP or cloud provider, that provider normally controls the corresponding public reverse DNS namespace unless it delegates that responsibility.

Conceptually:

```
ISP / address-block authority
        |
        v
reverse DNS zone
        |
        v
PTR records
```

This is why service providers often expose settings such as:

```
Reverse DNS
PTR record
rDNS hostname
```

The customer chooses the desired PTR hostname, but the provider publishes it in the reverse zone it controls.

---

## Forward DNS Does Not Prove Address Ownership

A domain administrator can point a DNS name toward an IP address:

```
www.example.com
→ 203.0.113.25
```

but that does not mean the domain owner controls that address.

It also does not automatically establish:

- TLS,
- HTTPS,
- service ownership,
- routing,
- server identity.

DNS only provides naming information.

Other protocols and security mechanisms determine whether the service itself is legitimate, reachable, encrypted, and correctly authenticated.

---

## Forward-Confirmed Reverse DNS

A common consistency check is to compare reverse DNS with forward DNS.

For example:

```
203.0.113.25
        |
        v
PTR
        |
        v
mail.example.com
```

and then resolve that hostname:

```
mail.example.com
        |
        v
A
        |
        v
203.0.113.25
```

The lookup returns to the original address.

This is commonly referred to as:

```
Forward-Confirmed Reverse DNS
FCrDNS
```

Conceptually:

```
IP address
    |
    v
PTR
    |
    v
hostname
    |
    v
A / AAAA
    |
    v
original IP address
```

FCrDNS provides consistency between forward and reverse DNS.

---

## Reverse DNS and Email

Reverse DNS is particularly important for mail servers.

Receiving mail systems may inspect the sending server's:

- PTR record,
- hostname,
- forward DNS,
- SMTP identity,
- reputation.

For example:

```
203.0.113.25
→ PTR
→ mail.example.com
```

and:

```
mail.example.com
→ A
→ 203.0.113.25
```

is a more consistent configuration than having unrelated forward and reverse names.

However:

> Reverse DNS does not authenticate an email sender by itself.

A malicious or untrusted server can also have valid PTR records.

Email authentication depends on additional mechanisms such as:

```
SPF
DKIM
DMARC
```

Reverse DNS is therefore better understood as one of several reputation, consistency, and validation signals.

---

## Reverse DNS in Logs and Monitoring

Reverse DNS can make IP-based information easier for administrators to understand.

For example, a firewall log might contain:

```
192.0.2.25
```

Without reverse DNS, the administrator must determine what system uses that address.

With a PTR record:

```
192.0.2.25
→ PTR
→ backup01.example.com
```

the address becomes easier to identify.

Reverse DNS can therefore help with:

- firewall logs,
- monitoring,
- SIEM analysis,
- incident response,
- packet analysis,
- troubleshooting,
- traceroute output,
- service identification.

---

## Reverse DNS Is Not Proof of Identity

PTR records are still DNS records.

A PTR response such as:

```
192.0.2.25
→ backup01.example.com
```

means that the reverse DNS infrastructure publishes that mapping.

It does not prove that the device is genuinely the expected backup server.

Therefore, reverse DNS should not be used as the only authentication or authorization mechanism.

A useful rule is:

```
PTR
→ useful identification information

PTR
≠ strong proof of identity
```

---

## Reverse Zone Delegation

An upstream organization can delegate a reverse DNS zone to another organization's authoritative DNS servers.

For example, suppose an organization manages:

```
192.0.2.0/24
```

The corresponding reverse zone is:

```
2.0.192.in-addr.arpa
```

Instead of asking the upstream provider to maintain every PTR record, the provider can delegate the reverse zone.

Conceptually:

```
Upstream provider
controls parent reverse namespace
        |
        | delegation
        v
Organization's DNS servers
control:
2.0.192.in-addr.arpa
```

The organization's administrators can then manage their own PTR records:

```
192.0.2.10
→ server01.example.com

192.0.2.20
→ mail.example.com

192.0.2.30
→ firewall01.example.com
```

This allows:

- self-service administration,
- automation,
- faster changes,
- reduced provider overhead,
- independent authoritative management.

Reverse delegation follows the same general DNS delegation principles described in [DNS Hierarchy](../03-dns-hierarchy/dns-hierarchy.md).

---

## The IPv4 Classless Delegation Problem

IPv4 reverse DNS naturally fits octet boundaries.

For example:

```
192.0.2.0/24
```

maps cleanly to:

```
2.0.192.in-addr.arpa
```

But smaller address ranges do not align naturally with the standard reverse DNS hierarchy.

For example:

```
192.0.2.0/28
```

contains:

```
192.0.2.0
through
192.0.2.15
```

However:

```
2.0.192.in-addr.arpa
```

represents the entire:

```
192.0.2.0/24
```

The provider cannot simply delegate that full reverse zone to the owner of only the `/28`.

---

## RFC 2317 Classless Reverse Delegation

A common solution for IPv4 ranges smaller than `/24` is **classless reverse DNS delegation**, commonly associated with RFC 2317.

The upstream provider keeps authority over the normal parent reverse zone.

For example:

```
2.0.192.in-addr.arpa
```

It then creates an additional namespace representing the customer's smaller range.

For example:

```
0-15.2.0.192.in-addr.arpa
```

The exact naming convention can vary.

The provider can use CNAME records in the parent reverse zone to redirect individual reverse names into the delegated namespace.

Conceptually:

```
10.2.0.192.in-addr.arpa
        |
        v
CNAME
        |
        v
10.0-15.2.0.192.in-addr.arpa
        |
        v
PTR
        |
        v
server01.example.com
```

This allows the customer to manage reverse DNS for only the addresses assigned to them.

A useful distinction is:

```
/24
→ naturally aligns with IPv4 reverse DNS

smaller prefixes such as /25, /26, /27, /28
→ may require classless reverse delegation
```

---

## Public vs Internal Reverse DNS

An organization can maintain reverse DNS for both public and private address spaces.

### Public Reverse DNS

Public reverse DNS usually depends on delegation from:

- an ISP,
- a hosting provider,
- a cloud provider,
- a regional Internet registry allocation chain.

Example:

```
203.0.113.25
→ PTR
→ mail.example.com
```

### Internal Reverse DNS

Inside a private network, the organization normally controls its own reverse zones.

For example:

```
10.10.20.25
→ PTR
→ workstation25.corp.example.com
```

An internal DNS environment might contain reverse zones corresponding to private subnets such as:

```
10.10.10.0/24
10.10.20.0/24
10.10.30.0/24
```

These can improve:

- administration,
- monitoring,
- troubleshooting,
- log readability,
- service identification.

---

## Reverse DNS and DHCP

In dynamically addressed environments, DHCP and DNS can work together.

For example:

```
Client receives:
10.10.20.25
```

The DNS infrastructure may update:

```
Forward record:

client01.example.com
→ A
→ 10.10.20.25
```

and:

```
Reverse record:

10.10.20.25
→ PTR
→ client01.example.com
```

Depending on the environment, the client or DHCP server can perform the dynamic DNS update.

Dynamic DNS updates are introduced in [DNS Zones](../05-dns-zones/dns-zones.md).

---

## Testing Reverse DNS with nslookup

`nslookup` can perform a reverse lookup by querying an IP address directly.

For example:

```
nslookup 203.0.113.25
```

A successful reverse lookup might return:

```
Name:    mail.example.com
Address: 203.0.113.25
```

Conceptually:

```
nslookup 203.0.113.25
        |
        v
PTR lookup
        |
        v
mail.example.com
```

The exact output format depends on the operating system and implementation.

---

## Testing Reverse DNS with dig

On systems with `dig`, the `-x` option performs a reverse lookup.

For IPv4:

```
dig -x 203.0.113.25
```

`dig` automatically converts the address into the correct:

```
in-addr.arpa
```

name and queries the PTR record.

Conceptually:

```
203.0.113.25
        |
        v
25.113.0.203.in-addr.arpa
        |
        v
PTR query
        |
        v
mail.example.com
```

For IPv6:

```
dig -x 2001:db8::25
```

`dig` automatically creates the required nibble-reversed:

```
ip6.arpa
```

query.

This avoids manually constructing the long IPv6 reverse-DNS name.

---

## Testing Forward and Reverse Consistency

A useful troubleshooting method is to test both directions.

First:

```
dig -x 203.0.113.25
```

Suppose the result is:

```
mail.example.com
```

Then resolve the returned hostname:

```
dig mail.example.com A
```

If the result contains:

```
203.0.113.25
```

the forward and reverse mappings confirm each other.

Conceptually:

```
203.0.113.25
        |
        v
PTR
        |
        v
mail.example.com
        |
        v
A
        |
        v
203.0.113.25
```

---

## Common Reverse DNS Problems

Common reverse DNS issues include:

```
No PTR record
```

The address has no reverse mapping.

```
PTR points to outdated hostname
```

The reverse zone was not updated after infrastructure changes.

```
PTR hostname has no A/AAAA record
```

Reverse lookup succeeds, but forward confirmation fails.

```
PTR and A/AAAA point to different systems
```

Forward and reverse DNS are inconsistent.

```
Wrong organization controls reverse DNS
```

The local administrator cannot directly change public PTR records because the provider controls the reverse zone.

```
Reverse zone not delegated
```

The organization expects to manage PTR records but has not received authority over the reverse namespace.

```
Classless subnet delegation missing
```

A subnet smaller than `/24` requires an appropriate classless reverse-DNS design.

---

## Reverse DNS Troubleshooting Flow

A simplified troubleshooting process can look like:

```
Start with IP address
        |
        v
Query PTR record
        |
        +---------------------+
        |                     |
        v                     v
PTR exists                No PTR
        |                     |
        v                     v
Resolve returned          Check reverse-zone
hostname                  ownership/delegation
        |
        v
Compare A/AAAA result
with original address
        |
        +---------------------+
        |                     |
        v                     v
Matches               Does not match
        |                     |
        v                     v
Consistent             Investigate DNS
forward/reverse         configuration
```

---

## Practical Example

Consider a public mail server:

```
mail.example.com
```

Forward DNS contains:

```
mail.example.com
→ A
→ 203.0.113.25
```

Reverse DNS contains:

```
25.113.0.203.in-addr.arpa
→ PTR
→ mail.example.com
```

The forward and reverse lookup path becomes:

```
mail.example.com
        |
        v
A
        |
        v
203.0.113.25
        |
        v
PTR
        |
        v
mail.example.com
```

This creates consistent forward and reverse DNS.

However, email authentication and secure transport still depend on separate mechanisms such as:

```
SPF
DKIM
DMARC
TLS
```

DNS consistency alone does not prove that the server or message is trustworthy.

---

## Key Takeaways

- Reverse DNS maps IP addresses back to DNS names.
- PTR records are used for reverse DNS lookups.
- Forward DNS and reverse DNS are maintained independently.
- Creating an A or AAAA record does not automatically create a PTR record.
- IPv4 reverse DNS uses `in-addr.arpa`.
- IPv4 octets are reversed to fit IP addressing into the DNS hierarchy.
- A `/24` IPv4 subnet maps naturally into an `in-addr.arpa` reverse zone.
- IPv6 reverse DNS uses `ip6.arpa`.
- IPv6 reverse DNS operates on individual hexadecimal digits called nibbles.
- Each IPv6 hexadecimal digit represents 4 bits.
- Public reverse DNS authority normally follows control or delegation of the IP address block.
- ISPs and cloud providers often control PTR records for public addresses unless reverse DNS is delegated.
- Forward-confirmed reverse DNS checks whether PTR and forward A/AAAA mappings return to the same address.
- Matching forward and reverse DNS can improve consistency and reputation, especially for mail servers.
- Reverse DNS alone does not authenticate a system or email sender.
- PTR records can improve logs, monitoring, troubleshooting, and security analysis.
- Reverse zone delegation allows another organization to manage PTR records independently.
- IPv4 networks smaller than `/24` do not naturally align with reverse DNS octet boundaries.
- RFC 2317-style classless reverse delegation can be used for smaller IPv4 address blocks.
- Internal private address spaces can also use reverse DNS.
- DHCP and dynamic DNS can be used to maintain forward and reverse records automatically.
- `nslookup <IP>` can test reverse DNS.
- `dig -x <IP>` performs a reverse PTR lookup and automatically constructs the reverse-DNS query name.