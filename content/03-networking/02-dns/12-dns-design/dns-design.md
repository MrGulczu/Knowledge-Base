# DNS Design

## Overview

DNS design is not only about choosing which records to create.

A good DNS architecture must also answer questions such as:

```
Where should authoritative DNS be hosted?

Where should recursive DNS be placed?

How should branches behave during WAN outages?

How should internal and public DNS be separated?

How should DNS changes be synchronized?

How should VPN users resolve names?

How should DNS logging be collected?

How much redundancy is actually required?
```

There is no single DNS architecture that is correct for every organization.

A small office with 20 employees has very different requirements from a company with:

```
multiple branches
private cloud infrastructure
remote workers
business-critical applications
centralized security monitoring
```

DNS design should therefore be based on:

```
business requirements
availability
security
administrative model
failure domains
latency
traffic patterns
number of locations
workload criticality
```

This topic uses a practical company scenario to demonstrate how these decisions can be made.

For detailed protocol behavior, see:

- [DNS Fundamentals](../01-dns-fundamentals/dns-fundamentals.md)
- [DNS Resolution Process](../02-dns-resolution-process/dns-resolution-process.md)
- [DNS Hierarchy](../03-dns-hierarchy/dns-hierarchy.md)
- [DNS Records](../04-dns-records/dns-records.md)
- [DNS Zones](../05-dns-zones/dns-zones.md)
- [DNS Caching and TTL](../06-caching-and-ttl/caching-and-ttl.md)
- [DNS Forwarding and Recursion](../07-forwarding-and-recursion/forwarding-and-recursion.md)
- [Reverse DNS](../08-reverse-dns/reverse-dns.md)
- [Internal and Public DNS](../09-internal-and-public-dns/internal-and-public-dns.md)
- [DNS Security](../10-dns-security/dns-security.md)
- [DNS Troubleshooting](../11-dns-troubleshooting/dns-troubleshooting.md)

---

## Example Organization

Assume we are designing DNS for a medium/big-sized organization with approximately:

```
700 employees
```

The company has:

```
Headquarters
~350 employees
central ICT department

Branch A
~120 employees

Branch B
~80 employees

Private cloud / datacenter

~150 regular remote workers

Microsoft 365 / SaaS services

Public-facing services

Internal business applications
```

The company owns:

```
example.com
```

and wants to use the same general namespace internally and publicly where appropriate.

---

## Network and Service Requirements

The organization uses services such as:

```
www.example.com
portal.example.com
vpn.example.com

files.example.com
erp.example.com
monitoring.example.com
internal APIs
domain services
```

Some services are public.

Others must only be available internally.

The company also wants:

```
central DNS administration
central security monitoring
central DNS logging
DNS filtering
branch resiliency
VPN integration
consistent naming
```

This immediately tells us that a single DNS server is not enough.

---

## Public and Internal DNS Should Be Separate

The first major design decision is to separate:

```
public authoritative DNS
```

from:

```
internal DNS
```

Both may use:

```
example.com
```

but contain different information.

---

## Public DNS

The public version of:

```
example.com
```

contains only records that must be visible from the Internet.

For example:

```
www.example.com
portal.example.com
vpn.example.com
mail.example.com
MX
SPF
DKIM
DMARC
```

Conceptually:

```
Internet
   |
   v
Public authoritative DNS
   |
   +-- www.example.com
   +-- portal.example.com
   +-- vpn.example.com
   +-- mail.example.com
```

---

## Internal DNS

The internal version of:

```
example.com
```

can contain additional records such as:

```
files.example.com
erp.example.com
monitoring.example.com
dc01.example.com
internal APIs
private application endpoints
```

It may also contain internal versions of records that exist publicly.

For example:

```
portal.example.com
```

could resolve publicly to:

```
198.51.100.50
```

while internally it resolves to:

```
10.20.10.50
```

This is a split-DNS design.

