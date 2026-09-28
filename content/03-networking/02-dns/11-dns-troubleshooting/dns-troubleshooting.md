# DNS Troubleshooting

## Overview

DNS troubleshooting is most effective when the problem is broken into separate layers.

A user may report:

```
I cannot open www.example.com
```

but the actual problem could be:

- general network connectivity,
- routing,
- firewall policy,
- incorrect DNS client configuration,
- unreachable DNS resolver,
- blocked UDP or TCP port 53,
- stale cache,
- missing DNS records,
- broken delegation,
- split-DNS mismatch,
- forwarding failure,
- DNSSEC validation failure,
- application-specific DNS behavior,
- service or application failure after DNS resolution succeeds.

The goal of DNS troubleshooting is therefore to determine:

```
Is DNS actually the problem?
```

and if it is:

```
Where in the DNS path does resolution break?
```

This topic focuses on a structured troubleshooting approach using tools such as:

```
nslookup
dig
dig +tcp
dig +trace
dig -x
```

The DNS resolution process is covered in [DNS Resolution Process](../02-dns-resolution-process/dns-resolution-process.md).

Caching and TTL behavior are covered in [DNS Caching and TTL](../06-caching-and-ttl/caching-and-ttl.md).

Forwarding and recursion are covered in [DNS Forwarding and Recursion](../07-forwarding-and-recursion/forwarding-and-recursion.md).

DNSSEC and other security-related mechanisms are covered in [DNS Security](../10-dns-security/dns-security.md).

---

## Start by Separating DNS from Network Connectivity

A failed hostname does not automatically mean DNS is broken.

For example:

```
www.example.com
```

depends on both:

```
name resolution
+
network connectivity
```

A useful first step is to test basic IP reachability separately from DNS.

For example:

```
ping 1.1.1.1
```

If IP connectivity works, then test DNS separately:

```
nslookup www.example.com
```

or:

```
dig www.example.com
```

Conceptually:

```
Can I reach an IP address?
        |
        +-- No
        |    |
        |    v
        |  investigate network connectivity
        |
        +-- Yes
             |
             v
       Can DNS resolve the name?
             |
             +-- No
             |    |
             |    v
             |  investigate DNS
             |
             +-- Yes
                  |
                  v
             DNS likely works
             investigate application/service
```

---

## Ping Is Only a Quick Indicator

A failed:

```
ping 1.1.1.1
```

does not prove that Internet connectivity is unavailable.

ICMP can be blocked.

The important principle is:

> Test IP connectivity and DNS resolution separately.

Do not rely on one combined test and assume the failure is DNS-related.

---

## Check Which DNS Server the Client Is Using

If basic connectivity works but DNS resolution fails, identify the configured resolver.

On Windows:

```
ipconfig /all
```

This can show:

- DNS server addresses,
- interface configuration,
- DHCP information,
- suffix information.

On Linux systems using `systemd-resolved`:

```
resolvectl status
```

Another common source is:

```
cat /etc/resolv.conf
```

The exact behavior depends on the Linux distribution and resolver configuration.

---

## Test the Configured Resolver Directly

Suppose the client uses:

```
10.10.10.10
```

as its DNS server.

Query that resolver directly.

With `nslookup`:

```
nslookup www.example.com 10.10.10.10
```

With `dig`:

```
dig @10.10.10.10 www.example.com
```

This removes uncertainty about which DNS server is actually being queried.

Conceptually:

```
Client
  |
  v
Which resolver is configured?
  |
  v
Can that resolver answer?
```

---

## Common Client-Side Possibilities

A useful distinction is:

```
Wrong DNS server configured
→ client configuration issue
```

```
Correct DNS server but unreachable
→ routing / firewall / network issue
```

```
DNS server reachable but query fails
→ resolver / forwarding / zone / policy issue
```

This helps narrow down the investigation quickly.

---

## Check DNS Transport

Traditional DNS commonly uses:

```
UDP/53
TCP/53
```

A common mistake is allowing only UDP/53.

UDP is commonly used for normal queries, but TCP/53 is also required in some DNS situations.

Therefore, if the DNS server is reachable but queries fail, investigate:

```
client firewall
server firewall
network firewall
ACL
NAT policy
endpoint security
DNS server listening configuration
```

---

## Testing TCP Port 53 on Windows

On Windows:

```
Test-NetConnection 10.10.10.10 -Port 53
```

This tests:

```
TCP/53
```

It does not test UDP/53.

A successful TCP test therefore does not prove that normal UDP DNS is working.

---

## Testing DNS with dig

A normal `dig` query commonly uses UDP first:

```
dig @10.10.10.10 www.example.com
```

To force TCP:

```
dig +tcp @10.10.10.10 www.example.com
```

This allows UDP and TCP behavior to be compared.

---

## UDP Works but TCP Fails

If:

```
dig www.example.com
```

works, but:

```
dig +tcp www.example.com
```

fails, investigate:

```
TCP/53
```

Possible causes include:

- firewall blocking,
- ACL restrictions,
- server not listening on TCP,
- endpoint security,
- NAT problems.

A useful rule is:

```
UDP DNS works
TCP DNS fails
→ investigate TCP/53
```

---

## TCP Works but UDP Fails

The opposite situation is also possible.

If:

```
dig www.example.com
```

fails but:

```
dig +tcp www.example.com
```

works, investigate:

```
UDP/53
```

This often indicates a firewall or network-path issue affecting UDP.

---

## Understand DNS Response Codes

A DNS server can respond in several different ways.

The distinction between them is important.

---

## NXDOMAIN

`NXDOMAIN` means:

```
the queried DNS name does not exist
```

For example:

```
doesnotexist.example.com
→ NXDOMAIN
```

This means the DNS server successfully processed the query and determined that the name itself does not exist.

It is different from a timeout or server failure.

---

## SERVFAIL

`SERVFAIL` means the DNS server could not successfully complete the query.

Possible causes include:

- broken delegation,
- unreachable upstream DNS,
- resolver failure,
- authoritative server failure,
- DNSSEC validation failure,
- forwarding problems.

A useful interpretation is:

```
SERVFAIL
→ "I tried, but something went wrong."
```

---

## REFUSED

`REFUSED` means:

```
the DNS server received the query
but intentionally refuses to answer
```

Possible causes include:

- recursion disabled,
- client not authorized,
- policy restrictions,
- zone-transfer restrictions,
- DNS security controls.

A useful interpretation is:

```
REFUSED
→ "I received the query, but I will not answer it."
```

---

## Timeout

A timeout means that no usable DNS response was received within the expected time.

Possible causes include:

- unreachable DNS server,
- blocked UDP/53,
- blocked TCP/53,
- firewall drops,
- overloaded server,
- routing problems,
- upstream DNS failure.

A timeout is not equivalent to `NXDOMAIN`.

---

## Quick Response-Code Reference

```
NXDOMAIN
→ name does not exist

SERVFAIL
→ resolution failed internally

REFUSED
→ server intentionally refuses query

Timeout
→ no usable response received
```

---

## Internal vs Public DNS Comparison

Split-DNS environments can produce different answers depending on which resolver is used.

For example:

```
Internal DNS:
service.example.com
→ NXDOMAIN
```

while:

```
Public DNS:
service.example.com
→ 203.0.113.80
```

This strongly suggests:

```
internal/public DNS mismatch
```

or:

```
record missing from internal zone
```

This is especially common when the internal DNS server is authoritative for the same namespace as the public DNS.

For more detail, see [Internal and Public DNS](../09-internal-and-public-dns/internal-and-public-dns.md).

---

## Example: Works Externally but Fails Internally

Suppose:

```
service.example.com
```

was added only to public DNS.

External users resolve:

```
service.example.com
→ 203.0.113.80
```

but the internal `example.com` zone does not contain that record.

Internal users may receive:

```
NXDOMAIN
```

because the internal authoritative zone shadows the public version.

This produces the common symptom:

```
works externally
fails internally
```

---

## Check for Stale DNS Cache

If most users receive:

```
service.example.com
→ 203.0.113.80
```

but one user still reaches:

```
203.0.113.50
```

the problem may be cached DNS data.

