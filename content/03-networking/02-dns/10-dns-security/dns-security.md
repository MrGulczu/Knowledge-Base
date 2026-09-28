# DNS Security

## Overview

DNS is a critical infrastructure service.

If DNS information is manipulated, redirected, abused, or bypassed, users can be sent to the wrong systems even when they type the correct hostname.

DNS security therefore focuses on protecting:

- DNS data,
- DNS resolution,
- DNS infrastructure,
- DNS transport,
- DNS policy,
- DNS monitoring.

This topic covers:

```
DNS filtering
DNS sinkholing
DNS spoofing
DNS cache poisoning
DNS hijacking
rogue DNS servers
DNS tunneling
DNSSEC
DoH
DoT
open resolvers
zone-transfer exposure
DNS monitoring
DNS rebinding
```

The basic DNS resolution process is covered in [DNS Resolution Process](../02-dns-resolution-process/dns-resolution-process.md).

DNS forwarding architecture is covered in [DNS Forwarding and Recursion](../07-forwarding-and-recursion/forwarding-and-recursion.md).

Internal and public DNS design is covered in [Internal and Public DNS](../09-internal-and-public-dns/internal-and-public-dns.md).

---

## DNS Filtering

**DNS filtering** applies policy to DNS queries or DNS answers.

For example, if a client requests:

```
malicious-example.com
```

the filtering resolver can block the request instead of returning the legitimate destination.

Conceptually:

```
Client
  |
  v
DNS filtering resolver
  |
  +-- allowed domain
  |      |
  |      v
  |   normal answer
  |
  +-- blocked domain
         |
         v
      policy response
```

A filtering resolver can respond in several ways.

Examples include:

```
NXDOMAIN
```

or:

```
0.0.0.0
```

or a controlled sinkhole address.

The exact behavior depends on the DNS platform and security policy.

---

## DNS Filtering and Firewalls

DNS filtering and firewall policy are related, but they are not the same mechanism.

A DNS resolver can apply policy based on:

```
domain names
categories
threat intelligence
security policy
```

A firewall can control:

```
which DNS servers clients may contact
which DNS transport is allowed
whether direct external DNS is permitted
whether DNS traffic is redirected
```

For example:

```
Client
  |
  v
Firewall
  |
  | only approved DNS allowed
  v
Internal filtering resolver
  |
  v
Internet DNS
```

This can reduce the chance that users bypass organizational DNS policy by manually selecting another resolver.

A useful distinction is:

```
DNS filtering
→ controls DNS names and answers

Firewall policy
→ controls DNS traffic paths
```

---

## DNS Sinkholing

A **DNS sinkhole** intentionally returns a controlled destination instead of the real destination for a selected DNS name.

For example:

```
malware-command.example
        |
        v
DNS security resolver
        |
        v
Sinkhole IP
10.10.50.10
```

The sinkhole destination can be used for:

- a user-facing block page,
- a controlled monitoring system,
- malware detection,
- incident-response telemetry,
- a harmless non-production destination.

---

## Why Use a Sinkhole?

Returning:

```
NXDOMAIN
```

can be enough if the only goal is to stop resolution.

However, a sinkhole can provide additional information.

For example:

```
Compromised endpoint
      |
      v
malicious-domain.example
      |
      v
DNS sinkhole
      |
      v
controlled destination
      |
      v
security monitoring
```

Administrators may then identify:

- which endpoint attempted access,
- when the attempt happened,
- how often it repeats,
- which malicious domain was queried.

A useful distinction is:

```
NXDOMAIN
→ simple blocking

Sinkhole
→ blocking plus controlled redirection
→ can provide telemetry or a block page
```

Neither method is universally better.

The correct behavior depends on the operational and security goal.

---

## DNS Spoofing

**DNS spoofing** is the broader concept of providing false DNS information.

An attacker attempts to make a client or resolver believe that a legitimate DNS name points to an incorrect destination.

For example:

```
Legitimate:

bank.example
→ 198.51.100.50
```

but an attacker causes the victim to receive:

```
bank.example
→ 203.0.113.200
```

The victim may then connect to the wrong system.

Possible impacts include:

- phishing,
- credential theft,
- malware delivery,
- traffic redirection,
- interception attempts.

---

## DNS Cache Poisoning

**DNS cache poisoning** is a specific form of DNS spoofing.

The attacker causes a recursive resolver to cache false DNS data.

For example:

```
Attacker
   |
   v
Inject false DNS information
   |
   v
Recursive resolver cache
   |
   v
bank.example
→ malicious IP
```

Clients using that resolver can then receive the poisoned answer.

