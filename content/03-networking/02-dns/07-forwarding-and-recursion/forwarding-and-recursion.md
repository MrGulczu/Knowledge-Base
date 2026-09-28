# DNS Forwarding and Recursion

## Overview

DNS forwarding and recursion determine how a DNS server obtains answers that it does not already have locally.

A DNS server may:

- answer from authoritative zone data,
- answer from cache,
- forward the query to another resolver,
- or perform recursive resolution itself.

The basic DNS resolution path is introduced in [DNS Resolution Process](../02-dns-resolution-process/dns-resolution-process.md).

Caching behavior is covered in [DNS Caching and TTL](../06-caching-and-ttl/caching-and-tll.md).

This topic focuses on:

```
recursive queries
iterative queries
recursive resolvers
authoritative servers
DNS forwarding
conditional forwarding
forward-first
forward-only
root hints
forwarding chains
forwarding loops
```

---

## Recursive Resolution

A **recursive resolver** is responsible for finding a complete DNS answer on behalf of a client.

For example:

```
Client asks:

www.example.com
```

The client sends the query to a recursive resolver and expects the resolver to return the final answer.

Conceptually:

```
Client
  |
  | recursive query
  v
Recursive resolver
  |
  v
Find complete answer
  |
  v
Return result to client
```

The client does not normally walk the DNS hierarchy itself.

Instead, the recursive resolver performs the necessary work.

A useful rule is:

```
Recursive resolver
→ finds answers on behalf of clients
```

---

## Recursive Queries

A **recursive query** asks a DNS resolver to obtain the complete answer.

Conceptually:

```
Client
  |
  | "Find the answer for me"
  v
Recursive resolver
```

The resolver can then:

- answer from cache,
- answer from local authoritative data,
- forward the query,
- or perform recursive resolution itself.

The client expects a final response rather than a referral telling it where to ask next.

---

## Iterative Queries

When a recursive resolver performs the resolution process itself, it normally sends **iterative queries** to the DNS hierarchy.

An iterative query means:

> Give me the best information you currently have.

The responding server can provide:

- a final authoritative answer,
- a referral to another DNS server,
- or another valid DNS response.

Conceptually:

```
Recursive resolver
    |
    | iterative query
    v
Root server
    |
    | referral
    v
TLD server
    |
    | referral
    v
Authoritative server
    |
    | final answer
    v
Recursive resolver
```

The important distinction is:

```
Client → Recursive resolver
recursive query

Recursive resolver → DNS hierarchy
iterative queries
```

---

## Recursive Resolver vs Authoritative Server

A recursive resolver and an authoritative DNS server have different roles.

### Recursive Resolver

A recursive resolver:

- receives client queries,
- checks its cache,
- follows DNS referrals when necessary,
- contacts authoritative servers,
- returns the final answer to the client.

Conceptually:

```
Recursive resolver
→ finds answers
```

### Authoritative DNS Server

An authoritative DNS server:

- stores authoritative zone data,
- answers for zones it is responsible for,
- provides authoritative responses,
- can return referrals for delegated child zones.

Conceptually:

```
Authoritative server
→ provides authoritative answers
  for its own zones
```

The authoritative server does not normally perform recursive resolution for unrelated names.

For more information about authoritative zones, see [DNS Zones](../05-dns-zones/dns-zones.md).

---

## DNS Forwarding

A DNS server does not always need to perform recursion itself.

Instead, it can **forward** a query to another recursive resolver.

For example:

```
Client
  |
  v
Internal DNS server
  |
  | forward query
  v
Upstream recursive resolver
  |
  v
DNS hierarchy
  |
  v
Authoritative server
```

The internal DNS server asks the upstream resolver to obtain the final answer.

A useful distinction is:

```
Forwarding
→ ask another resolver to resolve the query

Recursion
→ resolve the query by following DNS referrals yourself
```

A DNS forwarder is therefore normally another recursive resolver, not the authoritative server for the requested domain.

---

## Why Use DNS Forwarding

Forwarding can provide several operational benefits.

These include:

- centralized DNS resolution,
- centralized logging,
- DNS filtering,
- security inspection,
- policy enforcement,
- reduced direct Internet DNS traffic,
- simplified branch-office DNS design,
- controlled upstream dependencies.

