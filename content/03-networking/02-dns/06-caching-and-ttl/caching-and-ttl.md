# DNS Caching and TTL

## Overview

DNS caching allows previously resolved DNS information to be reused instead of requiring a complete lookup every time the same name is requested.

Caching improves:

- performance,
- response time,
- scalability,
- resolver efficiency,
- authoritative DNS load.

The basic DNS resolution process is covered in [DNS Resolution Process](../02-dns-resolution-process/dns-resolution-process.md).

This topic focuses specifically on:

```
DNS caching
TTL
remaining TTL
positive caching
negative caching
cache expiration
DNS changes
DNS propagation
cache flushing
planned TTL changes
```

---

## Why DNS Caching Exists

Without caching, every repeated DNS request could require contacting upstream DNS servers again.

For example:

```
Client
   |
   v
Recursive resolver
   |
   v
Root
   |
   v
TLD
   |
   v
Authoritative server
```

If another client requests the same DNS name shortly afterward, repeating the entire process would be unnecessary.

Instead, the recursive resolver can reuse previously obtained information:

```
Client
   |
   v
Recursive resolver
   |
   v
Cached answer
```

This reduces:

- DNS traffic,
- lookup latency,
- authoritative server load,
- repeated recursive resolution.

Caching is one of the mechanisms that allows DNS to scale globally.

---

## What TTL Means

TTL stands for:

```
Time To Live
```

In DNS, TTL defines how long a cached DNS record may be reused before it should be considered expired.

For example:

```
www.example.com
A
203.0.113.20
TTL 3600
```

means:

```
3600 seconds
= 1 hour
```

A resolver can normally keep that answer cached for up to one hour.

Conceptually:

```
Authoritative server
returns:

www.example.com
A → 203.0.113.20
TTL 3600
        |
        v
Recursive resolver caches record
        |
        v
TTL counts down
        |
        v
3600 → 3599 → 3598 → ...
        |
        v
TTL reaches 0
        |
        v
cached entry expires
```

The important point is:

> TTL controls the lifetime of the cached copy, not the lifetime of the authoritative DNS record itself.

The authoritative record can continue to exist after the cached copy expires.

---

## Remaining TTL

When a resolver returns a cached DNS answer, it normally returns the **remaining TTL**, not the original TTL.

For example, suppose the authoritative server returns:

```
TTL 3600
```

and the recursive resolver caches the answer.

After:

```
20 minutes
= 1200 seconds
```

the remaining TTL is approximately:

```
3600 - 1200
= 2400 seconds
```

A client querying the recursive resolver could therefore receive:

```
www.example.com
A
203.0.113.20
TTL 2400
```

Conceptually:

```
Authoritative DNS
TTL 3600
      |
      v
Recursive resolver
cache starts at 3600
      |
      v
20 minutes pass
      |
      v
remaining TTL 2400
      |
      v
client receives cached answer
with TTL 2400
```

The original TTL does not restart every time another client receives the cached answer.

Otherwise, frequently requested records could remain cached indefinitely.

---

## Positive Caching

**Positive caching** means storing a successful DNS answer.

For example:

```
www.example.com
→ A
→ 203.0.113.20
```

The resolver stores the answer for the duration allowed by its TTL.

Conceptually:

```
Query
  |
  v
Successful DNS response
  |
  v
Cache result
  |
  v
Reuse until TTL expires
```

Positive caching can apply to many DNS record types, including:

```
A
AAAA
MX
NS
TXT
SRV
CAA
```

and others.

---

## Negative Caching

DNS can also cache negative answers.

**Negative caching** means remembering that requested DNS information does not exist.

For example:

```
doesnotexist.example.com
→ NXDOMAIN
```

Instead of repeatedly asking authoritative servers about the same nonexistent name, the recursive resolver can cache the negative answer for a limited period.

Conceptually:

```
Query
  |
  v
Negative response
  |
  v
Cache negative result
  |
  v
Avoid repeated unnecessary queries
```

Negative caching improves DNS efficiency in the same way as positive caching.

---

## NXDOMAIN vs Missing Record Type

