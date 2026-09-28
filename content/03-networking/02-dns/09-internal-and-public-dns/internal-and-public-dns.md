# Internal and Public DNS

## Overview

Organizations often use DNS in two different contexts:

```
Public DNS
→ DNS information intended to be reachable from the Internet

Internal DNS
→ DNS information intended only for internal users and systems
```

Both use the same DNS protocol, but they serve different purposes and can expose different information.

A public DNS zone might contain records such as:

```
www.example.com
mail.example.com
vpn.example.com
```

while internal DNS might contain:

```
dc01.example.com
db01.example.com
fileserver01.example.com
```

or use a separate internal namespace such as:

```
dc01.corp.example.com
db01.corp.example.com
```

This topic focuses on:

- public DNS,
- internal DNS,
- authoritative and recursive roles,
- internal and public namespaces,
- split DNS,
- same-domain designs,
- separate internal subdomains,
- information exposure,
- operational risks.

The general DNS hierarchy is covered in [DNS Hierarchy](../03-dns-hierarchy/dns-hierarchy.md).

DNS zones are covered in [DNS Zones](../05-dns-zones/dns-zones.md).

Forwarding and recursive behavior are covered in [DNS Forwarding and Recursion](../07-forwarding-and-recursion/forwarding-and-recursion.md).

---

## Public DNS

Public DNS contains information intended to be available through the public DNS hierarchy.

For example:

```
www.example.com
→ A
→ 203.0.113.20
```

or:

```
mail.example.com
→ A
→ 203.0.113.25
```

These records may need to be resolved by:

- customers,
- external users,
- remote systems,
- mail servers,
- public services,
- Internet applications.

Public DNS is not necessarily operated by an ISP or external DNS provider.

An organization can:

- host its own public authoritative DNS,
- use a managed DNS provider,
- use a cloud DNS service,
- delegate DNS hosting to another organization.

The important point is:

> Public DNS is authoritative DNS information intentionally exposed through the public DNS hierarchy.

---

## Internal DNS

Internal DNS contains information intended for use inside an organization's private environment.

For example:

```
dc01.example.com
→ 10.10.10.10

db01.example.com
→ 10.10.20.20

fileserver01.example.com
→ 10.10.30.15
```

These names may only be resolvable from:

- the internal LAN,
- corporate Wi-Fi,
- VPN connections,
- private cloud networks,
- other trusted networks.

Internal DNS allows the organization to control its own naming structure independently.

For example:

```
Internal DNS
|
├── dc01.example.com
├── db01.example.com
├── fileserver01.example.com
└── backup01.example.com
```

This gives administrators control over:

- internal hostnames,
- private IP mappings,
- service records,
- internal aliases,
- reverse DNS,
- internal service discovery.

---

## Public DNS Is Not the Same as Public Recursive DNS

The phrase "public DNS" can sometimes cause confusion because DNS servers can perform different roles.

A **public authoritative DNS server** publishes authoritative data for public zones.

For example:

```
Authoritative for:
example.com
```

It can answer:

```
www.example.com
mail.example.com
```

because those records belong to its zone.

A **public recursive resolver** performs DNS resolution on behalf of clients.

Conceptually:

```
Client
  |
  v
Public recursive resolver
  |
  v
DNS hierarchy
```

These are different roles.

A useful distinction is:

```
Public authoritative DNS
→ publishes public zone data

Public recursive DNS
→ resolves names for clients
```

---

## Internal DNS Can Perform Multiple Roles

An internal DNS server can perform more than one role.

For example:

```
Internal DNS server
|
├── authoritative for example.com
|
└── recursive resolver for other domains
```

If an internal client asks for:

```
db01.example.com
```

the server may answer from its authoritative internal zone.

If the client asks for:

```
www.microsoft.com
```

the server can resolve the query according to its forwarding or recursion configuration.

The exact forwarding behavior is covered in [DNS Forwarding and Recursion](../07-forwarding-and-recursion/forwarding-and-recursion.md).

---

## Public and Internal Namespaces

Organizations must decide how internal naming relates to their public domain.

Two common approaches are:

```
Same namespace internally and publicly
```

or:

```
Separate internal subdomain
```

For example:

```
Public:
example.com
```

