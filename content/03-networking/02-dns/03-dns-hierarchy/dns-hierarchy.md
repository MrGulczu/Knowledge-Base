# DNS Hierarchy

## Overview

DNS is not a flat database.

It is a **hierarchical and distributed naming system** where responsibility for different parts of the namespace can be delegated to different DNS servers and different administrative teams.

The basic DNS roles and naming concepts are introduced in [DNS Fundamentals](../01-dns-fundamentals/dns-fundamentals.md).

The way a resolver follows the hierarchy during an actual lookup is covered in [DNS Resolution Process](../02-dns-resolution-process/dns-resolution-process.md).

This topic focuses specifically on the structure of the DNS namespace and how administrative responsibility is divided.

A simplified DNS hierarchy looks like this:

```
.
└── com.
    └── example.com.
        └── sales.example.com.
            └── www.sales.example.com.
```

DNS becomes more specific as we move from **right to left**.

---

## The DNS Root

The highest level of the DNS hierarchy is the **DNS root**.

It is represented by:

```
.
```

The root sits above every Top-Level Domain.

Conceptually:

```
.
├── com.
├── net.
├── org.
├── pl.
├── de.
└── ...
```

The root zone contains delegation information that allows resolvers to discover which DNS servers are responsible for Top-Level Domains.

For example, when a resolver needs information about:

```
www.example.com
```

the root does not normally contain the final DNS record.

Instead, the root can direct the resolver toward the DNS servers responsible for:

```
.com
```

This referral behavior is covered in more detail in [DNS Resolution Process](../02-dns-resolution-process/dns-resolution-process.md).

---

## Top-Level Domains

A **Top-Level Domain**, commonly abbreviated as **TLD**, is the level directly below the DNS root.

Examples include:

```
.com
.net
.org
.pl
.de
.fr
```

The hierarchy can be represented as:

```
.
├── com.
├── net.
├── org.
├── pl.
├── de.
└── fr.
```

TLDs can be divided into different categories.

Common examples such as:

```
.com
.net
.org
```

are generic Top-Level Domains.

Country-code Top-Level Domains include:

```
.pl
.de
.fr
.uk
```

The important point is:

> A TLD represents one level of the DNS hierarchy and delegates responsibility for domains beneath it.

---

## Domains Below a TLD

Consider:

```
example.com
```

The hierarchy is:

```
.
└── com.
    └── example.com.
```

Here:

```
.com
```

is the Top-Level Domain.

The name:

```
example.com
```

is a domain beneath `.com`.

In traditional terminology, this is often described as a **second-level domain**.

It is important not to confuse:

```
.com
```

with:

```
example.com
```

Only `.com` is the TLD.

---

## Subdomains

A domain can contain additional domains beneath it.

For example:

```
sales.example.com
```

is a subdomain of:

```
example.com
```

The hierarchy becomes:

```
.
└── com.
    └── example.com.
        └── sales.example.com.
```

Subdomains can continue to additional levels:

```
eu.sales.example.com
```

which produces:

```
.
└── com.
    └── example.com.
        └── sales.example.com.
            └── eu.sales.example.com.
```

There is no requirement that every subdomain must become a separate DNS zone.

A subdomain may simply exist inside the same zone as its parent.

---

## DNS Names and Labels

Consider:

```
www.sales.example.com.
```

The individual parts separated by dots are DNS **labels**:

```
www
sales
example
com
```

The final dot represents the DNS root.

The hierarchy is:

```
.                         Root
└── com.                  TLD
    └── example.com.      Domain under .com
        └── sales.example.com.
            └── www.sales.example.com.
```

The name becomes increasingly specific as we move from **right to left**.

The basic concepts of labels, FQDNs, and the trailing root dot are introduced in [DNS Fundamentals](../01-dns-fundamentals/dns-fundamentals.md).

---

## A DNS Name Does Not Define Its Purpose

A name such as:

```
www.sales.example.com
```

does not automatically mean that DNS treats it as a web service.

It is simply a DNS name.

Different DNS record types can be associated with it.

The label:

```
www
```

is therefore a naming convention rather than a built-in DNS service definition.

---

## DNS Domain vs DNS Zone

The difference between a **domain** and a **zone** is one of the most important concepts in DNS hierarchy.

A useful distinction is:

```
Domain
→ logical and hierarchical structure
  of the DNS namespace

Zone
→ administrative container for
  authoritative DNS data
```

A domain describes where a name exists in the hierarchy.

A zone describes which DNS authority manages a particular portion of that hierarchy.

---

## One Domain Can Contain Many Subdomains in One Zone

Suppose we have:

```
example.com
```

and the following names:

```
sales.example.com
www.sales.example.com
mail.sales.example.com
vpn.example.com
```

All of these can exist inside a single DNS zone:

```
Zone: example.com

example.com
├── sales.example.com
├── www.sales.example.com
├── mail.sales.example.com
└── vpn.example.com
```

In this design:

```
sales.example.com
```

is a subdomain, but it is **not a separate DNS zone**.

The same authoritative DNS zone manages the entire namespace shown above.

