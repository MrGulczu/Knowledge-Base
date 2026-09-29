# DHCP Troubleshooting

## Overview

DHCP troubleshooting should follow a structured process rather than immediately changing DHCP server settings.

A useful starting point is to determine:

```
What exactly is failing?
Is one client affected?
Is an entire VLAN affected?
Did the request reach the DHCP server?
Did the server generate a response?
Did the response return to the client?
```

Follow the DHCP communication path:

```
Client
   |
   v
Switch / Wi-Fi
   |
   v
VLAN
   |
   v
Gateway / DHCP Relay
   |
   v
Firewall / ACL
   |
   v
DHCP Server
```

Then follow the response back toward the client.

---

## APIPA Address

On Windows, an address from `169.254.0.0/16` is an IPv4 link-local address commonly associated with **APIPA - Automatic Private IP Addressing**.

For example:

```
IPv4 Address:   169.254.83.21
Subnet Mask:    255.255.0.0
Default Gateway: None
```

When a client configured for automatic IPv4 addressing uses APIPA, it strongly indicates that the client did not successfully obtain a normal DHCP lease.

It does not prove exactly where DHCP failed.

```
Client
   |
   | DHCPDISCOVER
   v
Switch / VLAN
   |
   v
DHCP Relay
   |
   v
Firewall / ACL
   |
   v
DHCP Server
   |
   | DHCPOFFER
   v
Return Path
```

Possible causes include:

```
Wrong VLAN
DHCP traffic blocked
Missing or incorrect DHCP relay
Routing problem
Firewall or ACL problem
DHCP server unavailable
Missing or disabled scope
Pool exhaustion
Failure on the return path
```

---

## Determine the Scope of the Problem

Before investigating shared infrastructure, check another client in the same VLAN.

```
Client A:
169.254.x.x

Client B:
10.0.20.105
Valid DHCP lease
```

If another client in the same VLAN works, the shared DHCP infrastructure is probably functioning.

A useful rule is:

```
One client affected
        |
        v
Start local

Many clients in same VLAN affected
        |
        v
Start with shared infrastructure
```

### One Client Affected

Check:

```
Is the interface configured for DHCP?
Is the correct interface being used?
Is the physical or wireless link operational?
Is the switch port in the correct VLAN?
Is the client connected to the expected SSID?
Is the DHCP client service working?
Is local firewall/security software interfering?
Does the client actually send DHCP messages?
```

### Multiple Clients Affected

If all tested clients in one VLAN fail while other VLANs work, investigate the shared path for that VLAN.

---

## One VLAN Fails While Others Work

Suppose:

```
VLAN 20 - Employees
DHCP works

VLAN 30 - Printers
DHCP works

VLAN 40 - Wi-Fi
DHCP fails
```

First confirm that multiple clients in VLAN 40 are actually affected.

Then check:

```
VLAN 40 exists and is active?
        |
        v
Clients assigned to VLAN 40?
        |
        v
VLAN carried across required trunks?
        |
        v
VLAN 40 gateway/interface operational?
        |
        v
DHCP relay configured for VLAN 40?
        |
        v
Relay points to correct DHCP server?
        |
        v
Firewall / ACL permits DHCP?
        |
        v
DHCP scope for VLAN 40 exists?
        |
        v
Scope enabled?
        |
        v
Addresses available?
```

Because other VLANs use the same DHCP server successfully, a complete server outage is less likely.

---

## Client Has an Address but No Connectivity

Receiving a DHCP lease does not guarantee that all network configuration is correct.

Example:

```
IP address:  10.0.20.105
Subnet:      255.255.255.0
Gateway:     10.0.20.1
DNS:         10.0.10.10
```

Test connectivity progressively:

```
1. Verify the client's configuration
2. Test the default gateway
3. Test a remote internal IP
4. Test a known external IP
5. Test DNS name resolution
```

Conceptually:

```
Client
   |
   v
Default Gateway
   |
   v
Remote Internal Network
   |
   v
External Network
   |
   v
DNS Name Resolution
```

If a known external IP is reachable but a hostname cannot be resolved, investigate DNS.

Useful checks include:

```
Which DNS server did DHCP provide?
Can the client reach the DNS server?
Can the DNS server resolve the name?
Does the expected DNS record exist?
Is the expected DNS suffix/domain configured?
```

See [DHCP and DNS](../10-dhcp-and-dns/dhcp-and-dns.md).

### Ping Is Not Absolute Proof

Ping is useful, but ICMP can be blocked independently of other communication.

