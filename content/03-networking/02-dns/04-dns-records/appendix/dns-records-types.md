# DNS Record Types Reference

## Overview

This file is a quick-reference catalog of DNS Resource Record (RR) types registered by IANA.

For explanations of the most commonly used DNS records, see [DNS Records](../../04-dns-records/dns-records.md).

The table includes both normal data records and special query/meta types. Older, experimental, deprecated, and obsolete types are retained because they can still appear in documentation, packet captures, historical configurations, or protocol references.

> **Source of truth:** IANA — Domain Name System (DNS) Parameters, Resource Record (RR) TYPEs registry.

> This reference should be periodically compared with the IANA registry because new RR types can be assigned over time.

---

## Assigned DNS Record Types

|Value|Type|Status / Category|Description / Typical Use|
|---|---|---|---|
|0|Reserved|Reserved|Special value; not allocated for ordinary DNS resource records.|
|1|A|Assigned|Maps a DNS name to an IPv4 address.|
|2|NS|Assigned|Identifies an authoritative name server for a DNS zone or delegation.|
|3|MD|Obsolete|Mail destination record. Obsolete; MX should be used instead.|
|4|MF|Obsolete|Mail forwarder record. Obsolete; MX should be used instead.|
|5|CNAME|Assigned|Creates an alias from one DNS name to another canonical DNS name.|
|6|SOA|Assigned|Start of Authority. Stores administrative and synchronization information for a DNS zone.|
|7|MB|Experimental|Identifies a mailbox domain name. Historical experimental mail record.|
|8|MG|Experimental|Identifies a member of a mail group. Historical experimental mail record.|
|9|MR|Experimental|Identifies a renamed mailbox/domain name. Historical experimental mail record.|
|10|NULL|Experimental|Experimental record capable of carrying arbitrary binary data.|
|11|WKS|Assigned / Legacy|Well Known Services record. Describes services available at an address; rarely used today.|
|12|PTR|Assigned|Domain name pointer. Commonly used for reverse DNS lookups.|
|13|HINFO|Assigned|Stores host information such as CPU and operating-system information.|
|14|MINFO|Assigned / Legacy|Stores mailbox or mailing-list information.|
|15|MX|Assigned|Identifies mail exchange servers responsible for receiving email for a domain.|
|16|TXT|Assigned|Stores one or more text strings. Commonly used for verification and policy data such as SPF, DKIM, and DMARC-related information.|
|17|RP|Assigned|Identifies the responsible person for a domain or DNS name.|
|18|AFSDB|Assigned / Legacy|Identifies an AFS database server or DCE/NCA directory server.|
|19|X25|Assigned / Legacy|Stores an X.25 PSDN address.|
|20|ISDN|Assigned / Legacy|Stores an ISDN address.|
|21|RT|Assigned / Legacy|Route Through record. Identifies an intermediate host through which traffic should be routed.|
|22|NSAP|Deprecated|Stores an NSAP address. Deprecated historical OSI networking record.|
|23|NSAP-PTR|Deprecated|Reverse pointer for an NSAP address. Deprecated historical OSI networking record.|
|24|SIG|Assigned / Legacy|Stores a security signature. Still used by some specialized DNS security mechanisms such as SIG(0); DNSSEC normally uses RRSIG.|
|25|KEY|Assigned / Legacy|Stores security key information. Modern DNSSEC normally uses DNSKEY.|
|26|PX|Assigned / Legacy|Stores X.400-to-RFC 822 mail mapping information.|
|27|GPOS|Assigned / Legacy|Stores geographical position data. Historical predecessor to LOC.|
|28|AAAA|Assigned|Maps a DNS name to an IPv6 address.|
|29|LOC|Assigned|Stores geographical location information for a DNS name.|
|30|NXT|Obsolete|Historical DNSSEC Next Domain record. Replaced by newer DNSSEC mechanisms.|
|31|EID|Assigned / Specialized|Endpoint Identifier record associated with the historical Nimrod architecture.|
|32|NIMLOC|Assigned / Specialized|Nimrod Locator record.|
|33|SRV|Assigned|Service discovery record. Identifies service target, port, priority, and weight.|
|34|ATMA|Assigned / Specialized|Stores an ATM network address.|
|35|NAPTR|Assigned|Naming Authority Pointer. Supports rule-based name rewriting and service discovery; used by technologies such as ENUM and SIP.|
|36|KX|Assigned|Key Exchanger record. Identifies a host that provides key-exchange services.|
|37|CERT|Assigned|Stores certificates or certificate-related public-key information in DNS.|
|38|A6|Obsolete|Historical IPv6 address record. Obsolete; AAAA should be used instead.|
|39|DNAME|Assigned|Creates an alias for an entire subtree of the DNS namespace.|
|40|SINK|Experimental / Specialized|Experimental "Kitchen Sink" record intended to carry arbitrary structured data.|
|41|OPT|Meta / Pseudo-RR|EDNS pseudo-resource record used to extend DNS message capabilities. It is not normal zone data.|
|42|APL|Assigned|Address Prefix List. Stores lists of IPv4 or IPv6 address prefixes.|
|43|DS|Assigned / DNSSEC|Delegation Signer. Links a child zone's DNSKEY information into the parent zone for DNSSEC validation.|
|44|SSHFP|Assigned|Stores SSH host-key fingerprints so SSH keys can be verified through DNS.|
|45|IPSECKEY|Assigned|Stores public-key and gateway information used with IPsec.|
|46|RRSIG|Assigned / DNSSEC|Stores DNSSEC signatures covering DNS RRsets.|
|47|NSEC|Assigned / DNSSEC|Proves authenticated denial of existence and links signed DNS names in DNSSEC.|
|48|DNSKEY|Assigned / DNSSEC|Stores public keys used to validate DNSSEC signatures.|
|49|DHCID|Assigned|Stores DHCP client identity information, commonly used with DHCP/DNS update coordination.|
|50|NSEC3|Assigned / DNSSEC|Provides authenticated denial of existence using hashed DNS names.|
|51|NSEC3PARAM|Assigned / DNSSEC|Publishes parameters used by NSEC3.|
|52|TLSA|Assigned / DANE|Associates a TLS service with certificate or public-key information using DANE.|
|53|SMIMEA|Assigned / DANE|Associates S/MIME certificate information with an email identity through DNS.|
|55|HIP|Assigned / Specialized|Stores Host Identity Protocol information.|
|56|NINFO|Assigned / Specialized|Specialized NINFO record type. Limited deployment.|
|57|RKEY|Assigned / Specialized|Specialized RKEY record for key-related information. Limited deployment.|
|58|TALINK|Assigned / Specialized|Trust Anchor Link record. Used for specialized DNSSEC trust-anchor relationships.|
|59|CDS|Assigned / DNSSEC|Child DS. Allows a child zone to signal desired DS record changes to its parent.|
|60|CDNSKEY|Assigned / DNSSEC|Child DNSKEY. Allows a child zone to publish DNSKEY information intended for parent DS synchronization.|
|61|OPENPGPKEY|Assigned|Publishes OpenPGP public-key information in DNS.|
|62|CSYNC|Assigned|Child-to-Parent Synchronization. Allows a child zone to signal delegation-related changes to its parent.|
|63|ZONEMD|Assigned|Stores a cryptographic message digest over DNS zone data to verify zone integrity.|
|64|SVCB|Assigned|General-purpose Service Binding record for discovering service endpoints and connection parameters.|
|65|HTTPS|Assigned|SVCB-compatible record specifically used for HTTP/HTTPS service binding and endpoint discovery.|
|66|DSYNC|Assigned|Discovers endpoints used for DNS delegation synchronization.|
|67|HHIT|Assigned / Specialized|Hierarchical Host Identity Tag record.|
|68|BRID|Assigned / Specialized|UAS Broadcast Remote Identification record. Used with unmanned aircraft system remote identification.|
|69|UNECE|Assigned / Specialized|Carries a value coded according to a UNECE recommendation or external registry.|
|70|ISO|Assigned / Specialized|Carries a value coded according to an ISO standard or external registry.|
|99|SPF|Obsolete|Historical dedicated SPF record type. SPF data is published using TXT records instead.|
|100|UINFO|IANA Reserved|IANA-reserved User Information record type. Not intended for normal deployment.|
|101|UID|IANA Reserved|IANA-reserved User Identifier record type.|
|102|GID|IANA Reserved|IANA-reserved Group Identifier record type.|
|103|UNSPEC|IANA Reserved|IANA-reserved unspecified record type.|
|104|NID|Assigned / ILNP|Node Identifier record used by the Identifier-Locator Network Protocol (ILNP).|
|105|L32|Assigned / ILNP|32-bit Locator record used by ILNP.|
|106|L64|Assigned / ILNP|64-bit Locator record used by ILNP.|
|107|LP|Assigned / ILNP|Locator Pointer record used by ILNP.|
|108|EUI48|Assigned|Stores an IEEE EUI-48 identifier.|
|109|EUI64|Assigned|Stores an IEEE EUI-64 identifier.|
|128|NXNAME|Assigned / DNSSEC-related|NXDOMAIN indicator used by Compact Denial of Existence.|
|249|TKEY|Meta / Security|Transaction Key. Used to establish or negotiate shared keys for DNS transactions.|
|250|TSIG|Meta / Security|Transaction Signature. Authenticates DNS messages using shared-secret cryptography.|
|251|IXFR|QTYPE / Zone Transfer|Requests an incremental DNS zone transfer containing only changes since a known version.|
|252|AXFR|QTYPE / Zone Transfer|Requests a complete DNS zone transfer.|
|253|MAILB|QTYPE / Legacy|Query type for historical mailbox-related records such as MB, MG, and MR.|
|254|MAILA|Obsolete QTYPE|Historical mail-agent query type. Obsolete; MX is used instead.|
|255|ANY (`*`)|QTYPE / Meta|Requests some or all RR data available for a name. Servers may intentionally return limited answers.|
|256|URI|Assigned|Publishes a URI associated with a DNS name, including priority and weight information.|
|257|CAA|Assigned|Certification Authority Authorization. Publishes which CAs are authorized to issue certificates for a domain.|
|258|AVC|Assigned / Specialized|Application Visibility and Control record.|
|259|DOA|Assigned / Specialized|Digital Object Architecture record.|
|260|AMTRELAY|Assigned|Identifies an Automatic Multicast Tunneling relay.|
|261|RESINFO|Assigned|Publishes DNS resolver information as key/value pairs.|
|262|WALLET|Assigned|Publishes a public wallet address in DNS.|
|263|CLA|Assigned / Specialized|Bundle Protocol Convergence Layer Adapter information.|
|264|IPN|Assigned / Specialized|Bundle Protocol IPN node number information.|
|32768|TA|Assigned / DNSSEC Legacy|DNSSEC Trust Authorities record. Specialized historical trust-anchor mechanism.|
|32769|DLV|Obsolete|DNSSEC Lookaside Validation record. Obsolete.|