The same FQDN can be used by users regardless of location while DNS returns the appropriate destination.

For more detail, see [Internal and Public DNS](../09-internal-and-public-dns/internal-and-public-dns.md).

---

## Split DNS Trade-Off

Using the same zone internally and publicly provides:

```
consistent naming
control over internal answers
private records remain private
different internal/public addresses
simpler hybrid-cloud naming
VPN integration
```

However, it introduces an important operational requirement.

If the internal DNS server is authoritative for:

```
example.com
```

it normally does not ask public DNS for missing records in the same zone.

For example:

```
Public example.com
├── www
├── portal
└── vpn

Internal example.com
├── erp
└── files
```

An internal request for:

```
www.example.com
```

may fail if `www` does not also exist in the internal zone.

Therefore, records needed internally must also exist in the internal version of the zone.

A common operational problem is:

```
works externally
fails internally
```

because the public and internal zone contents are not synchronized correctly.

---

## Public Authoritative DNS

In this example, the public zone contains only a small number of records.

There is little benefit in operating dedicated public DNS servers in the company datacenter.

A managed DNS provider is therefore a good fit.

Conceptually:

```
Internet
   |
   v
Managed Public DNS Provider
authoritative for example.com
```

Advantages include:

```
geographic redundancy
high availability
simpler maintenance
reduced infrastructure exposure
DDoS resilience
easier DNSSEC support
```

The public DNS environment should remain intentionally simple.

---

## Registrar vs DNS Provider

The domain registrar and DNS provider do not have to be the same organization.

The registrar controls the registration of:

```
example.com
```

while the DNS provider operates the authoritative DNS servers.

The registrar delegates the domain to the name servers provided by the DNS provider.

---

## Internal DNS Architecture

The internal architecture can follow a hub-and-spoke model.

The headquarters acts as the central DNS management location.

Branches and other important sites receive synchronized authoritative copies and provide local recursive resolution.

Conceptually:

```
                         HQ
                  Central DNS Hub
                         |
       +-----------------+-----------------+
       |                 |                 |
       v                 v                 v
   Branch A          Branch B        Private Cloud
     DNS               DNS               DNS
```

---

## Central DNS Management

The headquarters is the main DNS management point.

DNS administrators:

```
create records
modify records
delete records
manage zones
manage reverse DNS
configure forwarding
control DNS policy
```

from the central environment.

Branch DNS servers should not become independent administrative authorities.

The goal is:

```
central management
+
distributed availability
```

---

## Authoritative and Recursive Roles

Authoritative DNS and recursive DNS solve different problems.

```
Authoritative DNS
→ stores and serves data for zones it owns
```

```
Recursive DNS
→ resolves names on behalf of clients
```

In large environments these roles can be separated.

In smaller branch locations, combining them on the same local DNS server may be reasonable.

---

## Branch DNS Design

Each important branch can contain a local DNS server that provides:

```
secondary authoritative copy of internal zones
+
recursive resolution for users
```

For example:

```
Branch A DNS
├── secondary authoritative example.com
├── reverse zones where required
└── recursive resolver
```

The branch does not independently manage the zone.

Instead, it receives synchronized data from HQ.

---

## Why Keep a Local Authoritative Copy

If a branch relies entirely on HQ DNS:

```
Branch
  |
  v
WAN
  |
  v
HQ DNS
```

then a WAN failure can also become a DNS failure.

A local secondary authoritative copy allows the branch to continue resolving internal names even if the connection to HQ is unavailable.

For example:

```
erp.example.com
files.example.com
monitoring.example.com
```

can continue resolving locally.

---

## Why Cache Alone Is Not Enough

A recursive cache can temporarily preserve previously resolved records.

However:

```
cache hit
→ works
```

while:

```
cache expired
→ authoritative server required
```

Therefore, relying only on cache does not provide reliable internal DNS availability during long outages.

If a branch must continue operating independently, it should have access to authoritative data.