Possible cache locations include:

```
application
operating system
local resolver
recursive resolver
```

The caching model is covered in [DNS Caching and TTL](../06-caching-and-ttl/caching-and-ttl.md).

---

## Flushing DNS Cache on Windows

On Windows:

```
ipconfig /flushdns
```

This clears the operating system DNS Client cache.

It does not necessarily clear:

- browser caches,
- application caches,
- upstream recursive resolver caches.

---

## Flushing DNS Cache on Linux

On systems using `systemd-resolved`:

```
resolvectl flush-caches
```

Some older systems may support:

```
systemd-resolve --flush-caches
```

If another caching service is in use, that specific service must be handled instead.

The exact command depends on the Linux resolver configuration.

---

## Compare Recursive and Authoritative Answers

A very useful troubleshooting technique is to query the same record from:

```
recursive resolver
```

and:

```
authoritative server
```

For example, query the normal recursive resolver:

```
dig @10.10.10.10 www.example.com A
```

Then query an authoritative server directly:

```
dig @ns1.example.com www.example.com A
```

---

## Example: Recursive Cache Problem

Suppose:

```
Recursive resolver:
www.example.com
→ 203.0.113.50
```

but:

```
Authoritative server:
www.example.com
→ 203.0.113.80
```

The authoritative data is already correct.

The problem may involve:

- stale cache,
- forwarding path,
- upstream resolver,
- resolver synchronization.

---

## Example: Authoritative Data Problem

If both return:

```
www.example.com
→ 203.0.113.50
```

but that value is incorrect, the issue is likely with the authoritative zone data itself.

A useful rule is:

```
Recursive answer differs from authoritative answer
→ investigate cache / resolver path

Both answers are wrong
→ investigate authoritative DNS data
```

---

## Forward Lookup vs Reverse Lookup

Forward and reverse DNS comparison is useful, but for a different purpose.

Forward:

```
mail.example.com
→ A
→ 203.0.113.25
```

Reverse:

```
203.0.113.25
→ PTR
→ mail.example.com
```

This checks:

```
forward/reverse consistency
```

It does not determine whether a recursive cache matches the authoritative zone.

Reverse DNS troubleshooting is covered in [Reverse DNS](../08-reverse-dns/reverse-dns.md).

---

## Check Delegation

If the authoritative server itself works but public recursive resolution fails, inspect the delegation path.

Check the parent-zone NS records.

For example:

```
example.com
→ NS ns1.example.net
→ NS ns2.example.net
```

Verify:

```
Do those servers actually host the zone?

Are they reachable?

Are the NS names correct?

Do parent and child NS records agree?
```

Incorrect delegation can cause:

```
SERVFAIL
timeouts
REFUSED
wrong answers
```

---

## Check Glue Records

Glue becomes especially important when the authoritative name server is inside the delegated namespace.

For example:

```
example.com
→ NS ns1.example.com
```

To contact:

```
ns1.example.com
```

the resolver needs its address.

Without glue, resolving the name server itself can create a dependency problem.

The parent can therefore provide:

```
ns1.example.com
→ 192.0.2.53
```

as glue.

Conceptually:

```
Parent zone
   |
   ├── NS → ns1.example.com
   |
   └── glue → 192.0.2.53
```

If glue is missing or wrong, recursive resolution can fail.

---

## Parent and Child NS Mismatch

Another possible issue is disagreement between:

```
parent delegation
```

and:

```
child-zone NS records
```

For example:

```
Parent says:
ns1.example.net
ns2.example.net
```

while the child zone publishes different NS records.

This can create inconsistent or unreliable resolution.

Useful checks include:

```
parent NS records
child NS records
authoritative server reachability
glue records
```

---

## Use dig +trace

`dig +trace` performs iterative DNS resolution and displays the delegation path.

For example:

```
dig +trace example.com
```

Conceptually:

```
Root
  |
  v
.com TLD
  |
  v
Authoritative server for example.com
  |
  v
Final answer
```

This can help identify:

- broken delegation,
- wrong NS records,
- missing glue,
- unreachable authoritative servers,
- parent/child mismatch.