Conceptually:

```
Legitimate authoritative DNS:

bank.example
→ 198.51.100.50
```

but the resolver cache contains:

```
bank.example
→ 203.0.113.200
```

Until the poisoned entry is removed or expires, multiple clients can be affected.

A useful distinction is:

```
DNS spoofing
→ false DNS information

DNS cache poisoning
→ false DNS information stored in resolver cache
```

---

## DNS Hijacking

**DNS hijacking** redirects DNS resolution itself.

Instead of only poisoning one cached record, an attacker may change where DNS queries are sent or who answers them.

Possible attack points include:

```
client DNS settings
router DNS settings
DHCP configuration
resolver configuration
domain registrar settings
authoritative DNS configuration
```

For example:

```
Client
  |
  v
Attacker-controlled DNS
  |
  v
bank.example
→ malicious IP
```

A useful distinction is:

```
Cache poisoning
→ corrupt cached DNS data

DNS hijacking
→ redirect DNS resolution toward unauthorized infrastructure
```

---

## Rogue DNS Servers

A **rogue DNS server** is an unauthorized DNS resolver used by clients.

It can appear because of:

- malicious DHCP,
- malware,
- compromised routers,
- incorrect manual configuration,
- unauthorized network equipment.

For example:

```
Client
  |
  v
Rogue DNS server
  |
  +-- bank.example → malicious IP
  +-- portal.example → malicious IP
  +-- update.example → malicious IP
```

If clients trust the rogue resolver, the attacker can influence hostname resolution.

---

## DNS Does Not Automatically Defeat TLS

Controlling DNS does not automatically mean an attacker can impersonate every HTTPS site.

For example:

```
bank.example
→ malicious IP
```

can redirect the browser to an attacker-controlled server.

However, the attacker would still need to present a certificate trusted for:

```
bank.example
```

Otherwise, the browser should detect a TLS certificate problem.

So:

```
DNS control
→ controls name-to-address mapping
```

but does not automatically control:

```
TLS certificates
application authentication
authorization
routing
```

DNS security is therefore one part of a larger security model.

---

## Preventing Rogue DNS Usage

Organizations can reduce rogue DNS risk using controls such as:

```
approved DHCP infrastructure
restricted DNS egress
firewall policy
network access controls
DNS monitoring
endpoint policy
```

For example:

```
Client
  |
  v
Firewall
  |
  +-- approved resolver → allowed
  |
  +-- arbitrary resolver → blocked
```

This helps ensure clients use the intended DNS infrastructure.

---

## DNS Tunneling

**DNS tunneling** abuses DNS queries and responses as a transport channel for data unrelated to normal name resolution.

DNS traffic is often allowed through security controls because it is required for ordinary network operation.

An attacker can abuse that trusted path.

For example:

```
Compromised endpoint
      |
      v
Encode data into DNS name
      |
      v
aGVsbG8.attacker.example
      |
      v
Corporate DNS
      |
      v
Attacker-controlled authoritative DNS
```

The attacker's DNS infrastructure receives the encoded information.

---

## DNS Tunneling Use Cases

DNS tunneling can be abused for:

```
data exfiltration
command-and-control
malware instructions
covert communication
bypassing restrictive outbound controls
```

Data can be split across multiple queries.

For example:

```
chunk1.attacker.example
chunk2.attacker.example
chunk3.attacker.example
```

The attacker can reconstruct the transmitted information from the queries received by the authoritative DNS server.

DNS responses can also carry encoded data back toward the compromised host.

---

## DNS Tunneling Indicators

Possible indicators include:

```
very long DNS names
high-entropy labels
random-looking subdomains
large numbers of unique subdomains
very high query frequency
unusual TXT queries
repeated queries to one unusual domain
```

None of these automatically proves DNS tunneling.

For example:

```
TXT
```

is also legitimately used for:

```
SPF
DKIM
DMARC
domain verification
```

Security monitoring should therefore combine multiple indicators.

---

## DNSSEC

**DNSSEC** adds cryptographic signatures to DNS data.

Its purpose is to allow validating resolvers to verify:

```
authenticity
integrity
```

of DNS information.

For example:

```
bank.example
→ A
→ 198.51.100.50
```

can be accompanied by DNSSEC records that allow the resolver to verify that the response has not been modified.

---

## DNSSEC Validation

A simplified DNSSEC validation process looks like:

```
DNS answer
   |
   v
RRSIG
digital signature
   |
   v
DNSKEY
public verification key
   |
   v
Validate signature
   |
   +-- valid
   |     |
   |     v
   |   accept DNS data
   |
   +-- invalid
         |
         v
      reject validation
```

