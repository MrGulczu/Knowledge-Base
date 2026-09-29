# DHCP Options Reference

## Overview

This file is a quick-reference catalog of commonly encountered DHCPv4 and BOOTP option codes.

For an explanation of how DHCP options are used, inherited, requested, and delivered to clients, see [DHCP Options](../dhcp-options.md).

The table includes common client configuration options, DHCP protocol-control options, legacy options, specialized options, and selected modern extensions. Some options are rarely used today but can still appear in documentation, packet captures, older environments, or vendor configurations.

> **Source of truth:** IANA - Dynamic Host Configuration Protocol (DHCP) and Bootstrap Protocol (BOOTP) Parameters.

> This reference should be periodically compared with the IANA registry because option assignments and statuses can change over time.

---

## Commonly Encountered DHCP Options

|Option|Name|Category|Description / Typical Use|
|---|---|---|---|
|1|Subnet Mask|Client Configuration|Provides the IPv4 subnet mask used by the client.|
|3|Router|Client Configuration|Provides one or more router addresses. Commonly used to configure the default gateway.|
|6|Domain Server|Client Configuration|Provides one or more DNS server IPv4 addresses.|
|12|Hostname|Client Identification|Provides or communicates the client host name.|
|15|Domain Name|Client Configuration|Provides the DNS domain name associated with the client.|
|28|Broadcast Address|Client Configuration|Provides the broadcast address for the client's subnet.|
|42|NTP Servers|Client Configuration|Provides one or more NTP server IPv4 addresses.|
|43|Vendor Specific|Vendor-Specific|Carries vendor-specific configuration information. Its internal format depends on the vendor or implementation.|
|50|Address Request|DHCP Protocol|Communicates the IPv4 address requested by the client.|
|51|Address Time|DHCP Protocol|Specifies the IPv4 address lease lifetime.|
|53|DHCP Message Type|DHCP Protocol|Identifies the DHCP message type, such as Discover, Offer, Request, ACK, NAK, or Release.|
|54|DHCP Server Identifier|DHCP Protocol|Identifies the DHCP server participating in the exchange.|
|55|Parameter Request List|DHCP Protocol|Allows the client to list the DHCP options it would like the server to provide.|
|57|DHCP Maximum Message Size|DHCP Protocol|Communicates the maximum DHCP message size the client is prepared to receive.|
|58|Renewal Time|DHCP Protocol|Provides the T1 lease renewal timer.|
|59|Rebinding Time|DHCP Protocol|Provides the T2 lease rebinding timer.|
|60|Class Identifier|Client Classification|Communicates a vendor or implementation class identifier. Often called the Vendor Class Identifier.|
|61|Client Identifier|Client Identification|Provides an identifier used to distinguish the DHCP client.|
|66|TFTP Server Name|Boot / Deployment|Provides the name of a TFTP server. Commonly encountered in network boot and provisioning environments.|
|67|Bootfile Name|Boot / Deployment|Provides the name of a boot file. Commonly encountered in PXE and other network boot environments.|
|82|Relay Agent Information|DHCP Relay|Allows a DHCP relay to insert information about the client-facing circuit or relay environment.|
|93|Client System Architecture|Boot / Deployment|Identifies the client system architecture. Commonly useful during PXE boot selection.|
|97|UUID/GUID|Client Identification|Carries a UUID/GUID-based client identifier, commonly encountered in PXE environments.|
|108|IPv6-Only Preferred|Transition / Modern Networks|Indicates how long the client should disable DHCPv4 in an IPv6-only preferred environment.|
|114|DHCP Captive-Portal|Client Configuration|Provides information used to discover a captive portal.|
|119|Domain Search|Client Configuration|Provides a DNS domain search list.|
|121|Classless Static Route|Routing|Provides classless IPv4 static routes to the client.|
|162|OPTION_V4_DNR|DNS / Modern Networks|Provides encrypted DNS resolver information to DHCPv4 clients.|
|255|End|DHCP Protocol|Marks the end of the DHCP option field.|