The internal environment could use:

```
example.com
```

as well, or:

```
corp.example.com
```

---

## Using the Same Namespace Internally and Publicly

An organization can use:

```
example.com
```

both publicly and internally.

For example:

```
Public example.com
|
├── www.example.com
├── mail.example.com
└── vpn.example.com
```

while the internal zone might contain:

```
Internal example.com
|
├── www.example.com
├── dc01.example.com
├── db01.example.com
└── fileserver01.example.com
```

This provides consistent naming.

Benefits can include:

- familiar names,
- simpler naming for users,
- consistency between internal and cloud services,
- compatibility with some hybrid designs,
- easier use of shared FQDNs.

However, this design introduces an important operational consequence.

---

## Internal Zone Shadowing

If an internal DNS server is authoritative for:

```
example.com
```

then internal clients querying that namespace normally receive answers from the internal zone.

For example:

```
Internal client
      |
      v
Internal DNS
authoritative for example.com
      |
      v
answer from internal zone
```

The DNS server does not normally ignore its own authoritative zone and query the public version of the same zone for missing records.

This means the internal zone effectively **shadows** the public zone.

---

## Example of Internal Zone Shadowing

Suppose the public DNS zone contains:

```
example.com
→ A
→ 203.0.113.20

www.example.com
→ A
→ 203.0.113.20
```

but the internal `example.com` zone contains only:

```
dc01.example.com
→ 10.10.10.10

db01.example.com
→ 10.10.20.20
```

An internal user queries:

```
www.example.com
```

The internal DNS server is authoritative for:

```
example.com
```

but it has no record for:

```
www.example.com
```

The query may therefore fail internally even though the public DNS record exists.

Conceptually:

```
Public DNS:
www.example.com
→ 203.0.113.20

Internal DNS:
www.example.com
→ missing
```

Result:

```
External users
→ resolution works

Internal users
→ resolution fails
```

---

## Public Records May Need to Be Recreated Internally

When the same namespace is used internally and publicly, records required by internal users often need to exist in the internal zone as well.

For example:

```
Public example.com

example.com
→ 203.0.113.20

www.example.com
→ 203.0.113.20

mail.example.com
→ 203.0.113.25
```

The internal zone may need equivalent records:

```
Internal example.com

example.com
→ 203.0.113.20

www.example.com
→ 203.0.113.20

mail.example.com
→ 203.0.113.25

dc01.example.com
→ 10.10.10.10

db01.example.com
→ 10.10.20.20
```

Otherwise:

```
works externally
fails internally
```

can become a common operational problem.

---

## Example: Apex Works Differently from WWW

A common issue can appear when:

```
example.com
```

is used as both the public and internal zone.

Public DNS might contain:

```
example.com
→ 203.0.113.20

www.example.com
→ 203.0.113.20
```

If the internal zone contains:

```
www.example.com
→ 203.0.113.20
```

but does not contain the corresponding record at the zone apex:

```
example.com
```

then internal users may successfully access:

```
www.example.com
```

while:

```
example.com
```

does not resolve to the public website as expected.

This is not a routing or web-server issue.

It is caused by the internal authoritative zone shadowing the public zone.

---

## Using a Separate Internal Subdomain

Another design is to separate public and internal namespaces.

For example:

```
Public:
example.com
```

and:

```
Internal:
corp.example.com
```

The resulting namespace might look like:

```
example.com
|
├── www.example.com
├── mail.example.com
└── corp.example.com
    |
    ├── dc01.corp.example.com
    ├── db01.corp.example.com
    └── fileserver01.corp.example.com
```

In this design:

```
www.example.com
```

continues to resolve through public DNS normally, while:

```
dc01.corp.example.com
```

is handled internally.

---

## Same Namespace vs Separate Internal Subdomain

### Same Namespace

Example:

```
Public:
example.com

Internal:
example.com
```

Advantages:

- consistent naming,
- shorter internal FQDNs,
- familiar user experience,
- can work well in some hybrid environments.

Disadvantages:

- internal zone shadows the public zone,
- public records may need to be reproduced internally,
- changes must be coordinated between public and internal DNS,
- missing records can create internal-only failures.

### Separate Internal Subdomain

Example:

```
Public:
example.com

Internal:
corp.example.com
```

Advantages:

- cleaner separation,
- public `example.com` continues resolving normally,
- less duplicated public DNS data,
- easier distinction between internal and public resources.

Disadvantages:

- longer internal names,
- separate namespace planning,
- users and administrators may need to understand both naming structures.

Neither design is automatically correct for every organization.

The choice depends on:

- operational requirements,
- application design,
- hybrid infrastructure,
- administrative complexity,
- security requirements,
- naming standards.

---

## Split DNS

**Split DNS**, also called **split-horizon DNS**, allows the same DNS name to return different information depending on which DNS environment receives the query.

For example:

```
portal.example.com
```

could resolve publicly as:

```
portal.example.com
→ 203.0.113.50
```

while internal DNS returns:

```
portal.example.com
→ 10.10.20.50
```

Conceptually:

```
                  portal.example.com
                         |
              +----------+----------+
              |                     |
        External query         Internal query
              |                     |
              v                     v
       203.0.113.50            10.10.20.50
```

Split DNS was introduced in [DNS Zones](../05-dns-zones/dns-zones.md).

---

## Different Services Behind the Same Name

Split DNS can also make the same FQDN lead to different services.

For example:

```
portal.example.com
```

could provide:

```
External users
→ public customer portal
```

while internal users receive:

```
Internal users
→ internal management console
```

Conceptually:

```
                  portal.example.com
                         |
              +----------+----------+
              |                     |
              v                     v
        Public service        Internal service
        203.0.113.50          10.10.20.50
```

This can be useful, but it requires careful design.

DNS only changes the destination returned to the client.

It does not automatically provide:

- authentication,
- authorization,
- TLS,
- application security,
- user separation.

Those controls must still be implemented by the services themselves.

---

## Why Use Different Internal and Public Answers?

Using a private internal destination can provide several benefits.

For example:

```
Internal:
portal.example.com
→ 10.10.20.50

Public:
portal.example.com
→ 203.0.113.50
```

Internal traffic can remain inside the private network instead of reaching the service through its public address.

Potential benefits include:

- reduced unnecessary Internet or edge traffic,
- simpler internal routing,
- no dependency on NAT reflection or hairpin NAT,
- direct access to private resources,
- different internal services,
- separate security policies,
- reduced public exposure.

---

## Public DNS Should Not Expose Unnecessary Internal Information

Internal infrastructure records generally do not need to be published publicly.

For example:

```
dc01.example.com
→ 10.10.10.10

db01.example.com
→ 10.10.20.20

backup01.example.com
→ 10.10.30.15
```

Publishing these records publicly can expose information about the internal environment.

Even though RFC1918 private addresses are not directly routable across the public Internet, the records can reveal:

- internal address ranges,
- host naming conventions,
- server roles,
- important infrastructure,
- likely high-value systems,
- organizational structure.

This information can assist reconnaissance.

A useful principle is:

> Public DNS should expose only the information required for public services.

---

## Information Disclosure

DNS information can be useful during reconnaissance.

Consider:

```
dc01.example.com
db01.example.com
backup01.example.com
vpn-gateway.example.com
```

Even without direct access to those systems, the names may reveal:

```
domain controller
database server
backup infrastructure
VPN gateway
```

This can help an attacker understand the environment before attempting further attacks.

Internal DNS should therefore avoid unnecessary public exposure of private infrastructure information.

---

## Public and Internal DNS Records Can Differ

The same DNS name does not need to contain identical data in every DNS environment.

For example:

```
Public:
files.example.com
→ 203.0.113.60
```

while internally:

```
Internal:
files.example.com
→ 10.10.30.60
```

The name remains the same, but the returned destination changes.

This design must be intentional.

Administrators should know which DNS view is authoritative for which clients.

---

## New Public Records and Internal DNS

Using the same namespace internally and publicly creates an important change-management requirement.

Suppose a new public service is created:

```
service.example.com
→ 203.0.113.80
```

External users query public DNS:

```
External client
      |
      v
Public DNS
      |
      v
service.example.com
→ 203.0.113.80
```

But internal users query the internal `example.com` zone.

If the new record is missing internally:

```
Internal client
      |
      v
Internal DNS
authoritative for example.com
      |
      v
service.example.com
not present
      |
      v
resolution fails
```

This produces:

```
works externally
fails internally
```

even though the public DNS record is configured correctly.

---

## Change Management in Split DNS

Split DNS environments should have clear procedures for DNS changes.

When adding or modifying public records, administrators should determine whether the same name is also required internally.

A simple checklist can include:

```
Is the record public?

Should internal users resolve it?

Should the internal answer match the public answer?

Should internal users receive a private address instead?

Does the internal authoritative zone shadow the public zone?

Does the change need to be made in both DNS environments?
```

This helps prevent inconsistent DNS behavior.

---

## Public Authoritative DNS

Public authoritative DNS should provide records required by systems outside the organization.

For example:

```
example.com
|
├── www.example.com
├── mail.example.com
├── vpn.example.com
└── portal.example.com
```

The public authoritative infrastructure should generally focus on authoritative DNS service.

It should not normally provide unrestricted recursive resolution to arbitrary Internet users.

The exact separation depends on the DNS architecture and implementation.

---

## Internal Recursive DNS

Internal clients commonly use internal recursive resolvers.

For example:

```
Internal client
      |
      v
Internal DNS resolver
      |
      +-- internal zone
      |
      +-- cache
      |
      +-- forwarder / recursion
```

The internal DNS server can:

- answer internal authoritative records,
- answer cached queries,
- resolve external names,
- enforce internal DNS policy.

The detailed mechanics of forwarding and recursion are covered in [DNS Forwarding and Recursion](../07-forwarding-and-recursion/forwarding-and-recursion.md).

---

## Internal DNS and VPN Users

Remote users connected through a corporate VPN may also need access to internal DNS.

For example:

```
Remote user
    |
    v
VPN
    |
    v
Internal DNS
    |
    v
db01.example.com
→ 10.10.20.20
```

Without access to the internal DNS resolver, private DNS names may not resolve correctly.

This means VPN design often needs to provide:

- internal DNS server addresses,
- appropriate DNS routes,
- correct search domains,
- access to internal DNS infrastructure.

The exact configuration depends on the VPN and operating system.

---

## Public DNS and Private IP Addresses

Technically, a public DNS zone can contain a private IP address.

For example:

```
internal.example.com
→ 10.10.20.20
```

However, doing this is usually undesirable unless there is a specific design reason.

The address will not normally be reachable from the public Internet, but the record can still leak internal network information.

The better design is generally:

```
Public DNS
→ public information

Internal DNS
→ private infrastructure information
```

---

## Internal DNS Does Not Automatically Mean Private Domain Names

An internal DNS server can host names underneath a publicly registered domain.

For example:

```
dc01.corp.example.com
```

can be resolvable only internally even though:

```
example.com
```

is a public Internet domain.

What matters is which authoritative DNS infrastructure publishes the record and which clients can query it.

This allows organizations to use their registered domain as the basis for internal naming without publishing all internal names publicly.

---

## Public and Internal Reverse DNS

Forward DNS is not the only area where internal and public DNS can differ.

Reverse DNS can also be separate.

For example:

```
Public:
203.0.113.25
→ mail.example.com
```

while internally:

```
10.10.20.25
→ app01.example.com
```

Public reverse DNS may be controlled by an ISP or hosting provider, while internal reverse DNS is normally controlled by the organization.

Reverse DNS is covered in [Reverse DNS](../08-reverse-dns/reverse-dns.md).

---

## DNS Filtering and Internal Resolvers

Internal resolvers can also be used as enforcement points for DNS policy.

For example:

```
Client
  |
  v
Internal DNS
  |
  v
DNS policy / filtering
```

This can allow an organization to control which domains clients can resolve.

DNS filtering is only mentioned here as an architectural role of internal resolvers.

The security mechanisms, limitations, and filtering techniques belong in the dedicated DNS Security topic.

---

## Example Small Environment

A small organization might use:

```
Public DNS:
example.com

Internal DNS:
corp.example.com
```

Public records:

```
www.example.com
→ 203.0.113.20

mail.example.com
→ 203.0.113.25
```

Internal records:

```
dc01.corp.example.com
→ 10.10.10.10

db01.corp.example.com
→ 10.10.20.20

files.corp.example.com
→ 10.10.30.20
```