This design is simpler and may be appropriate for smaller environments where separate administrative boundaries are unnecessary.

---

## A Subdomain Can Become a Separate Zone

A subdomain can also be delegated into its own DNS zone.

For example:

```
example.com
```

can delegate:

```
sales.example.com
```

to a different authoritative DNS zone.

Conceptually:

```
Zone: example.com

example.com
├── www.example.com
├── mail.example.com
└── sales.example.com
        |
        | delegation
        v

Zone: sales.example.com

sales.example.com
├── www.sales.example.com
├── mail.sales.example.com
└── app.sales.example.com
```

Now:

```
sales.example.com
```

is both:

- a subdomain of `example.com`,
- the apex of a separate DNS zone.

A useful rule is:

```
Subdomain
≠
Automatically a separate zone
```

but:

```
Delegated subdomain
→ separate child zone
```

---

## Why Delegate a Child Zone

Delegation creates a separate administrative boundary.

This is useful when different teams or organizations need control over different parts of the DNS namespace.

For example:

```
Central IT
→ manages example.com

Sales Infrastructure Team
→ manages sales.example.com
```

The parent administrator no longer needs to maintain every record below:

```
sales.example.com
```

Instead, the child zone can be managed independently.

This approach can be useful in:

- large enterprises,
- multinational organizations,
- separate business units,
- departments with independent infrastructure teams,
- managed service environments,
- hybrid or multi-cloud environments.

In smaller environments, keeping everything in one zone can be simpler and easier to manage.

---

## Parent and Child Zones

When one zone delegates part of its namespace to another zone, they form a **parent-child relationship**.

For example:

```
Parent Zone:
example.com

Child Zone:
sales.example.com
```

Conceptually:

```
example.com
    |
    | delegation
    v
sales.example.com
```

The parent remains responsible for telling resolvers which DNS servers are authoritative for the child.

The child becomes responsible for the DNS data inside the delegated namespace.

---

## The Parent Keeps the Delegation

After delegation, the parent does not need to contain the normal records that belong inside the child zone.

For example, the parent does not need to manage:

```
www.sales.example.com
mail.sales.example.com
app.sales.example.com
```

Instead, it keeps the information needed to refer resolvers to the child zone.

Conceptually:

```
Parent Zone: example.com

sales.example.com
    |
    ├── authoritative server 1
    └── authoritative server 2
```

The child zone then contains the authoritative data for its own namespace.

---

## NS Records and Delegation

Delegation is performed using **NS records**.

Suppose:

```
sales.example.com
```

is delegated to:

```
ns1.sales.example.com
ns2.sales.example.com
```

The parent zone contains delegation information conceptually similar to:

```
sales.example.com
→ NS ns1.sales.example.com

sales.example.com
→ NS ns2.sales.example.com
```

These records tell resolvers which DNS servers are authoritative for the child zone.

The resolver can then send DNS queries directly to those authoritative servers.

NS records are introduced here only in the context of delegation. DNS record types are covered in [DNS Records](../04-dns-records/dns-records.md).

---

## Glue Records

Sometimes the name of the authoritative server exists inside the very child zone being delegated.

For example:

```
sales.example.com
→ NS ns1.sales.example.com
```

This creates a dependency.

To contact:

```
ns1.sales.example.com
```

the resolver first needs its IP address.

But to resolve that name, it may need to contact the DNS servers for:

```
sales.example.com
```

Conceptually:

```
Need IP of:
ns1.sales.example.com
        |
        v
Need to query:
sales.example.com
        |
        v
Need IP of:
ns1.sales.example.com
```

To break this circular dependency, the parent can provide address information for the child name server.

This is commonly called **glue**.

For example:

```
Delegation:

sales.example.com
→ NS ns1.sales.example.com

Glue:

ns1.sales.example.com
→ 192.0.2.53
```

The resolver now has enough information to contact the child name server directly.

The role of glue during an actual DNS lookup is also shown in [DNS Resolution Process](../02-dns-resolution-process/dns-resolution-process.md).

---

## Glue Is Not Required for Every Delegation

Suppose the child zone instead uses:

```
ns1.dns-provider.net
ns2.dns-provider.net
```

These names are outside:

```
sales.example.com
```

The resolver can resolve them normally through the DNS hierarchy.

In this situation, the parent does not necessarily need to provide glue.

The most important case for glue is when the authoritative name server is located inside the delegated child namespace.

For example:

```
sales.example.com
→ NS ns1.sales.example.com
```

---

## Zone Apex

The **zone apex** is the top name of a particular DNS zone.

For example:

```
Zone:
example.com
```

has the apex:

```
example.com
```

If:

```
sales.example.com
```

is delegated into its own zone, then:

```
Zone:
sales.example.com
```

has the apex:

```
sales.example.com
```

So multiple zones at different levels each have their own apex.

For example:

```
Zone: example.com
Apex: example.com

Zone: sales.example.com
Apex: sales.example.com

Zone: eu.sales.example.com
Apex: eu.sales.example.com
```

The zone apex therefore means:

> The name at the top of that specific DNS zone and the point where its administrative authority begins.

---

## Apex Records