---

## Core Client Configuration Options

These options directly affect normal IPv4 client configuration and are among the most useful options for systems and network administrators.

|Option|Name|Purpose|
|---|---|---|
|1|Subnet Mask|Defines the client's IPv4 subnet mask.|
|3|Router|Provides router addresses, commonly including the default gateway.|
|6|Domain Server|Provides DNS resolver addresses.|
|12|Hostname|Communicates host name information.|
|15|Domain Name|Provides the DNS domain name associated with the client.|
|28|Broadcast Address|Provides the subnet broadcast address.|
|42|NTP Servers|Provides NTP server addresses.|
|119|Domain Search|Provides a list of DNS suffixes used for name searches.|
|121|Classless Static Route|Provides additional IPv4 routes to the client.|

For the underlying concepts of gateways and subnet configuration, see [IP Addressing](../../../01-fundamentals/04-ip-addressing/ip-addressing.md). For DNS concepts, see [DNS Fundamentals](../../../02-dns/01-dns-fundamentals/dns-fundamentals.md).

---

## DHCP Lease and Protocol Options

Several DHCP options exist primarily to control the DHCP process itself rather than configure applications on the client.

|Option|Name|Purpose|
|---|---|---|
|50|Address Request|Identifies the IPv4 address the client is requesting.|
|51|Address Time|Defines the lease lifetime.|
|52|Overload|Indicates that additional DHCP options are stored in the BOOTP `file` or `sname` fields.|
|53|DHCP Message Type|Identifies the type of DHCP message.|
|54|DHCP Server Identifier|Identifies the DHCP server.|
|55|Parameter Request List|Lists options requested by the client.|
|56|DHCP Message|Carries an error or informational message from the DHCP server.|
|57|DHCP Maximum Message Size|Communicates the largest DHCP message the client can accept.|
|58|Renewal Time|Defines the T1 renewal timer.|
|59|Rebinding Time|Defines the T2 rebinding timer.|
|60|Class Identifier|Identifies a client vendor or implementation class.|
|61|Client Identifier|Provides an identifier for the DHCP client.|

The DORA exchange and DHCP message types are covered in [DHCP DORA Process](../../03-dora-process/dora-process.md). Lease time, T1, and T2 are covered in [DHCP Leases](../../04-dhcp-leases/dhcp-leases.md).

---

## DHCP Message Type Values - Option 53

Option 53 identifies the DHCP message carried by the packet.

|Value|Message Type|Purpose|
|---|---|---|
|1|DHCPDISCOVER|Client searches for available DHCP servers.|
|2|DHCPOFFER|Server offers configuration to a client.|
|3|DHCPREQUEST|Client requests or renews DHCP configuration.|
|4|DHCPDECLINE|Client reports that an offered address appears unusable, for example because it is already in use.|
|5|DHCPACK|Server acknowledges and grants the requested configuration.|
|6|DHCPNAK|Server rejects the client's requested configuration.|
|7|DHCPRELEASE|Client voluntarily releases its lease.|
|8|DHCPINFORM|Client requests additional configuration without requesting a new IPv4 address.|
|9|DHCPFORCERENEW|Server requests that a client begin lease renewal.|
|10|DHCPLEASEQUERY|Queries lease information.|
|11|DHCPLEASEUNASSIGNED|Indicates that a queried lease is unassigned.|
|12|DHCPLEASEUNKNOWN|Indicates that lease information is unknown.|
|13|DHCPLEASEACTIVE|Indicates that the queried lease is active.|
|14|DHCPBULKLEASEQUERY|Requests lease information in bulk.|
|15|DHCPLEASEQUERYDONE|Indicates completion of a bulk lease query.|
|16|DHCPACTIVELEASEQUERY|Supports continuous updates about active leases.|
|17|DHCPLEASEQUERYSTATUS|Communicates status information for an active lease query.|
|18|DHCPTLS|Identifies DHCP lease query communication using TLS.|