This design provides clear separation between public and internal namespaces.

---

## Example Same-Namespace Environment

Another organization may use:

```
example.com
```

both internally and publicly.

Public DNS:

```
example.com
→ 203.0.113.20

www.example.com
→ 203.0.113.20

mail.example.com
→ 203.0.113.25
```

Internal DNS:

```
example.com
→ 203.0.113.20

www.example.com
→ 203.0.113.20

mail.example.com
→ 203.0.113.25

dc01.example.com
→ 10.10.10.10

db01.example.com
→ 10.10.20.20
```

This provides consistent names but requires administrators to maintain the internal representation of public records needed by internal clients.

---

## Example Split-DNS Environment

Consider:

```
portal.example.com
```

Public DNS:

```
portal.example.com
→ 203.0.113.50
```

Internal DNS:

```
portal.example.com
→ 10.10.20.50
```

The same name can therefore direct users to different resources:

```
External user
      |
      v
203.0.113.50
Public portal
```

```
Internal user
      |
      v
10.10.20.50
Internal application
```

DNS determines which destination is returned.

Application security still needs to be implemented independently.

---

## Common Design Problems

### Public Record Missing Internally

```
Public:
service.example.com exists

Internal:
service.example.com missing
```

Result:

```
external resolution works
internal resolution fails
```

---

### Internal Infrastructure Published Publicly

```
dc01.example.com
→ 10.10.10.10
```

published in public DNS.

Result:

```
unnecessary infrastructure disclosure
```

---

### Inconsistent Split-DNS Records

Public and internal records point to different destinations unintentionally.

Result:

```
different users reach different systems
unexpected troubleshooting results
```

---

### Incorrect Resolver Configuration

Internal clients use an external resolver directly.

Result:

```
public names resolve
internal-only names fail
```

---

### Missing VPN DNS Configuration

Remote users connect to the network but do not use internal DNS.

Result:

```
network connection works
internal names do not resolve
```

---

## Troubleshooting Internal vs Public DNS

When a name resolves differently depending on location, useful questions include:

```
Which DNS server is the client using?

Is that DNS server authoritative for the queried domain?

Does an internal zone shadow the public zone?

Does the record exist in the internal zone?

Does the same record exist publicly?

Should the internal and public answers be identical?

Is split DNS intentionally configured?

Is the client connected through VPN?

Is the VPN providing internal DNS settings?

Is the client using a public resolver instead of the corporate resolver?

Is the result cached?
```

A useful troubleshooting comparison is:

```
Internal resolver result
vs
Public resolver result
vs
Authoritative public result
```

This can quickly reveal whether the problem is caused by DNS view differences rather than network connectivity.

---

## Key Takeaways

- Public DNS contains DNS information intended to be reachable through the public DNS hierarchy.
- Internal DNS contains DNS information intended for private networks and trusted users.
- Public DNS does not have to be hosted by an ISP or third-party provider.
- Public authoritative DNS and public recursive DNS perform different roles.
- Internal DNS servers can be both authoritative for internal zones and recursive resolvers for other domains.
- Organizations can use the same DNS namespace internally and publicly.
- An internal authoritative zone can shadow the public version of the same zone.
- When the same namespace is used internally and publicly, public records required internally may need to exist in both DNS environments.
- Missing internal copies of public records can create situations where a service works externally but fails internally.
- A separate internal subdomain such as `corp.example.com` can provide cleaner separation.
- Same-namespace and separate-subdomain designs both have advantages and disadvantages.
- Split DNS allows the same DNS name to return different answers depending on the client's DNS environment.
- Split DNS can direct internal and external users to different services using the same FQDN.
- DNS does not provide authentication, authorization, or TLS by itself.
- Public DNS should normally expose only information required for public services.
- Publishing internal hostnames and private IP addresses can leak useful infrastructure information.
- Split-DNS environments require careful change management.
- Internal recursive resolvers can also act as DNS policy enforcement points.
- Remote VPN users often need access to internal DNS in order to resolve private names.
- Public and internal reverse DNS can also be managed separately.
- Troubleshooting should compare internal and public DNS results before assuming the problem is routing or application connectivity.