A successful ping is useful evidence of connectivity. A failed ping does not always prove that routing or the destination is unavailable.

---

## Incorrect DHCP Options

A client may receive a valid address but incorrect options.

Example:

```
IP address:  10.0.20.105
Subnet:      255.255.255.0
Gateway:     10.0.30.1
DNS:         10.0.10.10
```

The client belongs to:

```
10.0.20.0/24
```

but the gateway is:

```
10.0.30.1
```

The gateway is outside the client's local `/24` network. The client cannot normally reach it directly, so communication outside the local subnet fails.

DNS may also appear broken because the DNS server is on another network and requires working routing.

Troubleshoot the source of the option:

```
Wrong DHCP option received
        |
        v
Check applicable DHCP configuration
        |
     +--+--+
     |     |
 Incorrect Correct
     |     |
     v     v
Fix DHCP  Investigate client
config    lease/configuration
```

Depending on the DHCP implementation, options can be configured at different levels:

```
Server
Scope
Policy
Reservation
Client-specific configuration
```

Check which configuration actually applies to the client.

See [DHCP Options](../06-dhcp-options/dhcp-options.md).

---

## DHCP Pool Exhaustion

A scope can be enabled and reachable but unable to provide new leases.

```
Scope:
10.0.20.50 - 10.0.20.200

Status:
Enabled

Available addresses:
0
```

Existing clients with valid leases may continue operating while new clients fail to obtain addresses.

Before increasing the pool, check:

```
How many active leases exist?
Is that number expected?
Can the pool safely be extended?
```

If the number of leases matches the real number of devices, the pool may simply be too small.

Before extending it, check addresses outside the current pool for:

```
Default gateway
Servers
Printers
Network equipment
Static assignments
Reservations
Exclusions
Other DHCP pools
```

For example:

```
Subnet:
10.0.20.0/24

Current pool:
10.0.20.50 - 10.0.20.200
```

Do not assume unused-looking addresses are actually free.

If the number of leases is unexpectedly high, investigate temporary devices, lease duration, unexpected clients, stale/unexpected leases, and suspicious DHCP activity.

See [DHCP Scopes and Pools](../05-scopes-and-pools/scopes-and-pools.md) and [DHCP Security](../12-dhcp-security/dhcp-security.md).

---

## Reservation Not Working

Suppose:

```
Reservation:

MAC / Client Identifier:
AA:BB:CC:11:22:33

Reserved IP:
10.0.20.75
```

but the client receives:

```
10.0.20.106
```

First verify the identity presented by the client.

If the requesting interface identifies as:

```
AA:BB:CC:44:55:66
```

the reservation may not match.

Possible reasons include:

```
NIC replacement
Ethernet vs Wi-Fi
USB network adapter
Docking station
MAC randomization
Incorrectly entered identifier
```

A useful sequence is:

```
Correct client identifier/MAC?
        |
        v
Reservation in correct scope?
        |
        v
Reserved address valid and available?
        |
        v
Existing lease still active?
        |
        v
Release/renew as appropriate
and verify assignment
```

Some DHCP implementations use a client identifier rather than only the hardware MAC address.

See [DHCP Reservations and Exclusions](../07-reservations-and-exclusions/reservations-and-exclusions.md).

---

## Remote VLAN DHCP Failure

Consider:

```
DHCP Server Network
10.0.10.0/24
      |
      | DHCP works locally
      v
Local Clients


Remote VLAN
10.0.20.0/24
      |
      X
DHCP fails
```

Suppose the DHCP server is operational, the correct scope exists and is enabled, and addresses are available.

The remote VLAN depends on DHCP relay, so investigate the relay path.

Check:

```
Is DHCP relay enabled?
Is it configured on the correct interface/VLAN?
Does it point to the correct DHCP server?
Can the relay reach the DHCP server?
Is routing correct?
Does the firewall/ACL permit DHCP communication?
```

See [DHCP Relay](../08-dhcp-relay/dhcp-relay.md).

---

## Did the Server Receive the Request?

This is one of the most useful troubleshooting boundaries.

```
Does DHCP server receive the request?
        |
     +--+--+
     |     |
    No    Yes
     |     |
     v     v
Check      Does server
client-    generate
to-server  response?
path
```

If the request does not reach the server, investigate:

```
Client
Switch / Wi-Fi
VLAN
Relay
Routing
Firewall / ACL
```

If it reaches the server, continue with server-side processing.

---

## Request Arrives but No Offer Is Generated