For example:

```
Branch clients
      |
      v
Branch DNS
      |
      v
Corporate resolver
      |
      v
Approved Internet resolver
```

This can ensure that Internet DNS queries pass through approved infrastructure.

---

## Conditional Forwarding

**Conditional forwarding** sends queries for a specific DNS namespace to a specific DNS server.

For example:

```
Query:
host.branch.example.com
        |
        v
Match:
branch.example.com
        |
        v
Forward to:
10.20.0.10
```

Other queries can use the normal forwarding path.

For example:

```
www.microsoft.com
        |
        v
General forwarder
```

The rule is:

```
Conditional forwarding
→ specific domain
→ specific DNS resolver
```

Conditional forwarding is useful in environments with:

- branch offices,
- separate internal domains,
- partner networks,
- hybrid environments,
- mergers,
- private namespaces,
- isolated infrastructure.

The purpose is to ensure that a query reaches the DNS server that actually knows how to resolve that namespace.

---

## Example Conditional Forwarding Design

Consider:

```
corp.example.com
branch.example.com
```

The central resolver may use:

```
branch.example.com
→ forward to branch DNS
```

while all unrelated queries use:

```
Internet queries
→ general upstream resolver
```

Conceptually:

```
                    Internal DNS
                         |
          +--------------+--------------+
          |                             |
          v                             v
branch.example.com                all other names
          |                             |
          v                             v
Branch DNS                      General forwarder
```

This allows namespace-specific DNS routing.

---

## Forward-First

With **forward-first**, the DNS server tries the configured forwarder first.

If the forwarder fails or does not respond, the DNS server may fall back to performing recursive resolution itself.

Conceptually:

```
Client query
    |
    v
Internal DNS
    |
    v
Try forwarder
    |
    +------------------+
    |                  |
 success              failure
    |                  |
    v                  v
return answer      perform recursion
                       |
                       v
                  Root → TLD → Authoritative
```

A useful rule is:

```
Forward-first
→ try forwarder
→ if forwarding fails, perform recursion
```

Forward-first can provide better resilience because the resolver can still resolve names if the upstream forwarder becomes unavailable.

However, it can also bypass centralized DNS controls if direct recursion is allowed.

---

## Forward-Only

With **forward-only**, the DNS server depends entirely on the configured forwarder.

Conceptually:

```
Client query
    |
    v
Internal DNS
    |
    v
Forwarder
    |
    +-------------+
    |             |
 success        failure
    |             |
    v             v
answer         query fails
```

The server does not fall back to recursive resolution using the DNS hierarchy.

A useful rule is:

```
Forward-only
→ use configured forwarder only
→ no fallback recursion
```

Forward-only can be useful when:

- DNS must pass through approved resolvers,
- filtering is centralized,
- logging is centralized,
- direct DNS access to the Internet is blocked,
- security policy requires a controlled DNS path.

The tradeoff is that the forwarder becomes an important dependency.

---

## Root Hints

If a recursive resolver is going to perform recursion itself, it needs a starting point.

That starting point is provided by **root hints**.

Root hints contain information about the DNS root servers.

Conceptually:

```
Recursive resolver
      |
      | uses local root hints
      v
Root server
      |
      | referral
      v
TLD server
      |
      | referral
      v
Authoritative server
```

A useful rule is:

```
Root hints
→ starting points for recursive DNS resolution
```

Root hints do not contain the final answer for the requested name.

They tell the resolver which root DNS servers it can contact to begin resolution.

---

## Root Hints and Forwarding

Whether root hints are used depends on the resolver configuration.

### Forward-Only

```
Query
  |
  v
Forwarder
  |
  +-- success → return answer
  |
  +-- failure → query fails
```

Root hints are not used as fallback.

### Forward-First

```
Query
  |
  v
Forwarder
  |
  +-- success → return answer
  |
  +-- failure
          |
          v
      use root hints
          |
          v
      Root → TLD → Authoritative
```

The resolver does not ask a root server "for hints."

The root hints already exist locally and identify which root servers can be contacted.