Important DNSSEC-related record types include:

```
DNSKEY
DS
RRSIG
NSEC
NSEC3
```

The complete DNS record type reference is available in [DNS Record Types Reference](../04-dns-records/appendix/dns-records-types.md).

---

## DNSSEC Chain of Trust

DNSSEC builds a chain of trust through the DNS hierarchy.

Simplified:

```
Root
 |
 v
TLD
 |
 v
example.com
 |
 v
signed DNS records
```

A parent zone can publish information that helps validate the child zone.

The goal is to make forged or modified DNS responses detectable.

---

## What DNSSEC Does Not Do

DNSSEC does not encrypt normal DNS transport.

Someone observing the network may still be able to see:

```
which names are queried
which DNS servers are contacted
DNS response metadata
```

DNSSEC answers the question:

```
Is this DNS data authentic and unchanged?
```

It does not primarily answer:

```
Can someone observe this DNS query?
```

---

## DNS over HTTPS

**DNS over HTTPS (DoH)** transports DNS using HTTPS.

It commonly uses:

```
TCP/443
```

Conceptually:

```
Client
  |
  | encrypted HTTPS
  v
DoH resolver
```

The DNS query is protected inside encrypted HTTPS transport.

This makes passive inspection of the DNS contents more difficult for systems on the network path.

---

## DNS over TLS

**DNS over TLS (DoT)** sends DNS over a TLS-protected connection.

It commonly uses:

```
TCP/853
```

Conceptually:

```
Client
  |
  | encrypted TLS
  v
DoT resolver
```

Like DoH, its purpose includes protecting DNS traffic from passive observation and modification while in transit.

---

## DNSSEC vs Encrypted DNS

DNSSEC and encrypted DNS solve different problems.

```
DNSSEC
→ authenticity and integrity of DNS data
```

```
DoH / DoT
→ confidentiality and transport protection
```

They can be used together.

A useful comparison is:

|Mechanism|Primary Goal|
|---|---|
|DNSSEC|Verify DNS data authenticity and integrity|
|DoH|Encrypt DNS transport over HTTPS|
|DoT|Encrypt DNS transport over TLS|

---

## Encrypted DNS in Enterprise Networks

Encrypted DNS improves privacy, but unmanaged encrypted DNS can create challenges for enterprise security.

For example, an organization may intend:

```
Client
  |
  v
Corporate filtering resolver
```

but an application may instead use:

```
Application
  |
  | DoH
  v
External public resolver
```

This can bypass:

- corporate DNS filtering,
- malicious-domain blocking,
- internal DNS namespaces,
- DNS logging,
- centralized monitoring,
- policy enforcement.

DoH can be especially difficult to distinguish from ordinary HTTPS because both may use TCP/443.

---

## Open DNS Resolvers

An **open DNS resolver** accepts recursive queries from essentially any Internet client.

For example:

```
Internet client
      |
      v
Open resolver
      |
      v
Performs recursion
```

Recursive DNS normally should be restricted to trusted or authorized clients unless there is a deliberate reason to operate a public recursive service.

---

## DNS Reflection and Amplification

One of the most important risks of open resolvers is **DNS reflection/amplification DDoS**.

An attacker can spoof the victim's source address:

```
Attacker
spoofs victim IP
      |
      v
Open DNS resolver
```

The resolver sends its response to the victim:

```
Open resolver
      |
      v
Victim
```

If the DNS response is significantly larger than the original request, the resolver amplifies the attack traffic.

When many resolvers are abused simultaneously, the victim can receive very large volumes of unwanted traffic.

---

## Other Open Resolver Risks

Additional risks include:

```
resource abuse
→ CPU, memory, and bandwidth consumed by untrusted clients

cache snooping
→ possible information disclosure in some circumstances

larger attack surface
→ recursive service exposed to arbitrary Internet hosts
```

Being an open resolver does not automatically mean cache poisoning will succeed.

However, unnecessary Internet exposure gives attackers more opportunity to interact with and target the resolver.

---

## Public Authoritative DNS vs Open Recursion

Public authoritative DNS often needs to be reachable from the Internet.

That does not mean it should provide open recursive service.

A useful distinction is:

```
Public authoritative DNS
→ Internet-accessible for its authoritative zones

Recursive DNS
→ normally limited to trusted clients
```

Keeping these roles separated can reduce unnecessary exposure.

---

## Zone Transfer Exposure

DNS zone transfers are legitimate mechanisms used to synchronize authoritative DNS servers.

