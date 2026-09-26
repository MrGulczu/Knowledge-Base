# DNS Resolution Process

## Overview

DNS resolution is the process used to obtain DNS information for a requested name.

A common example is resolving:

```
www.example.com
```

into an IP address that an application can use to communicate with the destination.

The basic DNS roles, including the local resolver, recursive resolver, authoritative DNS server, DNS hierarchy, local cache, and hosts file, are introduced in [DNS Fundamentals](../01-dns-fundamentals/dns-fundamentals.md).

This topic focuses on how those components interact during an actual DNS lookup.

A simplified resolution path looks like this:

```
Application
    |
    v
Local Resolver
    |
    v
Configured DNS Server
    |
    v
Recursive Resolution
    |
    v
Authoritative Answer
    |
    v
Configured DNS Server
    |
    v
Local Resolver
    |
    v
Application
```

The complete process can be shorter or longer depending on:

- information already available locally,
- cached DNS information,
- DNS server configuration,
- forwarding configuration,
- DNS aliases,
- availability of DNS servers,
- the type of response returned.

---

## Example Environment

The examples in this topic use the following environment:

```
Workstation:
192.168.10.25

Configured DNS Server:
192.168.10.10

Requested Name:
www.example.com
```

We will initially assume:

```
Workstation DNS cache:
No matching entry

Hosts file:
No matching entry

DNS Server cache:
No matching entry

DNS Server:
Allowed to perform recursive resolution
```

The workstation therefore needs to request the information from DNS infrastructure.

---

## Step 1 - The Application Requests Name Resolution

The process begins when an application needs information about a DNS name.

For example, a user enters:

```
www.example.com
```

into a browser.

The browser does not normally perform the complete DNS hierarchy lookup itself.

Instead, the application asks the operating system's local resolver to resolve the name.

Conceptually:

```
Browser
   |
   | Resolve www.example.com
   v
Operating System
DNS Resolver
```

The role of the client resolver is covered in more detail in [DNS Fundamentals](../01-dns-fundamentals/dns-fundamentals.md).

---

## Step 2 - The Local Resolver Checks Local Information

Before sending a DNS query across the network, the operating system may be able to satisfy the request locally.

Possible sources can include:

- local DNS cache,
- hosts file,
- other operating-system-specific name-resolution mechanisms.

A simplified view is:

```
Application
    |
    v
Local Resolver
    |
    ├── DNS cache
    |
    ├── hosts file
    |
    └── no usable local answer
            |
            v
        DNS query
```

The exact lookup order depends on the operating system and resolver configuration.

The difference between the DNS cache and hosts file is already covered in [DNS Fundamentals](../01-dns-fundamentals/dns-fundamentals.md).

### Local Cache Hit

If the requested information is already cached and still valid, the resolver may return it immediately.

For example:

```
Local DNS Cache:

www.example.com
→ 203.0.113.20
```

In this situation:

```
Application
    |
    v
Local Resolver
    |
    | cache hit
    v
203.0.113.20
```

No DNS query needs to leave the workstation.

### Local Cache Miss

If no usable local information exists:

```
www.example.com
→ not found locally
```

the resolver must contact one of its configured DNS servers.

---

## Step 3 - The Resolver Selects a Configured DNS Server

A workstation can have one or more DNS servers configured.

For example:

```
DNS Servers:

192.168.10.10
192.168.10.11
```

These addresses may have been configured manually or supplied automatically, commonly through DHCP.

The basic relationship between static configuration, DHCP, and DNS server configuration is covered in [DNS Fundamentals](../01-dns-fundamentals/dns-fundamentals.md). General IP configuration and DHCP basics are introduced in [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md).

The resolver selects an appropriate configured DNS server according to the behavior of the operating system.

For this example:

```
Selected DNS Server:
192.168.10.10
```

---

## Step 4 - The Client Creates a DNS Query

The local resolver creates a DNS request for the required information.

Conceptually:

```
Question:

What address information exists for
www.example.com?
```

DNS queries are application-layer messages.

Before the request can travel across the network, it is encapsulated using the lower networking layers.

The general encapsulation process is already covered in [Network Models](../../01-fundamentals/01-network-models/network-models.md), so it is not repeated in detail here.

Traditional DNS commonly uses:

```
UDP/53
```

and can also use:

```
TCP/53
```

when required.

The differences between TCP and UDP are covered in [TCP and UDP](../../01-fundamentals/09-tcp-udp/tcp-udp.md), while the role of port numbers is covered in [Ports and Sockets](../../01-fundamentals/10-ports-sockets/ports-sockets.md).

---