---

## Cached Referral Information

A recursive resolver does not necessarily begin at the root for every uncached final answer.

It may already have useful delegation information cached.

For example:

```
example.com
→ authoritative NS information cached
```

The resolver may be able to skip the root and TLD steps and query the authoritative server directly.

Conceptually:

```
Recursive resolver
        |
        | authoritative server already known
        v
Authoritative server
        |
        v
final answer
```

Caching therefore improves not only final-answer performance but also the efficiency of recursive resolution.

For more detail, see [DNS Caching and TTL](../06-caching-and-ttl/caching-and-tll.md).

---

## Multiple Forwarding Layers

Large environments can have more than one forwarding layer.

For example:

```
Client
  |
  v
Branch DNS
  |
  v
Corporate resolver
  |
  v
Central security resolver
  |
  v
Internet DNS
```

This architecture can provide:

- centralized filtering,
- centralized logging,
- traffic control,
- namespace-specific routing,
- policy enforcement,
- simplified branch configurations.

However, additional forwarding layers also introduce disadvantages.

These include:

- additional latency for uncached queries,
- more infrastructure dependencies,
- more failure points,
- more complex troubleshooting,
- longer query paths.

Conceptually:

```
More forwarding layers
→ more control
→ more dependencies
→ potentially more latency
```

Caching can reduce the performance impact because frequently requested answers can often be returned before the entire chain is traversed.

---

## Forwarding Dependencies

Consider:

```
Client
  |
  v
Branch DNS
  |
  v
Corporate DNS
  |
  v
Central Resolver
```

If:

```
Branch DNS        OK
Corporate DNS     OK
Central Resolver  DOWN
```

external DNS resolution may still fail.

This means forwarding designs should consider:

- redundancy,
- multiple forwarders,
- failure behavior,
- network reachability,
- monitoring,
- whether fallback recursion is allowed.

A forwarding architecture should avoid creating unnecessary single points of failure.

---

## Forwarding Loops

Incorrect DNS forwarding configuration can create a **forwarding loop**.

For example:

```
DNS Server A
    |
    | forwards to
    v
DNS Server B
    |
    | forwards to
    v
DNS Server A
```

A query can circulate between the two resolvers:

```
Client
  |
  v
DNS A
  |
  v
DNS B
  |
  v
DNS A
  |
  v
DNS B
  |
  v
...
```

The request eventually fails because of timeout or resolver loop-protection behavior.

The important rule is:

> A forwarding path must eventually terminate at a resolver that can answer from cache, authoritative data, or perform successful recursive resolution.

Forwarding loops are especially important to avoid in environments with:

- branch resolvers,
- central DNS servers,
- conditional forwarding,
- partner DNS,
- hybrid networks,
- multiple internal namespaces.

---

## Example Enterprise DNS Flow

Consider an organization with:

```
branch.example.com
corp.example.com
Internet DNS
```

A branch DNS server might be configured as:

```
branch.example.com
→ answer from local authoritative data

corp.example.com
→ conditional forward to central corporate DNS

everything else
→ forward to approved recursive resolver
```

Conceptually:

```
                    Branch DNS
                        |
        +---------------+---------------+
        |               |               |
        v               v               v
branch.example.com  corp.example.com   Internet names
        |               |               |
        v               v               v
 local zone       corporate DNS    approved resolver
```

This allows DNS traffic to follow different paths based on namespace and administrative requirements.

---

## Forwarding vs Delegation

Forwarding and delegation solve different problems.

### Forwarding

Forwarding determines:

```
Where should this resolver send a query?
```

Example:

```
branch.example.com
→ forward query to 10.20.0.10
```

This is resolver behavior.

### Delegation

Delegation determines:

```
Which authoritative DNS servers are responsible for this namespace?
```

Example:

```
example.com
        |
        | NS delegation
        v
branch.example.com
```

This is part of the authoritative DNS hierarchy.

A useful distinction is:

```
Forwarding
→ resolver configuration

Delegation
→ authoritative namespace structure
```

Delegation is covered in [DNS Hierarchy](../03-dns-hierarchy/dns-hierarchy.md).

---

## Forwarding vs Caching