---

## Forward-First for Public Resolution

Branches normally forward public DNS queries to HQ.

For example:

```
Branch client
     |
     v
Branch DNS
     |
     v
HQ DNS
     |
     v
Internet resolution
```

This allows the organization to maintain:

```
central DNS filtering
central query visibility
consistent security policy
central monitoring
```

The branch resolver can be configured conceptually as:

```
forward-first
```

This means:

```
try central forwarder first
```

and if the central forwarder is unavailable:

```
perform recursion directly
```

---

## Branch Behavior During HQ Outage

During normal operation:

```
Internal names
→ resolved locally

Public names
→ forwarded to HQ
```

During an HQ outage:

```
Internal names
→ still resolved locally from secondary zone

Public names
→ resolved directly by branch resolver
```

This creates a useful balance between:

```
central control
```

and:

```
site resilience
```

---

## Availability vs Central Control

Forward-first introduces an important design trade-off.

Normally:

```
public DNS query
→ HQ
→ central logging
```

During an HQ outage:

```
public DNS query
→ direct branch recursion
```

Central logging may therefore become incomplete during the outage.

This leads to an important principle:

> Availability and centralized control can conflict.

A resilient DNS design should define which controls may temporarily be bypassed during infrastructure failure.

---

## Collect Logs from Every Resolver

Instead of relying only on HQ forwarding logs, every resolver should generate its own telemetry.

For example:

```
HQ DNS -----------Branch A DNS ------Branch B DNS -------+----> SIEM
Cloud DNS ----------/
VPN DNS -----------/
```

This means branch activity can still be recorded locally even when HQ is unreachable.

When connectivity returns, logs can again be sent to the central SIEM.

---

## Branch DNS Redundancy

Not every branch requires two local DNS servers.

The decision should depend on:

```
number of users
criticality of local services
acceptable downtime
WAN reliability
business continuity requirements
```

---

## One Local DNS Server with HQ Failover

A smaller branch may use:

```
Client
  |
  +--> Branch DNS
  |
  +--> HQ DNS
```

This provides local resolution during normal operation while HQ serves as an alternative resolver.

However:

```
Branch DNS failure
+
WAN/HQ failure
=
no DNS
```

This may be acceptable for a small or non-critical site.

---

## Two Local DNS Servers

A critical branch may instead use:

```
Branch A

DNS 1
├── secondary authoritative copy
└── recursive resolver

DNS 2
├── secondary authoritative copy
└── recursive resolver
```

This allows the branch to survive:

```
single local DNS failure
```

and:

```
HQ/WAN outage
```

at the same time.

The amount of redundancy should match the business requirement.

---

## Failure Domains Matter

DNS redundancy is only useful if redundant servers do not depend on exactly the same infrastructure.

For example:

```
DNS 1
DNS 2
```

may still share:

```
same hypervisor
same storage
same switch
same power source
same WAN link
```

If they do, they may fail together.

Therefore:

> DNS redundancy should be evaluated by failure domain, not only by server count.

---

## Private Cloud DNS

A private cloud does not automatically need its own DNS server.

The decision should depend on:

```
service criticality
DNS dependency
network-path reliability
latency
query volume
failure impact
```

---

## When the Cloud Can Depend on HQ DNS

If the cloud contains only a few systems and has a reliable S2S connection to HQ, it may be reasonable to use:

```
HQ/internal DNS
```

directly.

For example:

```
few backup servers
monitoring collectors
small internal applications
```

may not justify additional DNS infrastructure.

---

## When a Local Cloud DNS Server Makes Sense

If the cloud hosts:

```
application clusters
internal APIs
database services
container platforms
authentication services
automation
monitoring
```

then DNS becomes a much more important dependency.

In that situation a local DNS server may provide:

```
secondary authoritative copy
+
recursive resolver
```

so cloud workloads do not completely depend on an S2S tunnel.

---