## Step 5 - The Query Reaches the Configured DNS Server

The client sends the query to:

```
192.168.10.10
```

Conceptually:

```
Workstation
192.168.10.25
     |
     | DNS query:
     | www.example.com?
     v
DNS Server
192.168.10.10
```

The configured DNS server may act as a recursive resolver for the client.

Its job is to provide the client with a usable answer if possible.

---

## Step 6 - The DNS Server Checks Its Own Cache

Before performing additional DNS queries, the recursive resolver can check whether it already has valid information cached.

For example:

```
Recursive Resolver Cache:

www.example.com
→ 203.0.113.20
```

### Resolver Cache Hit

If valid information exists:

```
Client
   |
   v
Recursive Resolver
   |
   | cache hit
   v
203.0.113.20
```

the resolver can return the answer without contacting the wider DNS hierarchy.

This is one of the main reasons DNS caching significantly reduces repeated DNS traffic.

### Resolver Cache Miss

If the resolver has no valid cached answer:

```
www.example.com
→ not found in cache
```

it must obtain the information elsewhere.

This is where recursive resolution begins.

---

## Recursive and Iterative Queries

The difference between **recursive** and **iterative** behavior is central to understanding DNS resolution.

### Recursive Query

A recursive query effectively means:

> Find the complete answer for me.

The client normally expects its recursive resolver to perform the necessary work.

Conceptually:

```
Client
  |
  | Recursive request
  | "Resolve www.example.com for me"
  v
Recursive Resolver
```

The client does not normally follow DNS referrals itself.

### Iterative Query

An iterative query effectively means:

> Give me the best information you currently have and I will continue the lookup.

This is commonly how a recursive resolver communicates with the DNS hierarchy.

For example:

```
Recursive Resolver
      |
      | query
      v
Root DNS Server
      |
      | referral
      v
Recursive Resolver
```

The resolver receives information about where it should ask next and continues the process itself.

A simplified rule is:

```
Client
→ Recursive Resolver
→ asks for a complete answer

Recursive Resolver
→ DNS hierarchy
→ follows referrals
```

A DNS server can also forward a recursive query to another recursive resolver, so recursive and iterative behavior should not be understood simply as "clients versus servers."

---

## Step 7 - The Recursive Resolver Starts with the DNS Root

If the resolver has no cached information that lets it skip part of the hierarchy, it can begin with the DNS root.

The requested name is:

```
www.example.com.
```

The final dot represents the DNS root, as explained in [DNS Fundamentals](../01-dns-fundamentals/dns-fundamentals.md).

The hierarchy is:

```
.
└── com.
    └── example.com.
        └── www.example.com.
```

The recursive resolver asks a root DNS server about:

```
www.example.com
```

Conceptually:

```
Recursive Resolver
      |
      | "Where can I find www.example.com?"
      v
Root DNS Server
```

---

## Root Server Response - Referral to the TLD

The root server is authoritative for the DNS root zone:

```
.
```

It is not the authoritative server for:

```
example.com
```

and does not normally contain the final address record for:

```
www.example.com
```

Instead, it knows where responsibility for top-level domains has been delegated.

For this query, it can return information about the DNS servers responsible for:

```
.com
```

Conceptually:

```
Root DNS Server
      |
      | "Ask the .com name servers"
      v
Recursive Resolver
```

This is a **referral**, not the final answer.

The resolver now knows where to continue.

---

## Step 8 - The Resolver Queries the .com TLD Servers

The resolver sends another query, this time to a DNS server responsible for the `.com` top-level domain.

```
Recursive Resolver
      |
      | www.example.com?
      v
.com TLD DNS Server
```

The `.com` server is authoritative for the `.com` zone.

However, it is not normally the server that stores the final record for:

```
www.example.com
```

Instead, it contains delegation information showing which DNS servers are authoritative for:

```
example.com
```

The `.com` server therefore returns another referral.

Conceptually:

```
.com TLD Server
      |
      | "These name servers are
      |  authoritative for example.com"
      v
Recursive Resolver
```

---

## NS Records in a Referral

The referral commonly identifies authoritative name servers using **NS records**.

A simplified example could look like:

```
example.com
→ ns1.example-dns.net
→ ns2.example-dns.net
```

The resolver now knows the names of the DNS servers responsible for `example.com`.

However, there is an immediate question:

```
How does the resolver reach
ns1.example-dns.net
if that is also a DNS name?
```

The resolver needs an IP address for the authoritative name server before it can send a query to it.

---

## Glue Records

A referral can include additional address information that helps the resolver contact the referred name server.

This information is commonly referred to as **glue**.