Forwarding and caching are also different mechanisms.

A resolver can answer from cache without forwarding anything.

For example:

```
Client query
    |
    v
Internal DNS
    |
    v
Cache hit
    |
    v
Return answer
```

Only when the resolver does not already have the necessary answer does forwarding or recursion become necessary.

Conceptually:

```
Query
  |
  v
Check cache
  |
  +----------------------+
  |                      |
  v                      v
Cache hit             Cache miss
  |                      |
  v                      v
return answer      forward or recurse
```

---

## Recursive Resolution Example

Suppose the resolver needs:

```
www.example.com
```

and has no useful cached information.

The process may look like:

```
Client
  |
  | recursive query
  v
Recursive resolver
  |
  | iterative query
  v
Root server
  |
  | referral to .com
  v
.com TLD server
  |
  | referral to example.com authoritative servers
  v
Authoritative server
  |
  | final answer
  v
Recursive resolver
  |
  | cache answer
  v
Client
```

If the resolver already has the `.com` or `example.com` delegation cached, some of these steps can be skipped.

---

## Forwarding Example

The same client query can follow a different path if forwarding is configured.

```
Client
  |
  | recursive query
  v
Internal DNS
  |
  | forwarded recursive query
  v
Upstream recursive resolver
  |
  v
Perform resolution
  |
  v
Return final answer
  |
  v
Internal DNS
  |
  v
Client
```

The internal DNS server delegates the recursive work to the upstream resolver.

---

## Design Considerations

DNS forwarding design should consider:

- which DNS servers are authoritative,
- which servers perform recursion,
- which servers forward,
- where Internet queries should go,
- how private namespaces are resolved,
- whether conditional forwarding is required,
- whether direct recursion should be allowed,
- what happens if an upstream resolver fails,
- whether DNS filtering must be enforced,
- how logging and monitoring are centralized,
- whether the design introduces forwarding loops.

A simple architecture is often easier to troubleshoot than a long chain of forwarding dependencies.

---

## Troubleshooting Forwarding

When DNS forwarding fails, useful questions include:

```
Can the client reach its configured DNS server?

Can that DNS server answer from cache?

Is the query being conditionally forwarded?

Can the resolver reach the configured forwarder?

Is the forwarder responding?

Is forward-only or forward-first configured?

Can fallback recursion reach root servers?

Is a firewall blocking UDP or TCP port 53?

Is there a forwarding loop?

Does the upstream resolver know how to resolve the requested namespace?
```

The exact commands and configuration steps depend on the operating system and DNS implementation and belong in the relevant systems-administration topics.

---

## Key Takeaways

- A recursive resolver finds complete DNS answers on behalf of clients.
- A recursive query asks another resolver to obtain the final answer.
- Recursive resolvers normally use iterative queries when walking the DNS hierarchy themselves.
- An iterative query asks a DNS server for the best information it currently has.
- Root and TLD servers normally provide referrals rather than performing recursion for the querying resolver.
- Authoritative DNS servers provide authoritative data for their own zones.
- DNS forwarding sends a query to another resolver instead of performing the recursive lookup locally.
- A forwarder is normally another recursive resolver, not the authoritative server for the requested domain.
- Conditional forwarding sends queries for specific namespaces to specific DNS servers.
- Forward-first tries a forwarder first and can fall back to normal recursive resolution.
- Forward-only depends entirely on the configured forwarder.
- Root hints provide recursive resolvers with starting points for contacting the DNS root infrastructure.
- Root hints are stored locally; they are not requested dynamically from a root server.
- Cached delegation information can allow a resolver to skip parts of the DNS hierarchy.
- Multiple forwarding layers can improve centralized control, filtering, and logging.
- Additional forwarding layers also increase latency, complexity, and infrastructure dependencies.
- Forwarding loops occur when DNS servers forward queries back toward each other.
- Forwarding paths should eventually terminate at a resolver that can answer the query or perform successful recursion.
- Forwarding is resolver behavior, while delegation defines authoritative DNS responsibility.
- Cache lookup normally happens before forwarding or recursive resolution is required.
- DNS forwarding design should balance security, control, resilience, simplicity, and performance.