Suppose a capture or server log confirms that the DHCP request reaches the server, but no `DHCPOFFER` is generated.

The client-to-server path has already been proven.

Check:

```
Does a scope exist for the client's subnet?
Is the scope enabled?
Are addresses available?
Does the server recognize the client's network?
Are exclusions preventing assignment?
Are policies or filters preventing an offer?
Is the DHCP service operating correctly?
What do the server logs show?
```

A missing scope for the relayed subnet is one possible cause.

---

## Offer Generated but Relay Does Not Receive It

If the server generates a response but the relay never receives it:

```
DHCP Server
    |
    | DHCPOFFER generated
    v
    X
Routing / Firewall / ACL
    |
    v
DHCP Relay
```

Investigate:

```
Return routing
Firewall policy
ACLs
Local server firewall
Relay reachability
Network path
```

---

## Relay Receives Response but Client Does Not

If the response reaches the relay but never reaches the client:

```
DHCP Server
      |
      v
Relay
      |
      | Response visible
      v
Switching / VLAN
      |
      X
Client
```

Investigate:

```
Relay behavior
VLAN configuration
Trunk configuration
Switching path
Wireless infrastructure
DHCP snooping/security policies
Client-side filtering
```

---

## Using DHCP Server Logs

DHCP server logs can help determine:

```
Did the server receive the request?
Which client requested an address?
Which scope was selected?
Was an address offered?
Was the request rejected?
Was the pool exhausted?
Was a reservation matched?
Was a lease created or renewed?
```

If the user reports a DHCP failure but the server logs show no request from that client, the problem is likely somewhere between the client and server.

```
Client
   |
   | DHCP request
   v
Switch / Wi-Fi
   |
   v
VLAN
   |
   v
Relay
   |
   v
Firewall / ACL
   |
   X
DHCP Server

No request received
```

Changing the DHCP pool would not be the logical first step.

---

## Packet Captures

Packet captures can identify exactly where the exchange stops.

Useful capture points include:

```
Client
Access network
Gateway / Relay
DHCP server
```

Compare what each point sees:

```
Client sends DHCPDISCOVER
        |
        v
Relay sees request
        |
        v
Server sees relayed request
        |
        v
Server sends DHCPOFFER
        |
        v
Relay receives response
        |
        v
Client receives response
```

If one step is missing, the failure boundary becomes much clearer.

---

## Follow the DORA Process

The normal DHCPv4 exchange provides a useful troubleshooting model:

```
DISCOVER
   |
   v
OFFER
   |
   v
REQUEST
   |
   v
ACK
```

Ask which message was the last one successfully observed.

For example:

```
DISCOVER visible
OFFER missing
```

points in a different direction from:

```
DISCOVER visible
OFFER visible
REQUEST visible
ACK missing
```

See [DHCP DORA Process](../03-dora-process/dora-process.md).

---

## DHCP and DNS Troubleshooting

A client can have a valid DHCP lease while DNS resolution is broken.

If:

```
Remote IP communication works
```

but:

```
Hostname resolution fails
```

investigate:

```
Did DHCP provide the expected DNS server?
Can the client reach that DNS server?
Does the DNS record exist?
Does the record contain the correct address?
If dynamic DNS is used, is DHCP/DNS updating correctly?
```

Do not treat a DNS failure as proof that DHCP address allocation failed.

---

## DHCP Security Features During Troubleshooting

Security controls can also interrupt DHCP.

If problems begin after switch security changes, check:

```
Is DHCP snooping enabled on the correct VLAN?
Are trusted interfaces configured correctly?
Is legitimate server traffic entering through an untrusted port?
Is DHCP rate limiting being triggered?
Has an interface entered a restricted/error-disabled state?
```

See [DHCP Security](../12-dhcp-security/dhcp-security.md).

---

## Practical Decision Tree

```
Client has valid DHCP lease?
        |
     +--+--+
     |     |
    No    Yes
     |     |
     |     v
     |   Correct DHCP configuration?
     |       |
     |    +--+--+
     |    |     |
     |   No    Yes
     |    |     |
     |    v     v
     |  Check   Test gateway
     |  options Test remote IP
     |          Test DNS
     |
     v
Other clients in same VLAN work?
        |
     +--+--+
     |     |
    Yes    No
     |     |
     v     v
Check      Check shared:
client     VLAN
port       Relay
VLAN       Firewall / ACL
           Scope
           Pool
           Server
```

---

## Packet-Flow Troubleshooting Model