---

## Unassigned, Private-Use, and Reserved Values

Not every numeric RR TYPE value currently has an assigned mnemonic.

|Value / Range|Status|Notes|
|---|---|---|
|54|Unassigned|Available for future assignment according to IANA policy.|
|71-98|Unassigned|Currently not assigned to named RR types.|
|110-127|Unassigned|Currently not assigned to named RR types.|
|129-248|Unassigned|Currently not assigned to named RR types.|
|265-32767|Unassigned|Large range available for future RR type assignments.|
|32770-61439|Unassigned|Currently not assigned to named RR types.|
|61440-65279|Reserved for future use|IETF review is required before this range can be defined for use.|
|65280-65534|Private Use|Reserved for private or experimental deployments that do not require global RR type assignment.|
|65535|Reserved|Reserved; not available for ordinary use.|

---

## Commonly Encountered Records

For normal systems and network administration, the record types most frequently encountered are:

|Type|Primary Purpose|
|---|---|
|A|DNS name to IPv4 address|
|AAAA|DNS name to IPv6 address|
|CNAME|Alias to another DNS name|
|MX|Mail exchanger discovery|
|NS|Authoritative name server|
|SOA|Zone authority and synchronization metadata|
|PTR|Reverse DNS lookup|
|TXT|Text, verification, and policy data|
|SRV|Service discovery|
|CAA|Certificate Authority authorization|
|DS|DNSSEC delegation|
|DNSKEY|DNSSEC public key|
|RRSIG|DNSSEC signature|
|NSEC / NSEC3|DNSSEC authenticated denial of existence|
|TLSA|DANE TLS certificate/public-key association|
|SVCB|General service endpoint discovery|
|HTTPS|HTTP/HTTPS endpoint and connection-parameter discovery|