---

## dig +trace vs Normal dig

A normal query:

```
dig example.com
```

typically asks the configured recursive resolver.

A trace:

```
dig +trace example.com
```

performs its own iterative resolution path.

That difference is useful.

If:

```
normal dig fails
```

but:

```
dig +trace works
```

the problem may be related to the configured recursive resolver or its forwarding path.

If the trace itself breaks, the public delegation or authoritative path may be the problem.

---

## DNSSEC Troubleshooting

DNSSEC validation failures often appear as:

```
SERVFAIL
```

even though the authoritative server contains the requested record.

Possible problems include:

```
expired RRSIG
invalid RRSIG
wrong DNSKEY
incorrect DS record
broken chain of trust
zone-signing problems
```

DNSSEC itself is covered in [DNS Security](../10-dns-security/dns-security.md).

---

## Request DNSSEC Data

With `dig`:

```
dig +dnssec example.com
```

This requests DNSSEC-related data where available.

The response may include records such as:

```
RRSIG
DNSKEY
```

However:

> `dig +dnssec` requests DNSSEC information; it does not by itself prove that the signatures are valid.

---

## Testing Validation Failure with +cdflag

A useful comparison is:

```
dig example.com
```

versus:

```
dig +cdflag example.com
```

`+cdflag` sets the Checking Disabled flag.

Suppose:

```
normal query
→ SERVFAIL
```

but:

```
query with +cdflag
→ DNS answer returned
```

That strongly suggests DNSSEC validation is involved.

Conceptually:

```
DNS data exists
        |
        v
DNSSEC validation
        |
        +-- valid
        |    |
        |    v
        |  return answer
        |
        +-- invalid
             |
             v
           SERVFAIL
```

Further investigation should inspect:

```
DS
DNSKEY
RRSIG
chain of trust
```

---

## Application Uses a Different Resolver

Sometimes command-line tools and applications behave differently.

For example:

```
nslookup www.example.com
→ fails
```

but:

```
Browser
→ website opens
```

One possible reason is that the browser uses:

```
DoH
```

or another resolver independently of the operating system.

Conceptually:

```
Operating system
      |
      | UDP/TCP 53
      v
Corporate resolver
      |
      X
DNS failure
```

while:

```
Browser
   |
   | DoH over HTTPS
   v
External resolver
   |
   v
DNS works
```

---

## Check for DoH or DoT

When applications behave differently, verify:

```
Which DNS server does the OS use?

Does the application use the OS resolver?

Is DoH enabled?

Is DoT configured?

Is an external encrypted resolver being used?

Do application and command-line tools return different results?
```

Encrypted DNS can bypass traditional DNS controls if the destination and transport are permitted.

DoH and DoT are covered in [DNS Security](../10-dns-security/dns-security.md).

---

## Troubleshooting CNAME Records

A CNAME does not provide the final IP address by itself.

For example:

```
app.example.com
→ CNAME
→ app01.example.net
```

The resolver still needs:

```
app01.example.net
→ A and/or AAAA
```

Conceptually:

```
app.example.com
        |
        v
CNAME
        |
        v
app01.example.net
        |
        v
A / AAAA
        |
        v
IP address
```

---

## CNAME Problems

Common CNAME issues include:

```
target hostname does not exist
```

```
target has no usable A/AAAA record
```

```
another broken CNAME exists in the chain
```

```
CNAME loop
```

```
DNSSEC failure on target domain
```

```
different internal/public target behavior
```

A CNAME loop might look like:

```
app.example.com
→ CNAME app01.example.net

app01.example.net
→ CNAME app.example.com
```

Resolution can never reach a final address.

---

## Troubleshooting MX Records

If email delivery to:

```
user@example.com
```

fails, inspect:

```
example.com
→ MX
```

For example:

```
example.com
→ MX 10 mail.example.com
```

Then verify that the MX target resolves to a usable IP address.

Conceptually:

```
example.com
    |
    v
MX
    |
    v
mail.example.com
    |
    v
A / AAAA
    |
    v
mail server address
```

Check:

```
Does the MX record exist?

Is the MX target correct?

Does the target hostname exist?

Does it have A and/or AAAA?

Are MX priorities correct?
```

---

## MX vs SPF, DKIM, and DMARC

These records solve different problems.

```
MX
→ where should incoming email be delivered?
```

```
SPF / DKIM / DMARC
→ authentication, authorization, and policy
```

If mail delivery routing fails, inspect MX first.

If mail is rejected, classified as spam, or fails authentication, also inspect:

```
SPF
DKIM
DMARC
```

---

## NXDOMAIN vs Missing A Record

Suppose:

```
dig mail.example.com A
```

returns:

```
NXDOMAIN
```

That means:

```
mail.example.com
does not exist
```

The MX target is broken.

This is different from:

```
NOERROR
but no A record
```

which means:

```
mail.example.com exists
but does not have an IPv4 A record
```

In that situation, check for:

```
AAAA
```

because the service may be IPv6-only.

A useful rule is:

```
NXDOMAIN
→ hostname does not exist

NOERROR + no A
→ hostname exists, but no IPv4 record
```

---

## Resolver Fails but Authoritative Server Works

Suppose:

```
dig @ns1.example.com www.example.com
```

works, but:

```
dig www.example.com
```

through the normal resolver returns:

```
SERVFAIL
```

This strongly suggests that the authoritative data exists.

Investigate:

```
recursive resolver connectivity
forwarding
delegation
UDP/53
TCP/53
DNSSEC
upstream resolvers
```

Conceptually:

```
Client
  |
  v
Recursive resolver
  |
  X
SERVFAIL
  |
  v
Authoritative server
  |
  v
correct answer exists
```

---

## Test Reverse DNS

To test a PTR record using `nslookup`:

```
nslookup 203.0.113.25
```

With `dig`:

```
dig -x 203.0.113.25
```

For IPv6:

```
dig -x 2001:db8::25
```

`dig -x` automatically constructs the correct reverse-DNS query.

Reverse DNS is covered in [Reverse DNS](../08-reverse-dns/reverse-dns.md).

---

## Check Forward and Reverse Consistency

A useful test is:

```
dig -x 203.0.113.25
```

Suppose it returns:

```
mail.example.com
```

Then query:

```
dig mail.example.com A
```

If the result includes:

```
203.0.113.25
```

the forward and reverse mappings are consistent.

This can help with:

- mail troubleshooting,
- logging,
- server identification,
- reputation checks.

---

## Structured DNS Troubleshooting Workflow

A practical workflow can look like:

```
User reports hostname problem
        |
        v
1. Test basic network connectivity
        |
        v
2. Test DNS resolution separately
        |
        v
3. Identify configured DNS resolver
        |
        v
4. Query that resolver directly
        |
        v
5. Check UDP/53 and TCP/53
        |
        v
6. Read the DNS response code
        |
        v
7. Compare internal vs public DNS
        |
        v
8. Check local and resolver cache
        |
        v
9. Compare recursive vs authoritative answer
        |
        v
10. Check NS delegation and glue
        |
        v
11. Use dig +trace if needed
        |
        v
12. Check DNSSEC if SERVFAIL persists
        |
        v
13. Follow CNAME / MX / PTR chains
        |
        v
14. Check application-specific DNS behavior
```

---

## Practical Decision Tree

```
Hostname does not work
        |
        v
Can IP connectivity be confirmed?
        |
   +----+----+
   |         |
  No        Yes
   |         |
   v         v
Network    Does DNS query work?
issue          |
         +-----+-----+
         |           |
        No          Yes
         |           |
         v           v
Check resolver    DNS probably works
configuration    investigate service
         |
         v
Can resolver be reached?
         |
   +-----+-----+
   |           |
  No          Yes
   |           |
   v           v
Routing /     Query resolver directly
firewall           |
                   v
             What response?
                   |
       +-----------+-----------+-----------+
       |           |           |           |
   NXDOMAIN     SERVFAIL     REFUSED     Timeout
       |           |           |           |
       v           v           v           v
Name missing   resolver /    policy /     transport /
or wrong zone  DNSSEC /      ACL issue    reachability
               delegation
```

---