A useful checklist is:

```
1. Did the client send a request?

2. Did the switch/VLAN carry it?

3. Did the relay receive it?

4. Did the server receive it?

5. Did the server generate a response?

6. Did the relay receive the response?

7. Did the response reach the client?

8. Did the client accept and apply the configuration?
```

Each proven step removes part of the path from the investigation.

---

## Example - Single Client with APIPA

```
PC-01 -> 169.254.20.15
PC-02 -> 10.0.20.105
PC-03 -> 10.0.20.106
```

Because other clients in the same VLAN work, focus on:

```
PC-01 DHCP configuration
PC-01 network interface
Switch port
VLAN assignment
Local security software
Whether PC-01 sends DHCP traffic
```

---

## Example - Entire VLAN with APIPA

```
VLAN 40:

PC-01 -> APIPA
PC-02 -> APIPA
PC-03 -> APIPA
```

while VLAN 20 and VLAN 30 work.

Focus on:

```
VLAN 40
   |
   v
Gateway
   |
   v
DHCP Relay
   |
   v
Firewall / ACL
   |
   v
VLAN 40 DHCP Scope
```

---

## Example - Valid Address but DNS Failure

```
Client:
10.0.20.105/24

Gateway:
10.0.20.1

DNS:
10.0.10.10
```

If the gateway and remote IP addresses are reachable but hostname resolution fails, focus on DNS configuration, reachability, records, and any dynamic DNS update process.

---

## Example - Pool Exhausted

```
DHCP Scope:
10.0.20.50 - 10.0.20.200

Available:
0
```

Check:

```
Active lease count
Expected client count
Lease duration
Unexpected clients
Static usage outside the pool
Whether the pool can safely be extended
Possible abnormal DHCP activity
```

Do not extend the pool until the rest of the subnet usage is understood.

---

## Troubleshooting Principle

A useful question throughout DHCP troubleshooting is:

```
What is the last point where I can prove
that the expected DHCP message exists?
```

For example:

```
No request in server logs
        |
        v
Problem is before the server


Request reaches server
but no offer is generated
        |
        v
Investigate server / scope / pool


Offer leaves server
but relay never receives it
        |
        v
Investigate return path


Offer reaches relay
but not client
        |
        v
Investigate relay / VLAN /
switching / access network
```

This approach reduces guesswork and avoids changing unrelated components.

---

## Key Takeaways

- Troubleshoot DHCP systematically instead of immediately changing server configuration.
- APIPA indicates that a Windows client configured for automatic IPv4 addressing did not successfully obtain a normal DHCP lease.
- APIPA does not prove where the DHCP exchange failed.
- Determine whether one client or multiple clients are affected first.
- If other clients in the same VLAN work, begin with the affected client, switch port, and VLAN assignment.
- If an entire VLAN fails while other VLANs work, investigate that VLAN's shared path, relay, firewall/ACL, scope, and pool.
- Receiving an IP address does not guarantee that all DHCP options are correct.
- Test connectivity progressively from the local gateway toward remote networks and DNS.
- A failed ping does not always prove failed connectivity because ICMP may be filtered.
- Trace incorrect gateway or DNS options back to the DHCP configuration that actually applies to the client.
- Pool exhaustion can prevent new clients from obtaining leases while existing clients continue operating.
- Before extending a pool, verify lease usage and addresses already used elsewhere in the subnet.
- Unexpectedly high lease usage should be investigated rather than automatically solved by enlarging the pool.
- When a reservation fails, verify the client identifier or MAC address first.
- Adapter changes, docking stations, Wi-Fi, and MAC randomization can affect reservation matching.
- If local DHCP works but a remote VLAN fails, investigate the relay and routed path.
- If the DHCP server never receives the request, investigate the client-to-server path.
- If the server receives the request but generates no offer, investigate the scope, pool, policies, exclusions, and DHCP service.
- If the server generates an offer but it does not reach the relay, investigate routing, firewall/ACL rules, and the return path.
- If the relay receives the response but the client does not, investigate relay behavior, VLANs, switching, wireless infrastructure, and security controls.
- DHCP server logs and packet captures help determine where the DHCP exchange stops.
- Following DISCOVER, OFFER, REQUEST, and ACK provides a practical DHCPv4 troubleshooting framework.
- Valid IP connectivity combined with failed hostname resolution points toward DNS rather than basic DHCP address allocation.
- Troubleshooting should be based on evidence about where communication stops rather than assumptions about which component is broken.