For a detailed explanation of the common records, see  [DNS Records](../../04-dns-records/dns-records.md).

---

## Status Notes

### Assigned

The RR type has an official IANA assignment.

This does not necessarily mean that the record is commonly deployed.

### Legacy / Specialized

The RR type remains assigned but is uncommon, historical, or intended for a narrow protocol or technology.

### Experimental

The RR type was registered for experimental use and should not be treated as a normal modern DNS record without understanding the associated specification.

### Deprecated / Obsolete

The RR type should generally not be used for new deployments.

Examples include:

```
MD
MF
NXT
A6
SPF
DLV
```

### Meta / QTYPE

Some registered TYPE values do not represent ordinary persistent records stored in a DNS zone.

Examples include:

```
OPT
TKEY
TSIG
IXFR
AXFR
ANY
```

They are used by DNS protocol operations, queries, extensions, authentication, or zone transfer mechanisms.

---

## Reference Maintenance

The DNS RR TYPE registry changes over time.

When maintaining this Knowledge Base, this file should be checked against the current:

```
IANA
Domain Name System (DNS) Parameters
Resource Record (RR) TYPEs
```

before major revisions or when a new DNS record type appears in documentation or packet captures.

---

## Key Takeaways

- DNS supports many more record types than the small set commonly used by administrators.
- IANA maintains the authoritative registry of DNS RR TYPE values.
- Record types have numeric values in addition to their familiar mnemonic names.
- Some RR types are normal zone data, while others are meta-types or query types.
- Some historical RR types remain registered even though they are obsolete or rarely used.
- Unassigned RR TYPE numbers are not valid substitutes for private experimentation.
- Values `65280-65534` are reserved for private use.
- The most commonly encountered records remain A, AAAA, CNAME, MX, NS, SOA, PTR, TXT, SRV, and CAA.
- DNSSEC introduces several additional important record types, including DS, DNSKEY, RRSIG, NSEC, and NSEC3.
- Modern service discovery increasingly uses records such as SVCB and HTTPS.
- This file is intended as a reference table; detailed explanations of common records belong in  [DNS Records](../../04-dns-records/dns-records.md)