For the normal client address allocation sequence, see [DHCP DORA Process](../../03-dora-process/dora-process.md).

---

## Boot and PXE-Related Options

DHCP is commonly involved in network boot and operating-system deployment environments.

|Option|Name|Purpose|
|---|---|---|
|13|Boot File Size|Provides the boot file size in 512-byte blocks.|
|43|Vendor Specific|Can carry vendor-specific boot or provisioning information.|
|60|Class Identifier|Can identify a client as belonging to a particular vendor or boot class.|
|66|TFTP Server Name|Provides the TFTP server name.|
|67|Bootfile Name|Provides the boot file name.|
|93|Client System Architecture|Identifies the client's architecture, such as an architecture used for network boot selection.|
|94|Client Network Device Interface|Identifies the client's network device interface information.|
|97|UUID/GUID|Provides a UUID/GUID-based identifier.|
|128-135|Vendor / PXE-related use|These values have historical and vendor-specific PXE-related uses. Their meaning is not universal and must be interpreted according to the implementation.|
|208|PXELINUX Magic|Historical PXELINUX-specific option. Deprecated.|
|209|Configuration File|Provides a configuration file name in PXELINUX environments.|
|210|Path Prefix|Provides a path prefix in PXELINUX environments.|
|211|Reboot Time|Provides a reboot time value in PXELINUX environments.|

PXE should not be treated as a single universal DHCP option. Actual deployment behavior depends on the PXE implementation, DHCP server, boot server, firmware architecture, and whether services such as proxyDHCP are used.

---

## Routing and Network Behavior Options

DHCP can provide clients with network behavior and routing parameters in addition to their basic address configuration.

|Option|Name|Purpose|
|---|---|---|
|19|Forward On/Off|Controls IPv4 forwarding behavior.|
|20|Source Route On/Off|Controls acceptance of source-routed packets.|
|21|Policy Filter|Provides routing policy filters.|
|23|Default IP TTL|Provides a default IPv4 TTL value.|
|26|Interface MTU|Provides the interface MTU.|
|28|Broadcast Address|Provides the subnet broadcast address.|
|31|Router Discovery|Controls router discovery behavior.|
|32|Router Request|Provides the router solicitation address.|
|33|Static Route|Provides classful static routes.|
|35|ARP Timeout|Provides the ARP cache timeout.|
|121|Classless Static Route|Provides classless IPv4 static routes and is preferred for modern classless routing information.|

Some of these options are legacy or uncommon in modern endpoint deployments but can still appear in protocol references and packet captures.

---

## Name, Directory, and Time Service Options

DHCP includes options for several network services, including some older technologies.

|Option|Name|Purpose|
|---|---|---|
|2|Time Offset|Provides the time offset from UTC. Deprecated in favor of modern timezone options.|
|4|Time Server|Provides legacy RFC 868 time server addresses. This is not the same as NTP Option 42.|
|5|Name Server|Provides IEN-116 name server addresses. This is not the normal DNS server option.|
|6|Domain Server|Provides DNS server addresses.|
|7|Log Server|Provides logging server addresses.|
|9|LPR Server|Provides Line Printer Remote server addresses.|
|12|Hostname|Provides host name information.|
|15|Domain Name|Provides the client's DNS domain name.|
|40|NIS Domain|Provides an NIS domain name.|
|41|NIS Servers|Provides NIS server addresses.|
|42|NTP Servers|Provides NTP server addresses.|
|44|NETBIOS Name Server|Provides NetBIOS name server addresses, commonly associated with WINS.|
|45|NETBIOS Datagram Distribution Server|Provides NetBIOS datagram distribution server addresses.|
|46|NETBIOS Node Type|Provides the NetBIOS node type.|
|47|NETBIOS Scope|Provides the NetBIOS scope.|
|95|LDAP|Provides Lightweight Directory Access Protocol information as defined for this option.|
|100|PCode|Provides an IEEE 1003.1 timezone string.|
|101|TCode|Provides a reference to a timezone database entry.|
|119|Domain Search|Provides the DNS domain search list.|