Certain DNS records are commonly associated with the zone apex.

For example:

```
Zone:
example.com

Apex:
example.com
```

can contain information such as:

```
SOA
NS
A
AAAA
MX
TXT
```

Names below the apex can include:

```
www.example.com
mail.example.com
vpn.example.com
```

The individual record types are not covered in detail here because this topic focuses on hierarchy rather than DNS record behavior.

---

## Authoritative Boundaries

Delegation creates an **authoritative boundary**.

Suppose:

```
example.com
```

delegates:

```
sales.example.com
```

The parent zone remains authoritative for its own namespace up to the delegation point.

For example:

```
Zone: example.com

Authoritative for:

example.com
www.example.com
mail.example.com
vpn.example.com
```

At:

```
sales.example.com
```

the parent delegates responsibility.

Conceptually:

```
example.com
    |
    | authoritative responsibility
    |
    ├── www.example.com
    ├── mail.example.com
    |
    └── sales.example.com
            |
            | delegation
            v
        sales.example.com zone
            |
            ├── www.sales.example.com
            ├── mail.sales.example.com
            └── app.sales.example.com
```

The child zone becomes authoritative for the delegated namespace.

A useful rule is:

> The parent keeps the delegation, the child keeps the authoritative DNS data for the delegated namespace.

---

## Delegation Does Not Require Different Physical Servers

A delegated child zone can be hosted on completely different DNS infrastructure, but this is not required.

The important change is the **administrative and authoritative boundary**, not necessarily the hardware.

For example:

```
example.com
```

and:

```
sales.example.com
```

could technically be hosted by the same DNS platform while still existing as separate zones.

What matters is that DNS treats them as separate authoritative zones with a delegation between them.

---

## Hierarchy and Administrative Responsibility

The DNS hierarchy allows responsibility to be divided at many levels.

For example:

```
.
└── com.
    └── example.com.
        ├── sales.example.com.
        ├── engineering.example.com.
        └── eu.example.com.
```

An organization could manage this as one zone:

```
Zone:
example.com
```

or delegate individual sections:

```
Zone:
example.com

Child Zone:
sales.example.com

Child Zone:
engineering.example.com

Child Zone:
eu.example.com
```

This gives organizations flexibility in how they structure DNS management.

---

## Why DNS Hierarchy Scales

A single global DNS database managed by one authority would be extremely difficult to operate.

Instead, DNS distributes responsibility.

Conceptually:

```
Root
  |
  ├── .com
  |     |
  |     ├── example.com
  |     ├── company.com
  |     └── other.com
  |
  ├── .pl
  |     |
  |     ├── example.pl
  |     └── company.pl
  |
  └── ...
```

Responsibility can then be delegated even further:

```
example.com
    |
    ├── sales.example.com
    ├── engineering.example.com
    └── eu.example.com
```

This provides several benefits:

- no single DNS server must store every DNS record,
- different organizations can manage their own namespaces,
- large organizations can divide administrative responsibility,
- DNS can scale globally,
- changes can be managed close to the systems they affect,
- authoritative responsibility is clearly defined.

---

## Complete Hierarchy Example

Consider:

```
www.sales.example.com.
```

The hierarchy can be read from right to left:

```
.
→ DNS root

com.
→ Top-Level Domain

example.com.
→ domain beneath .com

sales.example.com.
→ subdomain of example.com

www.sales.example.com.
→ DNS name beneath sales.example.com
```

If everything is managed in one zone:

```
Zone: example.com

example.com
└── sales.example.com
    └── www.sales.example.com
```

there is no separate delegation.

If `sales.example.com` is delegated:

```
Zone: example.com
        |
        | NS delegation
        v
Zone: sales.example.com
        |
        └── www.sales.example.com
```

then the DNS hierarchy is the same, but the **administrative structure is different**.

That distinction is central to understanding DNS domains and zones.

---

## Key Takeaways

- DNS is a hierarchical and distributed naming system.
- The DNS root is represented by `.` and sits at the top of the namespace.
- Top-Level Domains such as `.com`, `.net`, `.org`, and `.pl` exist directly below the root.
- `example.com` is a domain beneath `.com`; it is not itself a Top-Level Domain.
- A subdomain is a domain that exists beneath another domain.
- A subdomain does not automatically become a separate DNS zone.
- A DNS domain describes logical position in the namespace.
- A DNS zone is an administrative container for authoritative DNS data.
- A delegated subdomain becomes a separate child zone.
- Delegation allows different teams or organizations to manage different parts of the namespace.
- The parent zone keeps delegation information for the child.
- The child zone keeps the authoritative DNS data for its delegated namespace.
- NS records identify the authoritative DNS servers for a delegated zone.
- Glue provides address information needed to reach certain delegated name servers.
- Glue is especially important when the authoritative name server is located inside the delegated child namespace.
- The zone apex is the top name of a specific DNS zone.
- Every separate zone has its own zone apex.
- Delegation creates an authoritative boundary between parent and child zones.
- Separate zones do not necessarily require separate physical DNS servers.
- DNS hierarchy makes the global namespace scalable and allows administrative responsibility to be distributed.