Conceptually:

```
Authority information:

example.com
→ ns1.example-dns.net

Additional information:

ns1.example-dns.net
→ 192.0.2.53
```

The resolver can then contact:

```
192.0.2.53
```

without first becoming stuck trying to resolve the name of the DNS server required to continue the lookup.

Glue records are particularly important when the names of authoritative servers create a dependency that would otherwise prevent resolution from progressing.

The detailed behavior of DNS delegation and hierarchy is covered later in the dedicated DNS hierarchy topic.

---

## Step 9 - The Resolver Queries the Authoritative DNS Server

The recursive resolver can now query a DNS server authoritative for:

```
example.com
```

Conceptually:

```
Recursive Resolver
      |
      | www.example.com?
      v
Authoritative DNS Server
for example.com
```

The authoritative server contains the DNS information for the zone it manages.

The distinction between recursive and authoritative DNS servers is introduced in [DNS Fundamentals](../01-dns-fundamentals/dns-fundamentals.md).

---

## Final Authoritative Answer

If the authoritative server contains a direct address record for the requested name, it can return the relevant answer.

For example:

```
www.example.com
→ 203.0.113.20
```

Conceptually:

```
Authoritative Server
      |
      | final answer
      v
Recursive Resolver

www.example.com
→ 203.0.113.20
```

This differs from the previous root and TLD responses.

The root and TLD servers returned **referrals**.

The authoritative server can return the requested authoritative information.

A useful distinction is:

```
Referral
→ tells the resolver where to ask next

Final answer
→ contains the requested DNS information
```

---

## Resolution When the Name Is an Alias

The requested name does not always directly map to an IP address.

For example:

```
www.example.com
```

could be an alias for another name:

```
www.example.com
→ web01.example.net
```

In that situation, the resolver may need to continue resolution for:

```
web01.example.net
```

The process can therefore look like:

```
www.example.com
        |
        v
Alias
        |
        v
web01.example.net
        |
        v
Resolve target name
        |
        v
203.0.113.20
```

If the alias points into another DNS namespace, the authoritative server for `example.com` may not itself know the final IP address.

The recursive resolver continues resolving the target until it obtains the required final information.

DNS aliases and their associated record types are covered in detail in the dedicated DNS records topic.

---

## Step 10 - The Answer Returns to the Recursive Resolver

Once the recursive resolver obtains the required answer:

```
www.example.com
→ 203.0.113.20
```

it can return that information to the client.

Before or while doing so, it can also store the information in its own cache.

Conceptually:

```
Authoritative DNS Server
        |
        | answer + TTL
        v
Recursive Resolver
        |
        ├── cache answer
        |
        └── return answer to client
```

The response normally contains a **TTL - Time To Live** value that controls how long the record may remain cached.

The basic role of TTL is introduced in [DNS Fundamentals](../01-dns-fundamentals/dns-fundamentals.md). Caching behavior is covered in greater detail in the dedicated caching and TTL topic.

---

## Remaining TTL

Suppose the authoritative answer contains:

```
www.example.com
→ 203.0.113.20

TTL:
3600 seconds
```

The recursive resolver may cache the information.

As time passes, the remaining lifetime decreases.

For example:

```
Initial TTL:
3600

After 10 minutes:
approximately 3000

After 30 minutes:
approximately 1800

After 1 hour:
expired
```

If another client queries the recursive resolver while the entry is still valid, the resolver can return the cached result with its remaining TTL.

The original TTL is not simply restarted every time another client asks for the same information.

When the cached data expires, it must no longer be treated as a normal valid cached answer and fresh information must be obtained when needed.

---

## Step 11 - The Client Receives and May Cache the Answer

The recursive resolver returns the DNS response to the workstation:

```
DNS Server
192.168.10.10
     |
     | www.example.com
     | → 203.0.113.20
     v
Workstation
192.168.10.25
```

The workstation's local resolver may also cache the information.

This means caching can occur at multiple stages:

```
Authoritative Server
        |
        v
Recursive Resolver Cache
        |
        v
Client Resolver Cache
        |
        v
Application
```

A later lookup may therefore be answered:

- from the client cache,
- from the recursive resolver cache,
- or through a new DNS resolution process.

---

## Step 12 - The Application Receives the Result

After resolution succeeds, the operating system provides the result to the application.

For example:

```
Browser requested:

www.example.com
```

The resolver returns:

```
203.0.113.20
```

The browser can then begin the actual communication with the destination.

DNS resolution itself does not establish the application connection.

Instead:

```
DNS
→ discovers the required addressing information

Networking stack
→ performs the actual communication
```