Not every negative response means the same thing.

### NXDOMAIN

`NXDOMAIN` means that the requested DNS name itself does not exist.

For example:

```
doesnotexist.example.com
→ NXDOMAIN
```

### Name Exists but Record Type Does Not

A DNS name can exist while the requested record type does not.

For example:

```
server01.example.com
→ A exists
```

but:

```
Query:
MX for server01.example.com

Result:
no MX data
```

The name exists, but the requested record type does not.

Both situations can produce negative DNS caching behavior, but they represent different DNS conditions.

---

## Where DNS Can Be Cached

DNS information can be cached at multiple layers.

A simplified resolution path might look like:

```
Application
    |
    v
Operating System
    |
    v
Recursive DNS Resolver
    |
    v
Authoritative DNS Server
```

Potential caching layers include:

```
Application cache
Operating system cache
Local caching resolver
Corporate recursive resolver
ISP recursive resolver
Public recursive resolver
```

For example:

```
Browser
   |
   v
Operating system DNS cache
   |
   v
Corporate DNS resolver
   |
   v
Authoritative DNS
```

This is important because clearing one cache does not necessarily clear all others.

---

## Authoritative Data vs Cached Data

An authoritative DNS server serving its own zone is providing authoritative data.

For example:

```
Authoritative server for example.com
        |
        v
www.example.com
A → 203.0.113.20
```

A recursive resolver may store a cached copy of that answer.

For example:

```
Recursive resolver
        |
        v
Cached:
www.example.com
A → 203.0.113.20
```

The cached answer can still be valid while its TTL remains active.

Authoritative and non-authoritative answers are discussed further in [DNS Zones](../05-dns-zones/dns-zones.md).

---

## What Happens When TTL Reaches Zero

When a cached record reaches:

```
TTL = 0
```

it is no longer valid for normal reuse.

The resolver usually does not immediately perform a new query just because the TTL expired.

Instead:

```
Cached record
TTL 1
   |
   v
TTL 0
   |
   v
cache entry expires
   |
   v
wait for next request
   |
   v
new client query arrives
   |
   v
perform fresh DNS resolution
```

So the normal model is:

```
TTL expires
→ cached answer becomes invalid
→ next query triggers fresh resolution
```

Some DNS implementations can perform additional behaviors such as prefetching or serving stale answers under controlled conditions, but these are implementation-specific extensions rather than the basic caching model.

---

## DNS Changes and Cached Data

Suppose the current authoritative record is:

```
www.example.com
→ 203.0.113.20
TTL 3600
```

An administrator changes it to:

```
www.example.com
→ 203.0.113.50
```

The authoritative DNS server now contains the new value.

However, a recursive resolver may still have:

```
www.example.com
→ 203.0.113.20

Remaining TTL:
1800
```

That resolver can continue returning the old address for another 30 minutes.

Conceptually:

```
Authoritative DNS
203.0.113.50
        |
        |
        | new authoritative value
        |
        v

Recursive resolver cache
203.0.113.20
remaining TTL 1800
```

Until the cached answer expires, users of that resolver may continue reaching the old destination.

---

## DNS Propagation

The term **DNS propagation** is commonly used when describing how long a DNS change takes to become visible to users.

However, for ordinary DNS record changes, the new record is generally not pushed to every DNS resolver on the Internet.

Instead:

```
Authoritative DNS is updated
        |
        v
Some recursive resolvers still have old cached data
        |
        v
their TTL values expire
        |
        v
new queries are made
        |
        v
new authoritative data is learned
```

So in many real-world situations:

> DNS propagation is primarily the gradual expiration of cached answers and the retrieval of fresh authoritative data.

This is different from authoritative DNS synchronization.

For example, synchronization between authoritative servers can involve:

```
AXFR
IXFR
zone replication
```

as described in [DNS Zones](../05-dns-zones/dns-zones.md).

A useful distinction is:

```
Authoritative synchronization
→ distribution or replication of authoritative zone data

DNS propagation
→ commonly describes caches expiring and learning new data
```

---

## Root and Authoritative Infrastructure Changes