They are covered in [DNS Zones](../05-dns-zones/dns-zones.md).

However, unrestricted zone transfers can disclose the complete contents of a zone.

For example:

```
Attacker
   |
   | AXFR
   v
Authoritative DNS
   |
   v
Complete zone data
```

The attacker may learn:

- hostnames,
- subdomains,
- IP addresses,
- mail servers,
- name servers,
- service locations,
- naming conventions,
- TXT records,
- SRV records,
- potentially interesting infrastructure.

This information can assist reconnaissance.

---

## DNSSEC Keys and Zone Transfers

A zone transfer can expose public DNSSEC information such as:

```
DNSKEY
RRSIG
DS-related data
```

where applicable.

These are intentionally public DNS records.

Private DNSSEC signing keys should not be stored as ordinary public zone records and should not be exposed by a normal zone transfer.

---

## Restricting Zone Transfers

Zone transfers should normally be limited to authorized secondary DNS servers.

Possible controls include:

```
source-address restrictions
authenticated DNS transactions
access-control lists
platform-specific security controls
```

AXFR itself is not inherently insecure.

The security problem is allowing unauthorized systems to retrieve complete zone data.

---

## DNS Logging and Monitoring

DNS logs can provide valuable security information because many applications rely on DNS before establishing connections.

Monitoring DNS can help identify:

- malware,
- command-and-control activity,
- tunneling,
- policy bypass,
- misconfigured systems,
- suspicious domains,
- unauthorized DNS resolvers.

---

## Suspicious DNS Patterns

Useful indicators can include:

```
very long DNS names
```

Possible tunneling or encoded data.

```
high-entropy or random-looking labels
```

Possible DGA activity or tunneling.

```
large numbers of unique subdomains
```

Possible tunneling, malware, or scanning.

```
unusually frequent DNS queries
```

Possible malware, loops, tunneling, or misconfiguration.

```
large volumes of NXDOMAIN
```

Possible domain generation algorithms, incorrect configuration, or software errors.

```
unexpected TXT query patterns
```

Possible tunneling or unusual application behavior.

```
newly registered or newly observed domains
```

Potentially higher risk when combined with other indicators.

```
very low or rapidly changing TTL values
```

Potentially interesting when combined with suspicious traffic patterns.

```
queries to known malicious domains
```

Strong threat-intelligence indicator.

```
clients using unexpected external DNS servers
```

Possible policy bypass, malware, or rogue configuration.

```
unexpected DoH / DoT destinations
```

Possible bypass of centralized DNS controls.

---

## Indicators Are Not Proof

Many suspicious-looking DNS patterns also have legitimate uses.

For example:

```
low TTL
```

can support:

- failover,
- load balancing,
- cloud services,
- frequently changing infrastructure.

Likewise:

```
TXT queries
```

are normal for:

- SPF,
- DKIM,
- DMARC,
- service verification.

DNS monitoring should therefore correlate multiple signals rather than treat a single behavior as proof of malicious activity.

For example:

```
very long random subdomains
+
high query frequency
+
hundreds of unique labels
+
same uncommon domain
```

is more suspicious than any single indicator by itself.

---

## DNS Rebinding

**DNS rebinding** manipulates DNS responses so that a browser can be tricked into reaching private or local resources while using the same attacker-controlled hostname.

A simplified attack can begin with:

```
attacker.example
→ 198.51.100.50
```

The user opens the attacker's website.

Conceptually:

```
Browser
  |
  v
attacker.example
  |
  v
Public attacker server
```

The attacker later changes the DNS answer.

For example:

```
attacker.example
→ 192.168.1.10
```

or:

```
attacker.example
→ 127.0.0.1
```

The browser still refers to the hostname:

```
attacker.example
```

but subsequent traffic may now be directed toward a private or local address.

---

## Why DNS Rebinding Is Dangerous

The browser can potentially become a bridge between:

```
attacker-controlled web content
```

and:

```
private network resources
```

Conceptually:

```
Attacker-controlled page
        |
        v
Browser
        |
        v
DNS answer changes
        |
        v
Private/internal IP
        |
        v
Internal service
```

This can help an attacker attempt access to services that are not directly reachable from the Internet.

---

## DNS Rebinding Defenses

Possible defenses include:

- DNS rebinding protection,
- blocking suspicious public-to-private DNS answers,
- application authentication,
- host-header validation,
- network segmentation,
- browser protections,
- internal service hardening.

DNS should not be treated as a security boundary by itself.

Internal services should still require proper authentication and authorization.

---

## DNS Security Layers

DNS security is strongest when several controls work together.