Option 4 and Option 42 should not be confused. Option 4 refers to the older Time Protocol, while Option 42 provides NTP servers.

---

## DHCP Relay Agent Information - Option 82

**Option 82** is the Relay Agent Information option.

A DHCP relay can use it to provide additional information about where a client request entered the network.

Common Option 82 sub-options include:

|Sub-option|Name|Purpose|
|---|---|---|
|1|Agent Circuit ID|Identifies the client-facing circuit or attachment point.|
|2|Agent Remote ID|Provides an identifier associated with the remote client or access device.|
|5|Link Selection|Helps identify the link or subnet used for address selection.|
|6|Subscriber-ID|Identifies a subscriber.|
|9|Vendor-Specific Information|Carries relay vendor-specific information.|
|12|Relay Agent Identifier|Identifies the relay agent.|
|16|Access-Point-BSSID|Can identify the wireless access point BSSID.|
|19|DHCPv4 Relay Source Port|Communicates a DHCPv4 relay source port.|

Option 82 is especially useful in larger access networks where the DHCP infrastructure needs more information than simply the relay's basic network address.

The general role of DHCP relay was introduced in [DHCP Components](../../02-dhcp-components/dhcp-components.md).

---

## Modern and Specialized Options

Some DHCP options support more specialized or newer network functionality.

|Option|Name|Purpose|
|---|---|---|
|90|Authentication|Provides DHCP authentication information.|
|108|IPv6-Only Preferred|Allows an IPv6-capable network to signal that DHCPv4 should be disabled for a specified period.|
|114|DHCP Captive-Portal|Provides captive portal discovery information.|
|116|Auto-Config|Controls DHCP auto-configuration behavior.|
|118|Subnet Selection Option|Allows selection of the subnet from which configuration should be allocated.|
|120|SIP Servers DHCP Option|Provides SIP server information.|
|121|Classless Static Route|Provides classless static routes.|
|124|Vendor-Identifying Vendor Class|Provides vendor-identifying class information.|
|125|Vendor-Identifying Vendor-Specific Information|Provides vendor-specific information associated with an enterprise identifier.|
|162|OPTION_V4_DNR|Provides encrypted DNS resolver information.|
|212|OPTION_6RD|Provides information for IPv6 rapid deployment over IPv4.|
|220|Subnet Allocation Option|Supports dynamic allocation of IPv4 subnets.|
|221|Virtual Subnet Selection|Supports selection of virtual subnet or VPN-related address spaces.|

These options are not required in every DHCP environment. Their relevance depends on the technologies deployed in the network.

---

## Legacy and Less Common Options

The DHCP/BOOTP registry also contains options created for older or specialized technologies.

Examples include:

|Option|Name|Notes|
|---|---|---|
|8|Quotes Server|Historical Quote of the Day server information.|
|10|Impress Server|Historical Impress network image server information.|
|11|RLP Server|Resource Location Protocol server information.|
|16|Swap Server|Provides a swap server address.|
|17|Root Path|Provides a root disk path, historically useful for diskless systems.|
|18|Extension File|Provides a path to additional BOOTP information.|
|34|Trailers|Controls trailer encapsulation.|
|36|Ethernet|Controls Ethernet encapsulation behavior.|
|37|Default TCP TTL|Provides a default TCP TTL.|
|38|Keepalive Time|Provides a TCP keepalive interval.|
|39|Keepalive Data|Controls TCP keepalive garbage octet behavior.|
|48|X Window Font|Provides X Window font server addresses.|
|49|X Window Manager|Provides X Display Manager addresses.|
|62|NetWare/IP Domain|Provides NetWare/IP domain information.|
|63|NetWare/IP Option|Carries NetWare/IP sub-options.|
|86|NDS Tree Name|Provides a Novell Directory Services tree name.|
|87|NDS Context|Provides a Novell Directory Services context.|