The underlying routing, IP addressing, transport protocols, and encapsulation process are already covered throughout Networking Fundamentals.

Useful references include:

- [Network Models](../../01-fundamentals/01-network-models/network-models.md)
- [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md)
- [TCP and UDP](../../01-fundamentals/09-tcp-udp/tcp-udp.md)
- [Ports and Sockets](../../01-fundamentals/10-ports-sockets/ports-sockets.md)

---

## Complete Resolution Flow

The complete process can now be represented as:

```
Application
    |
    | needs www.example.com
    v
Local Resolver
    |
    ├── DNS cache
    ├── hosts file
    |
    | no local answer
    v
Configured DNS Server
Recursive Resolver
    |
    ├── resolver cache
    |
    | cache miss
    v
Root DNS Server
    |
    | referral to .com
    v
.com TLD DNS Server
    |
    | referral to example.com
    v
Authoritative DNS Server
for example.com
    |
    | final answer
    v
Recursive Resolver
    |
    ├── cache answer
    |
    v
Client Resolver
    |
    ├── may cache answer
    |
    v
Application
    |
    v
Uses returned IP address
to begin communication
```

This is the full conceptual path when no useful information is already cached.

In real environments, the process is often shorter because resolvers commonly already possess cached information about:

- TLD name servers,
- authoritative name servers,
- previous answers,
- delegation information.

---

## DNS Resolution with a Forwarder

A DNS server does not always perform the hierarchy lookup itself.

It may instead be configured to send unresolved queries to another DNS resolver known as a **forwarder**.

For example:

```
Workstation
    |
    v
Internal DNS Server
192.168.10.10
    |
    | forwarded query
    v
Upstream Recursive Resolver
    |
    v
DNS hierarchy
```

The internal DNS server can therefore:

```
1. Receive the client query
2. Check its own information and cache
3. Forward unresolved requests upstream
4. Receive the answer
5. Return the answer to the client
```

The upstream resolver becomes responsible for obtaining the answer.

This produces a flow such as:

```
Client
  |
  v
Internal DNS Server
  |
  v
Forwarder
  |
  v
Root / TLD / Authoritative DNS
  |
  v
Forwarder
  |
  v
Internal DNS Server
  |
  v
Client
```

Forwarders, conditional forwarding, and related DNS designs are covered in detail in a dedicated DNS forwarding topic.

---

## Successful Response vs Referral

It is important to distinguish two types of useful DNS responses.

### Final Answer

A final answer contains the requested DNS information.

For example:

```
www.example.com
→ 203.0.113.20
```

### Referral

A referral does not contain the final requested answer.

Instead, it tells the resolver where the lookup should continue.

For example:

```
Root:
→ Ask .com

.com:
→ Ask the authoritative servers for example.com
```

This distinction is what allows DNS responsibility to be distributed across many independent servers.

---

## Resolution Failure - NXDOMAIN

A DNS query does not always succeed.

Suppose the client requests:

```
does-not-exist.example.com
```

and the authoritative DNS server confirms that the requested name does not exist.

DNS can return a specific negative response:

```
NXDOMAIN
```

Conceptually:

```
Client
  |
  | does-not-exist.example.com?
  v
Recursive Resolver
  |
  v
Authoritative Server
  |
  | NXDOMAIN
  v
Recursive Resolver
  |
  | name does not exist
  v
Client
```

NXDOMAIN is a valid DNS response.

It means the DNS lookup was processed and the requested domain name was determined not to exist.

This is different from a DNS server simply failing to respond.

---

## Negative Caching

Certain negative DNS answers can also be cached.

For example:

```
First request:

does-not-exist.example.com
→ NXDOMAIN
```

A later request for the same nonexistent name may be answered from the resolver's negative cache instead of repeating the full DNS lookup.

Conceptually:

```
Client
  |
  v
Recursive Resolver
  |
  | negative cache hit
  v
NXDOMAIN
```

Negative caching prevents repeated queries for names that have already been determined not to exist.

The detailed timing and control of negative caching are covered later in the caching and TTL topic.

---

## Timeout Is Different from NXDOMAIN

These situations should not be confused:

```
NXDOMAIN
```

means:

```
The DNS infrastructure responded
and determined that the requested
name does not exist.
```

while:

```
Timeout
```

means:

```
A usable DNS response was not received.
```

Examples of timeout-related causes can include:

- unreachable DNS server,
- DNS service unavailable,
- firewall blocking DNS communication,
- routing problem,
- packet loss,
- server overload,
- other communication failures.

This distinction is important during troubleshooting because:

```
NXDOMAIN
→ DNS responded with a negative result

Timeout
→ communication or DNS service response failed
```

---

## Multiple Configured DNS Servers and Failure

A client may have multiple DNS servers configured.

For example:

```
192.168.10.10
192.168.10.11
```

If one server does not respond, the operating system may:

- wait for a timeout,
- retry the query,
- try another configured DNS server,
- adjust which server it prefers.

A simplified example is:

```
Client
  |
  | query
  v
192.168.10.10
  |
  X no response

Client
  |
  | retry / select another server
  v
192.168.10.11
  |
  ✓ response
```

The exact behavior depends on the operating system and resolver implementation.

It should therefore not be assumed that every client follows a universal strict "primary DNS, then secondary DNS" process.

Multiple DNS server configuration and redundancy are introduced in [DNS Fundamentals](../01-dns-fundamentals/dns-fundamentals.md).

---

## Why Resolution Usually Looks Faster Than the Full Diagram

The complete resolution process may appear long:

```
Client
→ Recursive Resolver
→ Root
→ TLD
→ Authoritative Server
→ Recursive Resolver
→ Client
```

but this full path does not need to occur for every DNS request.

Caching can allow several steps to be skipped.

For example, the recursive resolver may already know:

```
Where .com servers are
```

or:

```
Which name servers are authoritative
for example.com
```

or even:

```
www.example.com
→ 203.0.113.20
```

The lookup can then begin from the most useful information the resolver already has.

For example:

```
Full lookup:

Resolver
→ Root
→ .com
→ example.com
→ answer
```

may later become:

```
Cached delegation available:

Resolver
→ example.com authoritative server
→ answer
```

or simply:

```
Cached final answer available:

Resolver
→ answer
```

This is one of the reasons DNS caching is essential to the scalability and performance of DNS.

---

## Resolution Process Summary

Using the original environment:

```
Workstation:
192.168.10.25

DNS Server:
192.168.10.10

Requested Name:
www.example.com
```

the resolution process can be summarized as:

```
1. The application requests resolution of www.example.com.

2. The local resolver checks available local information.

3. If no local answer exists, it selects a configured DNS server.

4. The workstation sends a DNS query to 192.168.10.10.

5. The recursive resolver checks its cache.

6. If the resolver has a valid cached answer, it returns it.

7. If not, it begins resolution.

8. The resolver queries the DNS root.

9. The root returns a referral to .com.

10. The resolver queries a .com TLD server.

11. The TLD server returns a referral to the authoritative
    servers for example.com.

12. The resolver obtains enough information to contact
    an authoritative server.

13. The resolver queries the authoritative server.

14. The authoritative server returns the requested DNS
    information or an appropriate DNS response.

15. The recursive resolver may cache the response.

16. The resolver returns the response to the workstation.

17. The workstation may cache the response locally.

18. The local resolver returns the result to the application.

19. The application can then begin normal network
    communication using the resolved information.
```

---

## Key Takeaways

- DNS resolution begins when an application asks the operating system to resolve a DNS name.
- The local resolver may satisfy the request from local information before sending a DNS query.
- If no local answer exists, the resolver sends the query to a configured DNS server.
- A recursive resolver attempts to provide the client with a complete answer.
- The recursive resolver first checks its own cache before performing additional queries.
- Recursive requests and iterative lookups describe different responsibilities during DNS resolution.
- A recursive resolver commonly follows referrals through the DNS hierarchy.
- The root server is authoritative for the root zone and refers the resolver toward the appropriate TLD.
- A TLD server can refer the resolver toward the authoritative DNS servers for a domain.
- NS information identifies authoritative name servers.
- Glue information can provide address information needed to reach referred name servers.
- An authoritative DNS server is the source of authoritative DNS information for the zone it serves.
- A referral tells the resolver where to continue; it is not the same as a final answer.
- DNS aliases can require the resolver to continue resolving another name before reaching the final address.
- Successful answers can be cached by both recursive resolvers and client resolvers.
- Cached entries remain usable only while their DNS lifetime is valid.
- DNS forwarding allows one DNS server to pass unresolved queries to another resolver.
- NXDOMAIN is a valid negative DNS response indicating that the requested name does not exist.
- Certain negative DNS responses can also be cached.
- A timeout is different from NXDOMAIN because it means that a usable DNS response was not received.
- When multiple DNS servers are configured, failover and retry behavior depends on the client operating system and resolver implementation.
- The complete root-to-authoritative lookup does not happen for every request because caching can allow many steps to be skipped.
- DNS resolution discovers the required DNS information; normal IP communication begins afterward.