For example:

```
Client
  |
  v
Approved DNS path
  |
  v
Filtering resolver
  |
  v
DNS logging
  |
  v
Threat intelligence
  |
  v
Validated / protected DNS infrastructure
```

Additional security controls may include:

```
DNSSEC validation
firewall rules
endpoint configuration
network segmentation
monitoring
access control
secure administrative practices
```

No single mechanism protects against every DNS-related attack.

---

## Example Enterprise DNS Security Design

A simplified enterprise model might look like:

```
Internal clients
      |
      v
Firewall
      |
      | only approved DNS allowed
      v
Corporate DNS resolvers
      |
      +-- internal zones
      |
      +-- DNS filtering
      |
      +-- logging
      |
      +-- DNSSEC validation
      |
      v
Approved upstream DNS
```

Security monitoring can then inspect:

```
query frequency
requested domains
blocked domains
NXDOMAIN patterns
resolver usage
suspicious subdomains
DoH / DoT bypass attempts
```

This provides both prevention and visibility.

---

## Common DNS Security Mistakes

### Allowing Arbitrary External DNS

Clients are allowed to query any DNS resolver on the Internet.

Possible result:

```
corporate filtering bypass
loss of DNS visibility
rogue resolver usage
```

---

### Running an Open Recursive Resolver

A resolver accepts recursion from arbitrary Internet clients.

Possible result:

```
DDoS amplification abuse
resource consumption
larger attack surface
```

---

### Allowing Unrestricted AXFR

Anyone can request a full zone transfer.

Possible result:

```
infrastructure reconnaissance
zone data disclosure
```

---

### Publishing Internal Infrastructure Publicly

Internal hostnames and addresses are stored in public DNS.

Possible result:

```
information disclosure
easier reconnaissance
```

---

### Ignoring Encrypted DNS

Applications use external DoH or DoT resolvers without organizational control.

Possible result:

```
filtering bypass
logging gaps
internal DNS failures
```

---

### Relying on DNS Filtering Alone

A blocked domain cannot resolve through the approved resolver, but the attacker may still communicate using:

```
direct IP addresses
alternate domains
encrypted tunnels
compromised legitimate services
```

DNS filtering is useful, but it is not a complete security control.

---

## DNS Security and Defense in Depth

DNS controls should be part of a broader security architecture.

For example:

```
DNS filtering
+
firewall policy
+
TLS
+
endpoint protection
+
network segmentation
+
identity controls
+
monitoring
```

This helps compensate for the limitations of any one security layer.

A useful principle is:

> DNS is an important security control point, but DNS alone should never be the only security boundary.

---

## Key Takeaways

- DNS security protects DNS data, resolution, infrastructure, transport, and policy.
- DNS filtering can block or redirect queries based on organizational policy.
- Firewalls can restrict which DNS resolvers clients are allowed to use.
- DNS sinkholing redirects selected domains to a controlled destination.
- Sinkholes can provide both blocking and security telemetry.
- NXDOMAIN can also be a valid filtering response.
- DNS spoofing provides false DNS information.
- DNS cache poisoning causes false DNS data to be stored in a resolver cache.
- DNS hijacking redirects DNS resolution toward unauthorized infrastructure.
- Rogue DNS servers can manipulate name-to-address mappings for clients that use them.
- DNS control does not automatically bypass TLS certificate validation.
- DNS tunneling abuses DNS queries and responses as a covert communication channel.
- DNS tunneling can be used for data exfiltration and command-and-control.
- DNSSEC provides DNS data authenticity and integrity.
- DNSSEC does not encrypt DNS queries.
- DoH and DoT encrypt DNS transport.
- DNSSEC and encrypted DNS solve different security problems.
- Unmanaged DoH or DoT can bypass enterprise DNS filtering and monitoring.
- Open recursive resolvers can be abused for DNS reflection and amplification attacks.
- Public authoritative DNS should not automatically provide unrestricted recursion.
- Unrestricted AXFR can expose complete zone data and assist reconnaissance.
- Public DNSSEC records are not private signing keys.
- DNS logging can provide valuable visibility into malware, tunneling, and policy bypass.
- Long random DNS names, high query frequency, many NXDOMAIN responses, and unusual TXT traffic can be suspicious indicators.
- Individual indicators are not proof of malicious activity and should be correlated with other evidence.
- DNS rebinding can trick browsers into attempting access to private or local resources.
- Internal applications should still use authentication, authorization, and network controls.
- DNS filtering is useful but should be combined with other security mechanisms.
- DNS security works best as part of a defense-in-depth architecture.