These options are retained in the registry because DHCP and BOOTP have been used across many generations of network technologies.

---

## Reserved and Special Values

Not every DHCP option code is available for normal assignment.

|Value / Range|Status|Notes|
|---|---|---|
|0|Pad|Used for padding within the DHCP options field.|
|96|Removed / Unassigned|Previously allocated and later removed.|
|102-107|Removed / Unassigned|Not currently assigned for normal use.|
|110-111|Removed / Unassigned or Unassigned|Available status depends on the specific value in the current registry.|
|115|Removed / Unassigned|Not currently assigned for normal use.|
|126-127|Removed / Unassigned|Not currently assigned for normal use.|
|163-174|Unassigned|Currently unassigned.|
|178-207|Unassigned|Currently unassigned.|
|214-219|Unassigned|Currently unassigned.|
|222-223|Unassigned|Currently unassigned.|
|224-254|Reserved for Private Use|Intended for private or vendor-specific deployments rather than globally standardized option assignment.|
|255|End|Marks the end of the DHCP option field.|

Values in private-use ranges can have different meanings between vendors and environments and should not be interpreted without knowing the relevant implementation.

---

## Practical Administrator Reference

For normal enterprise DHCP administration, the options most likely to be encountered are:

|Option|Primary Purpose|
|---|---|
|1|Subnet mask|
|3|Default gateway / router|
|6|DNS servers|
|12|Host name|
|15|DNS domain name|
|42|NTP servers|
|43|Vendor-specific information|
|50|Requested IPv4 address|
|51|Lease lifetime|
|53|DHCP message type|
|54|DHCP server identifier|
|55|Parameter Request List|
|58|T1 renewal timer|
|59|T2 rebinding timer|
|60|Vendor / class identifier|
|61|Client identifier|
|66|TFTP server name|
|67|Boot file name|
|82|Relay agent information|
|93|Client system architecture|
|97|Client UUID/GUID|
|119|DNS domain search list|
|121|Classless static routes|

For explanations of how these options are used in DHCP operation, return to [DHCP Options](../dhcp-options.md).

---

## Reference Notes

### Client Configuration

These options provide values that directly configure normal client networking or network services, such as the subnet mask, gateway, DNS servers, domain information, or NTP servers.

### DHCP Protocol

These options are primarily involved in DHCP itself, including message identification, lease timing, server identification, and client requests.

### Vendor-Specific

The meaning or internal structure of these options depends partly or entirely on the vendor or implementation.

### Legacy

The option remains historically relevant or registered but is uncommon in modern enterprise networks.

### Specialized

The option is intended for a narrower protocol, deployment model, or network technology and may never appear in a normal office DHCP environment.

---

## Key Takeaways

- DHCPv4 options are identified by numeric option codes.
- Options can provide normal client configuration, control DHCP protocol behavior, identify clients, support relays, or provide specialized vendor information.
- Options 1, 3, and 6 are among the most important for normal IPv4 client configuration.
- Options 50 through 61 contain several important DHCP protocol and lease-management parameters.
- Option 53 identifies the DHCP message type.
- Option 55 contains the client's Parameter Request List.
- Options 58 and 59 provide the T1 renewal and T2 rebinding timers.
- Options 66 and 67 are commonly encountered in network boot environments, but PXE behavior can involve additional options and services.
- Option 82 carries DHCP relay agent information and can contain multiple relay sub-options.
- Some option codes are legacy, specialized, unassigned, removed, or reserved for private use.
- The IANA DHCP and BOOTP Parameters registry should be treated as the source of truth for current DHCPv4 option assignments.