## DNS Placement and Traffic

Very high DNS query volume can be another reason to place a resolver close to workloads.

However, DNS traffic itself is usually lightweight and heavily reduced by caching.

Therefore:

> Place DNS close to workloads primarily for availability, dependency reduction, and latency rather than bandwidth savings alone.

---

## VPN DNS Design

Remote VPN users introduce another DNS path.

Before the VPN is established, the endpoint must be able to resolve the public VPN hostname.

For example:

```
vpn.example.com
```

using:

```
home DNS
ISP DNS
public resolver
```

Conceptually:

```
Remote workstation
      |
      v
Public DNS
      |
      v
vpn.example.com
      |
      v
VPN gateway
```

---

## DNS After VPN Connection

Once connected, corporate devices should use corporate DNS servers.

For example:

```
Remote workstation
      |
      | VPN
      v
Corporate DNS
      |
      +--> internal example.com
      |
      +--> public Internet domains
```

This provides:

```
central logging
consistent filtering
malicious-domain blocking
internal DNS resolution
consistent DNS policy
```

---

## Why Not Keep Using Home DNS

If the remote endpoint uses:

```
corporate DNS for internal queries
```

but:

```
home/ISP DNS for Internet queries
```

central DNS visibility becomes incomplete.

If the organization requires centralized DNS monitoring, it is cleaner to use corporate DNS for all queries while the VPN is connected.

---

## Avoid Public DNS Fallback While Connected

Automatic fallback to public DNS can create problems.

For example:

```
erp.example.com
```

could accidentally be sent to a public resolver.

This may result in:

```
information leakage
NXDOMAIN
wrong public answer
bypassed security controls
missing central logs
```

Therefore, resiliency should come from:

```
multiple corporate DNS servers
```

rather than silent fallback to arbitrary public resolvers.

---

## DoH and DoT Consideration

If corporate policy requires centralized DNS visibility, applications using external:

```
DoH
DoT
```

can bypass the operating system resolver.

This can undermine:

```
DNS logging
filtering
policy enforcement
```

Encrypted DNS behavior is discussed in [DNS Security](../10-dns-security/dns-security.md).

---

## Zone Synchronization

The internal DNS zone is managed centrally in HQ.

Trusted branch and cloud DNS servers receive automatic copies.

Conceptually:

```
HQ Primary Authoritative DNS
        |
        | zone change
        v
Trusted secondary servers
```

---

## NOTIFY and Zone Transfer

When the zone changes, the primary can notify secondary servers.

Conceptually:

```
Primary
   |
   | NOTIFY
   v
Secondary
   |
   | check SOA serial
   v
Zone transfer
```

The secondary compares the SOA serial and synchronizes if a newer version exists.

---

## IXFR and AXFR

Two common transfer methods are:

```
IXFR
→ incremental zone transfer
```

and:

```
AXFR
→ full zone transfer
```

IXFR is useful when only part of the zone changed.

For example:

```
old serial: 2026092801
new serial: 2026092802

change:
+ app01.example.com A 10.20.30.40
```

The secondary can receive only the changes instead of the entire zone.

---

## Restrict Zone Transfers

Zone transfers should not be available to arbitrary systems.

For example:

```
HQ Primary
    |
    +--> Branch A DNS   allowed
    +--> Branch B DNS   allowed
    +--> Cloud DNS      allowed
    |
    X--> random host    denied
```

Transfers should be limited to explicitly trusted DNS servers.

Where supported and appropriate, authentication such as TSIG can provide additional protection.

---

## Dynamic DNS

Dynamic DNS updates should also follow the centralized trust model.

Workstations should not independently modify DNS records.

Instead:

```
Client
  |
  | DHCP request
  v
DHCP server
  |
  | trusted DNS update
  v
Internal DNS
```

---

## DHCP-Managed Records

Suppose a workstation receives:

```
PC-123
10.20.30.55
```