There are situations where DNS information really is distributed across authoritative infrastructure.

For example, if a new top-level domain were added to the DNS root zone, updated root-zone data would need to become available across the authoritative root-server infrastructure.

Conceptually:

```
Root zone updated
      |
      v
Authoritative root infrastructure
receives updated zone data
      |
      v
Root servers can answer
queries about the new TLD
```

This is different from normal recursive cache expiration after an ordinary A, AAAA, MX, or TXT record change.

---

## Flushing DNS Cache

A local DNS cache can usually be cleared manually.

Conceptually:

```
Client cache
old record
   |
   v
flush cache
   |
   v
local entry removed
```

### Windows

On Windows, the local DNS Client cache can be flushed with:

```
ipconfig /flushdns
```

A successful command normally reports that the DNS Resolver Cache was successfully flushed.

### Linux

Linux does not have one universal DNS cache because the active resolver depends on the distribution and system configuration.

On systems using `systemd-resolved`, the cache can be flushed with:

```
resolvectl flush-caches
```

Older systems may also support:

```
systemd-resolve --flush-caches
```

If a separate local caching service is used, that service must be flushed or restarted instead.

For example, with `nscd`:

```
sudo nscd -i hosts
```

or with `dnsmasq`:

```
sudo systemctl restart dnsmasq
```

The exact command therefore depends on which DNS caching service is active on the system.

After the cache is cleared, the client must request the DNS information again.

However, clearing the local cache does not guarantee that the client immediately receives fresh authoritative data.

For example:

```
Client cache
cleared
   |
   v
Recursive resolver
still has old answer cached
   |
   v
Client receives old answer again
```

The upstream resolver may still have a valid cached copy with remaining TTL.

Therefore:

> Flushing the local DNS cache only clears that particular cache layer.

It does not automatically invalidate all upstream DNS caches.

---

## TTL and Planned DNS Changes

TTL becomes especially important during planned migrations.

Suppose:

```
www.example.com
→ 203.0.113.20
```

needs to change to:

```
www.example.com
→ 203.0.113.50
```

and the current TTL is:

```
86400 seconds
= 24 hours
```

If the address is changed immediately, some recursive resolvers could continue using the old address for almost 24 hours.

A common migration approach is to lower the TTL **before** making the actual DNS change.

For example:

```
Current state
TTL 86400
        |
        v
Lower TTL to 300
        |
        v
Wait for old 86400-second caches to expire
        |
        v
Perform migration
        |
        v
Change record to new address
        |
        v
Most new cache entries expire within about 5 minutes
```

After the environment is stable, the TTL can be increased again.

---

## Why TTL Must Be Lowered in Advance

Lowering the TTL at the exact same moment as the DNS record change does not help resolvers that already cached the record using the previous higher TTL.

For example:

```
Resolver cached yesterday:

www.example.com
→ 203.0.113.20
TTL 86400
```

Today the administrator performs:

```
IP change
+
TTL change to 300
```

The resolver may still be allowed to use its old cached copy until the original TTL expires.

Therefore:

```
Lower TTL
        |
        v
Wait for previous TTL window
        |
        v
Change DNS record
```

is more effective than:

```
Change record and TTL
at the same moment
```

---

## Low TTL vs High TTL

TTL values involve a tradeoff.

### Lower TTL

Advantages:

- faster visibility of DNS changes,
- quicker recovery during migrations,
- shorter duration of stale cached answers.

Disadvantages:

- more DNS queries,
- increased resolver activity,
- increased authoritative DNS load.

Conceptually:

```
Low TTL
→ frequent refresh
→ more queries
→ faster reaction to changes
```

### Higher TTL

Advantages:

- fewer DNS queries,
- lower authoritative DNS load,
- better cache efficiency.

Disadvantages:

- changes take longer to become visible through caches,
- migrations can have longer transition periods.

Conceptually:

```
High TTL
→ longer caching
→ fewer queries
→ slower reaction to changes
```

The appropriate TTL depends on how frequently the record changes and how quickly changes need to become visible.

---

## Practical Migration Example

Suppose a company plans to move:

```
portal.example.com
```

from:

```
203.0.113.20
```

to:

```
203.0.113.50
```

The existing record is:

```
portal.example.com
A → 203.0.113.20
TTL 86400
```

### Step 1 - Lower TTL

Before the migration:

```
TTL 86400
        |
        v
TTL 300
```

### Step 2 - Wait

Wait long enough for DNS caches containing the previous:

```
TTL 86400
```

to expire.

### Step 3 - Change the Record

At migration time:

```
portal.example.com
A → 203.0.113.50
TTL 300
```

Resolvers that refresh after this point will obtain the new address and cache it only briefly.

### Step 4 - Verify

Confirm that:

```
authoritative DNS
recursive resolvers
clients
```

are receiving the expected result.

### Step 5 - Increase TTL Again

Once the migration is stable:

```
TTL 300
        |
        v
TTL 3600
```

or another operationally appropriate value.

---

## Multiple Cache Layers During Troubleshooting

Suppose a DNS record has changed, but a user still receives the old address.

Potential cache locations include:

```
Application cache
        |
        v
Operating system cache
        |
        v
Local DNS resolver
        |
        v
Corporate DNS resolver
        |
        v
Upstream recursive resolver
```

Troubleshooting should therefore identify:

1. which DNS server the client is querying,
2. what result that resolver currently returns,
3. what remaining TTL is present,
4. what the authoritative DNS server currently returns.

A local cache flush alone may not solve the issue if the stale answer exists farther upstream.

---

## Caching and Load

Caching reduces the amount of DNS traffic reaching authoritative DNS servers.

Without caching:

```
1000 clients
        |
        v
1000 repeated DNS lookups
        |
        v
authoritative DNS
```

With recursive caching:

```
1000 clients
        |
        v
recursive resolver
        |
        | one or limited upstream lookups
        v
authoritative DNS
```

The resolver can answer many clients from cache while the TTL remains valid.

This is one of the primary reasons DNS caching is important for scalability.

---

## Simple Caching Flow

A typical cached resolution process can be summarized as:

```
Client requests:
www.example.com
        |
        v
Recursive resolver checks cache
        |
        +-------------------------+
        |                         |
        v                         v
Cache hit                   Cache miss
        |                         |
        v                         v
Return cached answer       Perform resolution
with remaining TTL               |
                                  v
                           Receive authoritative data
                                  |
                                  v
                              Cache answer
                                  |
                                  v
                             Return result
```

After the TTL expires:

```
Cache entry expires
        |
        v
Next query
        |
        v
Fresh DNS resolution
```

---

## Key Takeaways

- DNS caching allows previously resolved information to be reused.
- Caching improves DNS performance, scalability, and response time.
- TTL means Time To Live.
- TTL defines how long a cached DNS answer may normally be reused.
- TTL does not define how long the authoritative record exists.
- Cached resolvers normally return the remaining TTL rather than restarting the original TTL.
- Positive caching stores successful DNS answers.
- Negative caching stores information that requested DNS data does not exist.
- NXDOMAIN means the requested DNS name does not exist.
- A DNS name can exist even when a requested record type does not.
- DNS information can be cached at multiple layers.
- Applications, operating systems, and recursive resolvers can all maintain DNS caches.
- Authoritative DNS data is different from cached recursive data.
- When TTL reaches zero, the cached record becomes invalid for normal reuse.
- Expired entries are normally refreshed when a new query requires them.
- DNS changes can appear delayed because resolvers may still have valid old cached answers.
- Ordinary DNS "propagation" is commonly the process of caches expiring and retrieving new authoritative data.
- Normal DNS record changes are not pushed to every recursive resolver on the Internet.
- Flushing a local DNS cache does not clear upstream resolver caches.
- Lowering TTL before a planned migration can reduce how long old DNS data remains cached.
- Lowering TTL only at the moment of the DNS change does not affect existing caches using the previous TTL.
- Lower TTL values provide faster change visibility but generate more DNS queries.
- Higher TTL values improve cache efficiency but make DNS changes slower to become visible.
- DNS troubleshooting should consider every caching layer between the application and the authoritative DNS server.