## Useful Command Reference

### Windows

Show network and DNS configuration:

```
ipconfig /all
```

Flush local DNS cache:

```
ipconfig /flushdns
```

Query DNS:

```
nslookup www.example.com
```

Query a specific DNS server:

```
nslookup www.example.com 10.10.10.10
```

Test TCP/53:

```
Test-NetConnection 10.10.10.10 -Port 53
```

### Linux / Unix

Show resolver configuration:

```
resolvectl status
```

or:

```
cat /etc/resolv.conf
```

Normal DNS query:

```
dig www.example.com
```

Query a specific resolver:

```
dig @10.10.10.10 www.example.com
```

Force TCP:

```
dig +tcp @10.10.10.10 www.example.com
```

Trace delegation:

```
dig +trace example.com
```

Request DNSSEC data:

```
dig +dnssec example.com
```

Disable validation checking at the queried resolver:

```
dig +cdflag example.com
```

Reverse lookup:

```
dig -x 203.0.113.25
```

Flush `systemd-resolved` cache:

```
resolvectl flush-caches
```

---

## Common Symptom Reference

|Symptom|Likely Area to Investigate|
|---|---|
|IP works, hostname fails|DNS|
|DNS server IP unreachable|Routing / firewall|
|Resolver reachable, DNS times out|UDP/TCP 53, resolver service|
|NXDOMAIN internally, valid publicly|Internal zone / split DNS|
|One user gets old IP|Local/application cache|
|Recursive answer differs from authoritative|Cache / resolver path|
|Authoritative answer wrong|Zone data|
|SERVFAIL only on validating resolver|DNSSEC|
|`dig +trace` fails|Delegation / authoritative path|
|Normal `dig` fails, `+tcp` works|UDP/53|
|Normal `dig` works, `+tcp` fails|TCP/53|
|Browser works, `nslookup` fails|DoH / alternate resolver|
|MX exists but target is NXDOMAIN|Broken MX target|
|PTR exists but forward mapping differs|Forward/reverse inconsistency|
|External works, internal fails|Split DNS / internal zone shadowing|

---

## Key Takeaways

- DNS troubleshooting should first separate name resolution from general network connectivity.
- Test basic IP reachability and DNS resolution independently.
- Verify which DNS server the client is actually using.
- Query the configured DNS resolver directly.
- DNS commonly requires both UDP/53 and TCP/53.
- `Test-NetConnection` checks TCP, not UDP.
- `dig +tcp` can help identify TCP/53 problems.
- NXDOMAIN means the DNS name itself does not exist.
- SERVFAIL means the DNS server could not complete resolution successfully.
- REFUSED means the server intentionally refuses the query.
- A timeout means no usable DNS response was received.
- Internal and public DNS should be compared when split DNS is used.
- A record can work publicly but fail internally if the internal authoritative zone is missing it.
- Stale DNS data may exist in application, operating-system, or recursive-resolver caches.
- Flushing a local cache does not clear upstream caches.
- Comparing recursive and authoritative answers helps isolate cache or resolver problems.
- Forward/reverse lookup comparison checks DNS consistency, not cache freshness.
- Broken NS delegation can cause SERVFAIL, timeouts, or incorrect resolution.
- Glue records are important when authoritative name servers are inside the delegated namespace.
- `dig +trace` shows the DNS delegation path and can identify where resolution breaks.
- DNSSEC failures can appear as SERVFAIL.
- `dig +dnssec` requests DNSSEC data but does not by itself validate it.
- `dig +cdflag` can help determine whether DNSSEC validation is involved.
- Different applications may use different DNS resolvers.
- DoH and DoT can cause browsers or applications to bypass the operating system resolver.
- CNAME troubleshooting requires following the full alias chain to a usable A or AAAA record.
- MX troubleshooting requires verifying that the MX target itself resolves.
- An A record is not mandatory if a service is reachable through AAAA, but NXDOMAIN means the hostname itself does not exist.
- `dig -x` and `nslookup <IP>` can test reverse DNS.
- A structured troubleshooting process is faster and more reliable than changing DNS configuration without first identifying where the failure occurs.