The DHCP infrastructure can create or update:

```
PC-123.example.com
→ A
→ 10.20.30.55
```

and optionally:

```
10.20.30.55
→ PTR
→ PC-123.example.com
```

This gives better control over:

```
A/PTR consistency
stale lease cleanup
auditability
record ownership
```

---

## Static Infrastructure Records

Critical infrastructure should remain centrally managed.

For example:

```
servers
routers
firewalls
load balancers
network appliances
critical services
```

should not have records changed simply because of a DHCP event.

A useful policy is:

```
Dynamic client devices
→ DHCP-managed DNS

Critical infrastructure
→ centrally managed static DNS
```

---

## TTL Strategy

A consistent default TTL is usually easier to operate than assigning different TTL values everywhere.

For example:

```
files.example.com
erp.example.com
monitoring.example.com
dc01.example.com
```

can normally use the organization's standard TTL.

TTL should be changed deliberately only when there is a clear reason.

---

## Planned DNS Changes

A lower TTL can be useful before:

```
service migration
IP address change
provider migration
load-balancer cutover
planned failover
```

A typical process is:

```
Normal TTL
    |
    v
Lower TTL before change
    |
    v
Wait for previous cache entries to expire
    |
    v
Change DNS record
    |
    v
Clients learn new answer faster
    |
    v
Restore normal TTL
```

Lowering TTL at the same time as changing the record does not remove old entries that are already cached.

Detailed caching behavior is covered in [DNS Caching and TTL](../06-caching-and-ttl/caching-and-ttl.md).

---

## Reverse DNS Design

Reverse DNS should follow the same administrative model as forward DNS.

Assume the organization uses:

```
HQ
10.10.0.0/16

Branch A
10.20.0.0/16

Branch B
10.30.0.0/16

Private Cloud
10.40.0.0/16
```

Reverse zones should be centrally managed by ICT.

Branches can receive synchronized authoritative copies where required.

---

## Central Reverse-DNS Ownership

Conceptually:

```
Central ICT / HQ
        |
        | manage PTR records
        v
Primary reverse zones
        |
        | zone transfer
        v
Branch / cloud DNS
```

This avoids each location using a different policy for PTR records.

For DHCP clients, the DHCP infrastructure can update both:

```
A
PTR
```

where appropriate.

---

## DNS Logging and SIEM Integration

DNS provides valuable security and operational telemetry.

If the organization uses a SIEM, all DNS resolvers should forward useful telemetry to it.

Possible fields include:

```
timestamp
client IP
queried FQDN
record type
response code
returned address
DNS server handling the query
query duration
cache hit/miss where available
DNSSEC status
forwarder/upstream used
allow/block decision
```

---

## Useful DNS Security Patterns

The SIEM can look for patterns such as:

```
NXDOMAIN spikes
many unique subdomains
long or high-entropy labels
unusual TXT queries
known malicious domains
newly observed domains
unexpected resolvers
DoH/DoT bypass attempts
unusual query volume
repeated SERVFAIL
repeated REFUSED
```

These patterns are indicators, not automatic proof of malicious activity.

They should be correlated with other telemetry.

---

## DNS Logging During Outages

If Branch A normally forwards public DNS through HQ:

```
Branch DNS
→ HQ DNS
```

central logs naturally contain those requests.

But if HQ fails:

```
Branch DNS
→ direct recursion
```

the branch should still log locally.

This is another reason every DNS resolver should produce its own telemetry rather than depending only on the central forwarding layer.

---

## DNS Log Retention

DNS can generate large amounts of telemetry.

A practical model may use:

```
recent detailed raw logs
→ incident investigation

longer-term normalized or summarized data
→ trends and baselines
```

The goal is to retain enough information to support investigation without storing unlimited high-volume raw DNS data unnecessarily.

---

## DNSSEC for Public DNS

DNSSEC provides authenticity and integrity validation for DNS data.

It can help protect against:

```
forged DNS responses
cache poisoning
tampering with DNS answers
```

If the managed public DNS provider supports DNSSEC reliably, enabling it is generally reasonable.

---

## Managed DNSSEC

A managed provider may handle:

```
zone signing
RRSIG generation
DNSKEY management
key rollover
```

while the organization must ensure the correct:

```
DS record
```

is published at the parent through the registrar.

Conceptually:

```
Root
 |
 v
.com
 |
 | DS
 v
example.com
 |
 | DNSKEY / RRSIG
 v
validated answer
```

---

## DNSSEC Operational Risk

DNSSEC must be operated correctly.

Problems such as:

```
wrong DS
expired RRSIG
bad DNSKEY rollover
broken chain of trust
```

can cause validating resolvers to return:

```
SERVFAIL
```

Therefore:

> Enable DNSSEC where it can be operated reliably, not simply because the feature exists.

Public DNSSEC is often easier to manage when the DNS provider automates the difficult parts.

---

## Internal DNSSEC

Internal DNSSEC is possible, but it adds operational complexity.

Whether to deploy it should depend on:

```
security requirements
threat model
regulatory requirements
operational maturity
available tooling
```

It should not automatically be assumed that every internal zone requires DNSSEC.

---

## Example Final Architecture

The resulting architecture for this organization could look like:

```
                         Internet
                            |
                            v
                Managed Public DNS Provider
                  authoritative example.com
                            |
            +---------------+---------------+
            |               |               |
            v               v               v
     www.example.com  portal.example.com  vpn.example.com


                         Corporate DNS

                              HQ
                       Central DNS Hub
                    primary internal zones
                     recursive / forwarding
                     central administration
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
      Branch A            Branch B          Private Cloud
         DNS                 DNS                 DNS
   secondary zones     secondary zones     secondary zones
   local recursion     local recursion     local recursion
          |                   |                   |
          v                   v                   v
       Clients              Clients             Workloads
```

Remote users fit into the same architecture:

```
Before VPN
---------
Public/local DNS
      |
      v
vpn.example.com
      |
      v
VPN connection


After VPN
--------
Remote workstation
      |
      v
Corporate DNS
      |
      +--> internal names
      |
      +--> public names
```

---

## Normal Branch Resolution Flow

For an internal name:

```
Branch client
     |
     v
Branch DNS
     |
     v
Local authoritative copy
     |
     v
erp.example.com
→ private address
```

For a public name:

```
Branch client
     |
     v
Branch DNS
     |
     v
HQ DNS
     |
     v
Internet DNS
     |
     v
google.com
```

---

## Branch Resolution During HQ Failure

For internal names:

```
Branch client
     |
     v
Branch DNS
     |
     v
Local authoritative copy
     |
     v
answer
```

For public names:

```
Branch client
     |
     v
Branch DNS
     |
     X
HQ unavailable
     |
     v
direct recursion
     |
     v
Root → TLD → authoritative
```

This allows the branch to remain operational without abandoning centralized control during normal conditions.

---

## Common DNS Design Mistakes

### One DNS Server for the Entire Organization

This creates:

```
single server
=
single failure point
```

---

### Two DNS Servers in the Same Failure Domain

For example:

```
DNS 1 VM
DNS 2 VM
```

on the same:

```
hypervisor
storage
switch
power source
```

may not provide meaningful resilience.

---

### Public and Internal Data Mixed Without Planning

Publishing internal records publicly can expose:

```
internal hostnames
private addressing
infrastructure details
```

---

### Internal Split Zone Missing Public Records

If internal DNS owns:

```
example.com
```

but does not contain records that internal users still need, the result may be:

```
works externally
fails internally
```

---

### Every Branch Manages Its Own DNS Independently

With one central ICT department, independent branch administration can create:

```
inconsistent records
different naming policies
duplicate entries
configuration drift
troubleshooting complexity
```

---

### Relying Only on Cache for Branch Resilience

Cache expires.

If the branch must continue resolving internal names during a WAN outage, it needs access to authoritative data.

---

### Public DNS Fallback for Corporate VPN Users

This can cause:

```
internal-name leakage
incorrect answers
bypassed filtering
missing logs
```

Redundant corporate DNS is usually preferable.

---

### Unrestricted Zone Transfers

Allowing arbitrary AXFR requests can expose zone data and increases security risk.

Zone transfers should be restricted to trusted secondary servers.

---

### Workstations Directly Updating DNS

Allowing every client to modify DNS independently increases the chance of:

```
stale records
incorrect ownership
record collisions
unauthorized changes
```

Trusted DHCP-managed updates provide better control.

---

### Different TTLs Everywhere Without Reason

Random TTL values make DNS behavior harder to understand and troubleshoot.

Use a standard policy and deviate only when required.

---

### Adding DNS Servers Without a Requirement

More DNS servers do not automatically create a better design.

Every DNS server introduces:

```
configuration
patching
monitoring
security
synchronization
troubleshooting
```

Deploy DNS infrastructure where it solves an actual availability, latency, dependency, or scale requirement.

---

## DNS Design Questions

Before deploying DNS, ask:

```
Which zones are public?

Which zones are internal?

Will split DNS be used?

Where are zones managed?

Which sites require local authoritative copies?

Which sites require local recursive resolvers?

What happens during WAN failure?

What happens during HQ failure?

Which resolvers do VPN users receive?

How are public queries forwarded?

Can branches resolve independently if required?

How are zone transfers secured?

Who is allowed to update DNS dynamically?

How are reverse zones managed?

What TTL policy is used?

Where are DNS logs collected?

What telemetry is sent to the SIEM?

Is DNSSEC required?

Are redundant servers actually in different failure domains?
```

These questions are often more important than the specific DNS software being used.

---

## Key Takeaways

- DNS architecture should be based on business requirements, not a single universal design.
- Public and internal authoritative DNS should normally be separated.
- Split DNS allows the same namespace to return different internal and public answers.
- Using the same internal and public zone requires careful synchronization of records needed in both environments.
- A managed provider is often the simplest choice for a small public DNS zone.
- HQ can act as the central DNS management and policy hub.
- Important branches can hold secondary authoritative copies of internal zones.
- Branch DNS servers can combine authoritative and recursive roles where that level of separation is sufficient.
- Local authoritative copies allow internal DNS to continue during WAN outages.
- Recursive cache alone is not a replacement for authoritative availability.
- Forward-first can preserve centralized policy while allowing branch fallback to direct recursion.
- Centralized control and availability may conflict during outages.
- DNS redundancy should be evaluated by failure domain, not only by server count.
- Smaller branches may use one local resolver with HQ as an alternative.
- Critical branches may justify two local DNS servers.
- Private-cloud DNS should be deployed when availability, dependency, latency, or workload justifies it.
- VPN users can use public DNS before tunnel establishment and corporate DNS after connection.
- Centralized DNS monitoring works best when corporate clients use corporate resolvers while connected.
- Trusted secondary DNS servers should synchronize automatically from the primary.
- IXFR and AXFR provide incremental and full zone transfer mechanisms.
- Zone transfers should be restricted to trusted systems.
- DHCP is a suitable trusted source for dynamic client DNS updates.
- Critical infrastructure records should remain centrally managed.
- Reverse DNS should follow the same central governance model as forward DNS.
- A standard TTL policy is easier to operate than arbitrary per-record values.
- TTL should be changed deliberately before planned migrations or cutovers.
- Every resolver should generate useful telemetry for central SIEM analysis.
- Public DNSSEC is usually worth considering when a managed provider can operate it reliably.
- Internal DNSSEC should be introduced based on actual security requirements and operational capability.
- More DNS infrastructure is not automatically better; each component should solve a real requirement.