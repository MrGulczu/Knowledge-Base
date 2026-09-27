# Ports Reference

## Overview

This file is a quick-reference catalog of commonly encountered network ports and services.

For the concepts behind ports, sockets, client/server communication, and ephemeral ports, see [Ports and Sockets](./ports-sockets.md).

The authoritative registry for service names and transport protocol port numbers is maintained by IANA:

[https://www.iana.org/assignments/service-names-port-numbers/](https://www.iana.org/assignments/service-names-port-numbers/)

The complete IANA registry contains thousands of assignments and changes over time. This Knowledge Base reference therefore focuses on ports that are most useful in networking, systems administration, cybersecurity, troubleshooting, enterprise infrastructure, and application environments.

> A registered port number does **not** prove that traffic using that port actually belongs to the registered service. Applications can use different ports, and malicious traffic can use otherwise legitimate port numbers.

---

## Port Number Ranges

TCP, UDP, SCTP, and DCCP use 16-bit port numbers.

The valid numeric range is:

```
0-65535
```

IANA divides the range into three major categories.

|Range|Name|Purpose|
|---|---|---|
|`0-1023`|System Ports / Well-Known Ports|Core and widely standardized network services. Assignments normally require IETF Review or IESG approval.|
|`1024-49151`|User Ports / Registered Ports|Registered applications, products, and services.|
|`49152-65535`|Dynamic / Private Ports|Not assigned by IANA. Commonly used for ephemeral client ports and private application use.|

Port `0` is reserved and is not normally used as an application service port.

---

## Reading the Tables

The same numeric port can exist independently for different transport protocols.

For example:

```
53/TCP
53/UDP
```

are separate transport endpoints even though both are commonly associated with DNS.

Similarly:

```
443/TCP
443/UDP
```

can both carry web traffic, but they are used by different transport mechanisms.

Throughout this reference:

```
TCP
→ connection-oriented transport

UDP
→ connectionless transport
```

For more detail, see [TCP and UDP](../09-tcp-udp/tcp-udp.md).

---

# System / Well-Known Ports

## Core Infrastructure and Legacy Services

|Port|Protocol|Service|Typical Use|
|---|---|---|---|
|0|TCP / UDP|Reserved|Reserved port number; not normally used by applications.|
|7|TCP / UDP|Echo|Returns received data. Historical diagnostic service; normally disabled today.|
|9|TCP / UDP|Discard|Discards received traffic. Historical testing service.|
|13|TCP / UDP|Daytime|Historical service returning the current date and time.|
|17|TCP / UDP|QOTD|Quote of the Day. Historical service.|
|19|TCP / UDP|CHARGEN|Character Generator. Historical testing service and commonly disabled because of abuse potential.|
|20|TCP|FTP Data|Traditional FTP data channel when active FTP mode is used.|
|21|TCP|FTP Control|FTP command and control connection.|
|22|TCP|SSH|Secure Shell remote administration, tunneling, SCP, and SFTP.|
|23|TCP|Telnet|Unencrypted remote terminal access. Normally replaced by SSH.|
|25|TCP|SMTP|Server-to-server email transport using Simple Mail Transfer Protocol.|
|37|TCP / UDP|Time|Historical Time Protocol. Not the same as NTP.|
|43|TCP|WHOIS|WHOIS directory queries.|
|49|TCP|TACACS+|Centralized authentication, authorization, and accounting, especially for network devices.|
|53|TCP / UDP|DNS|Domain Name System queries and responses. UDP is common for ordinary queries; TCP is used where required, including some larger responses and zone operations.|
|67|UDP|DHCP Server|DHCPv4 server listens for client requests.|
|68|UDP|DHCP Client|DHCPv4 client receives server responses.|
|69|UDP|TFTP|Trivial File Transfer Protocol. Simple file transfer with no built-in encryption or authentication.|
|70|TCP|Gopher|Historical Gopher information service.|
|79|TCP|Finger|Historical user-information protocol.|
|80|TCP|HTTP|Unencrypted HTTP web traffic.|
|88|TCP / UDP|Kerberos|Kerberos authentication, widely used by Active Directory and other identity systems.|
|110|TCP|POP3|Retrieves email from a mail server without transport encryption by default.|
|111|TCP / UDP|RPCBind / Portmapper|Maps RPC services to network ports, commonly associated with Unix/Linux RPC and NFS environments.|
|113|TCP|Ident|Historical Identification Protocol.|
|119|TCP|NNTP|Network News Transfer Protocol.|
|123|UDP|NTP|Network Time Protocol used for time synchronization.|
|135|TCP / UDP|Microsoft RPC Endpoint Mapper|Helps clients discover Microsoft RPC services and dynamically assigned RPC ports.|
|137|UDP|NetBIOS Name Service|Legacy NetBIOS hostname registration and lookup.|
|138|UDP|NetBIOS Datagram Service|Legacy NetBIOS datagram traffic.|
|139|TCP|NetBIOS Session Service|Legacy SMB/file-sharing sessions over NetBIOS.|
|143|TCP|IMAP|Email mailbox access without implicit TLS.|
|161|UDP|SNMP|Network-device monitoring and management using SNMP.|
|162|UDP|SNMP Trap|Asynchronous SNMP notifications and traps.|
|179|TCP|BGP|Border Gateway Protocol used to exchange routing information between autonomous systems.|
|389|TCP / UDP|LDAP|Lightweight Directory Access Protocol. TCP is common for directory queries; UDP can be used by connectionless LDAP mechanisms such as CLDAP.|
|427|TCP / UDP|SLP|Service Location Protocol used for service discovery.|
|443|TCP / UDP|HTTPS|Encrypted HTTP. TCP is used by HTTP/1.1 and HTTP/2; UDP is used by HTTP/3 over QUIC.|
|445|TCP|SMB / Microsoft-DS|Modern SMB file sharing, Windows domain services, named pipes, and related Microsoft networking.|
|464|TCP / UDP|Kerberos Password Change|Kerberos password change and password-setting protocol.|
|465|TCP|Submissions|Email message submission using implicit TLS.|
|500|UDP|IKE / ISAKMP|Internet Key Exchange used to establish IPsec security associations.|
|514|UDP|Syslog|Traditional unencrypted syslog transport.|
|515|TCP|LPD / LPR|Line Printer Daemon printing protocol.|
|520|UDP|RIP|Routing Information Protocol for IPv4.|
|521|UDP|RIPng|Routing Information Protocol next generation for IPv6.|
|546|UDP|DHCPv6 Client|DHCPv6 client traffic.|
|547|UDP|DHCPv6 Server|DHCPv6 server/relay traffic.|
|548|TCP|AFP|Apple Filing Protocol over TCP.|
|554|TCP / UDP|RTSP|Real Time Streaming Protocol session control.|
|563|TCP|NNTPS|NNTP protected with TLS.|
|587|TCP|Message Submission|Standard port for authenticated client submission of outgoing email, commonly with STARTTLS.|
|631|TCP|IPP / IPPS|Internet Printing Protocol and secure IPP printing.|
|636|TCP|LDAPS|LDAP protected with TLS.|
|646|TCP / UDP|LDP|MPLS Label Distribution Protocol.|
|749|TCP|Kerberos Administration|Kerberos administrative service.|
|853|TCP / UDP|Encrypted DNS|TCP is commonly used for DNS over TLS (DoT); UDP can be used by encrypted DNS transports such as DNS over QUIC depending on implementation.|
|873|TCP|rsync|File synchronization using rsync.|
|989|TCP|FTPS Data|FTP data connection protected with TLS.|
|990|TCP|FTPS|FTP control connection using implicit TLS.|
|992|TCP|Telnet over TLS|Telnet protected with TLS; uncommon.|
|993|TCP|IMAPS|IMAP protected with TLS.|
|995|TCP|POP3S|POP3 protected with TLS.|

---

# Common User / Registered Ports

## Proxy, VPN, Remote Access, and Tunneling

|Port|Protocol|Service|Typical Use|
|---|---|---|---|
|1080|TCP|SOCKS|SOCKS proxy, commonly SOCKS5.|
|1194|TCP / UDP|OpenVPN|Default registered port for OpenVPN VPN tunnels.|
|1494|TCP|Citrix ICA|Citrix Independent Computing Architecture sessions.|
|1701|UDP|L2TP|Layer 2 Tunneling Protocol. Often combined with IPsec.|
|1723|TCP|PPTP|Point-to-Point Tunneling Protocol control connection. PPTP is considered obsolete/insecure for modern VPN use.|
|3389|TCP / UDP|RDP|Microsoft Remote Desktop Protocol.|
|3478|TCP / UDP|STUN / TURN|NAT traversal and real-time communications, including WebRTC-related environments.|
|4500|UDP|IPsec NAT-T|IPsec NAT Traversal when IPsec traffic passes through NAT.|
|51820|UDP|WireGuard|Common/default WireGuard listening port. Deployments can use any UDP port.|
|5900|TCP|VNC|Common base port for Virtual Network Computing remote desktop.|

---

## Authentication and Directory Services

|Port|Protocol|Service|Typical Use|
|---|---|---|---|
|1645|UDP|RADIUS Authentication (Legacy)|Historical RADIUS authentication port. Modern deployments normally use 1812.|
|1646|UDP|RADIUS Accounting (Legacy)|Historical RADIUS accounting port. Modern deployments normally use 1813.|
|1812|UDP|RADIUS Authentication|Authentication and authorization requests for RADIUS.|
|1813|UDP|RADIUS Accounting|RADIUS accounting information.|
|3268|TCP|LDAP Global Catalog|Microsoft Active Directory Global Catalog queries without implicit TLS.|
|3269|TCP|LDAPS Global Catalog|Active Directory Global Catalog queries protected with TLS.|

---

## Databases and Data Services

|Port|Protocol|Service|Typical Use|
|---|---|---|---|
|1433|TCP|Microsoft SQL Server|Default Microsoft SQL Server database connection port.|
|1434|UDP|Microsoft SQL Browser|SQL Server instance discovery and SQL Browser service.|
|1521|TCP|Oracle Net|Common Oracle Database listener port.|
|3050|TCP|Firebird|Common Firebird database service port.|
|3306|TCP|MySQL / MariaDB|Default MySQL and MariaDB database connections.|
|5432|TCP|PostgreSQL|Default PostgreSQL database connections.|
|6379|TCP|Redis|Common/default Redis service port.|
|9042|TCP|Cassandra CQL|Apache Cassandra native CQL client connections.|
|11211|TCP / UDP|Memcached|Common Memcached caching service port. UDP support may be disabled in modern deployments.|
|27017|TCP|MongoDB|Common/default MongoDB database port.|

---

## File, Storage, and Version-Control Services

|Port|Protocol|Service|Typical Use|
|---|---|---|---|
|2049|TCP / UDP|NFS|Network File System. Modern NFS commonly uses TCP.|
|3260|TCP|iSCSI|iSCSI storage target communication.|
|3690|TCP|Subversion|Native Subversion version-control protocol.|
|9418|TCP|Git|Native unauthenticated Git protocol (`git://`).|

---

## Messaging, IoT, and Real-Time Communication

|Port|Protocol|Service|Typical Use|
|---|---|---|---|
|1883|TCP|MQTT|MQTT messaging without TLS, common in IoT environments.|
|5004|UDP|RTP|Common Real-time Transport Protocol media port assignment. Actual RTP deployments can negotiate other ports.|
|5005|UDP|RTCP|Common Real-time Transport Control Protocol assignment.|
|5060|TCP / UDP|SIP|Session Initiation Protocol signaling without implicit TLS.|
|5061|TCP|SIP over TLS|SIP signaling protected with TLS.|
|5222|TCP|XMPP Client|XMPP client-to-server communication.|
|5269|TCP|XMPP Server|XMPP server-to-server communication.|
|5353|UDP|mDNS|Multicast DNS, commonly used for local-link service discovery.|
|5355|TCP / UDP|LLMNR|Link-Local Multicast Name Resolution in Microsoft-oriented networks.|
|5671|TCP|AMQPS|AMQP messaging protected with TLS.|
|5672|TCP|AMQP|Advanced Message Queuing Protocol.|
|5683|UDP / TCP|CoAP|Constrained Application Protocol, primarily used by IoT devices.|
|5684|UDP / TCP|CoAPS|CoAP protected with DTLS/TLS depending on transport.|
|8883|TCP|MQTT over TLS|Common MQTT encrypted with TLS port.|

---

## Web, Proxy, and Application Services

|Port|Protocol|Service|Typical Use|
|---|---|---|---|
|3000|TCP|Development Web Applications|Common default for development frameworks and services such as Grafana in some deployments. Not tied to one universal application.|
|3128|TCP|HTTP Proxy|Common Squid and web-proxy port.|
|5601|TCP|Kibana|Common/default Kibana web interface port.|
|8000|TCP|Alternative HTTP / Development|Frequently used by development web servers and application services.|
|8008|TCP|Alternative HTTP|Common alternative HTTP/application port; exact use depends on software.|
|8080|TCP|HTTP Alternate|Common alternate web, proxy, and application-server port.|
|8081|TCP|Alternative HTTP/Application|Frequently used by web applications and management interfaces.|
|8443|TCP|HTTPS Alternate|Common alternate port for HTTPS-protected web applications and management interfaces.|
|9000|TCP|Application / Management|Commonly used by multiple products; meaning is application-specific.|
|9090|TCP|Application / Monitoring|Frequently used by management and monitoring applications.|
|9443|TCP|HTTPS Alternate|Common alternative TLS web-management port.|

---

## Monitoring, Logging, and Management

|Port|Protocol|Service|Typical Use|
|---|---|---|---|
|514|UDP|Syslog|Traditional syslog transport.|
|162|UDP|SNMP Trap|Event and alert notifications from SNMP devices.|
|5985|TCP|WinRM HTTP|Windows Remote Management over HTTP.|
|5986|TCP|WinRM HTTPS|Windows Remote Management protected with TLS.|
|6514|TCP|Syslog over TLS|Secure syslog transport using TLS.|
|9100|TCP|Raw Printing / JetDirect|Raw network printing used by many network printers.|
|10050|TCP|Zabbix Agent|Common/default Zabbix agent port.|
|10051|TCP|Zabbix Server / Proxy|Common/default Zabbix server and proxy communication port.|

---

## Containers, Orchestration, and Distributed Systems

|Port|Protocol|Service|Typical Use|
|---|---|---|---|
|2181|TCP|ZooKeeper|Common/default Apache ZooKeeper client port.|
|2375|TCP|Docker API|Common Docker Engine API port without TLS. Exposing it remotely without protection is dangerous.|
|2376|TCP|Docker API TLS|Common Docker Engine API port protected with TLS.|
|2379|TCP|etcd Client|Common/default etcd client API port.|
|2380|TCP|etcd Peer|Common/default etcd peer-to-peer communication.|
|4789|UDP|VXLAN|Virtual Extensible LAN overlay encapsulation.|
|6443|TCP|Kubernetes API|Common/default Kubernetes API server port.|
|8472|UDP|VXLAN / Overlay Networking|Commonly used by some Kubernetes overlay-network implementations.|
|9092|TCP|Apache Kafka|Common/default Kafka broker listener port.|
|10250|TCP|Kubernetes Kubelet|Kubelet API on Kubernetes nodes.|
|30000-32767|TCP / UDP|Kubernetes NodePort|Default Kubernetes NodePort service range. This is a Kubernetes convention, not a single IANA service assignment.|

---

## Search, Message Brokers, and Application Platforms

|Port|Protocol|Service|Typical Use|
|---|---|---|---|
|4369|TCP|Erlang Port Mapper Daemon|Erlang node discovery, used by systems such as RabbitMQ.|
|5671|TCP|AMQPS|TLS-protected AMQP.|
|5672|TCP|AMQP|Messaging protocol used by products such as RabbitMQ.|
|9200|TCP|Elasticsearch HTTP API|Common/default Elasticsearch REST/HTTP interface.|
|9300|TCP|Elasticsearch Transport|Common/default Elasticsearch node transport communication.|
|15672|TCP|RabbitMQ Management|Common/default RabbitMQ HTTP management interface.|

---

# Important Enterprise Combinations

Some enterprise technologies require several ports rather than one single port.

## Active Directory

Common Active Directory-related ports include:

|Port|Protocol|Purpose|
|---|---|---|
|53|TCP / UDP|DNS|
|88|TCP / UDP|Kerberos authentication|
|123|UDP|Time synchronization|
|135|TCP|RPC Endpoint Mapper|
|389|TCP / UDP|LDAP / CLDAP|
|445|TCP|SMB|
|464|TCP / UDP|Kerberos password change|
|636|TCP|LDAP over TLS|
|3268|TCP|Global Catalog|
|3269|TCP|Global Catalog over TLS|
|Dynamic RPC range|TCP|Additional RPC services negotiated through the endpoint mapper|

Modern Windows systems normally use a dynamic RPC port range rather than one fixed port for all RPC services.

---

## Email

|Port|Protocol|Purpose|
|---|---|---|
|25|TCP|SMTP server-to-server delivery|
|465|TCP|Message submission with implicit TLS|
|587|TCP|Authenticated message submission, commonly with STARTTLS|
|110|TCP|POP3|
|995|TCP|POP3 over TLS|
|143|TCP|IMAP|
|993|TCP|IMAP over TLS|

A client sending email normally should not be assumed to use TCP/25. Port 25 is primarily associated with mail-server-to-mail-server delivery, while client submission commonly uses 587 or 465.

---

## DNS

|Port|Protocol|Purpose|
|---|---|---|
|53|UDP|Normal DNS queries and responses|
|53|TCP|DNS over TCP, larger responses when needed, and DNS operations that require TCP|
|853|TCP|DNS over TLS|
|853|UDP|Used by encrypted DNS transports assigned to the secure DNS service port, including DNS over QUIC implementations|

DNS itself is covered in the dedicated DNS section of this Knowledge Base.

---

## IPsec VPN

|Port|Protocol|Purpose|
|---|---|---|
|500|UDP|IKE / initial IPsec negotiation|
|4500|UDP|IPsec NAT Traversal|

IPsec also uses IP protocol numbers such as ESP rather than only TCP or UDP port numbers.

This distinction is important:

```
TCP / UDP
→ use port numbers

ESP
→ IP protocol number 50
```

A protocol number is not the same thing as a port number.

---

## File Sharing

|Port|Protocol|Purpose|
|---|---|---|
|445|TCP|SMB|
|139|TCP|Legacy SMB over NetBIOS|
|137|UDP|NetBIOS Name Service|
|138|UDP|NetBIOS Datagram Service|
|2049|TCP / UDP|NFS|
|548|TCP|AFP|

Modern Windows SMB normally uses TCP/445 directly.

---

# Dynamic and Ephemeral Ports

The IANA Dynamic / Private Port range is:

```
49152-65535
```

These ports are not assigned to permanent services by IANA.

They are commonly used as temporary source ports when a client starts a connection.

For example:

```
Client:
192.168.10.25:52743

Server:
203.0.113.20:443
```

The client uses:

```
52743
```

as a temporary source port while the server listens on:

```
443
```

Conceptually:

```
192.168.10.25:52743
        |
        | HTTPS connection
        v
203.0.113.20:443
```

Operating systems can use implementation-specific ephemeral-port ranges, so the exact local range should not be assumed solely from the IANA dynamic/private range.

For the relationship between IP addresses, ports, and sockets, see [Ports and Sockets](./ports-sockets.md).

---

# Ports Are Not Protocol Identification

A port number should not be treated as proof of the application protocol being transported.

For example:

```
TCP/443
```

normally suggests HTTPS.

However, an application can potentially listen on:

```
TCP/443
```

and carry completely different traffic.

Likewise:

```
TCP/22
```

normally indicates SSH, but observing port 22 alone does not cryptographically prove that the traffic is SSH.

This is important for:

- firewall analysis,
- packet captures,
- IDS/IPS systems,
- network troubleshooting,
- threat hunting,
- service discovery.

A registered port should therefore be treated as an **expected or conventional service association**, not definitive identification of traffic contents.

---

# Security Notes

Opening a firewall port means allowing traffic to reach a transport endpoint.

It does not make the service itself secure.

For example:

```
TCP/22 allowed
```

only means SSH traffic can potentially reach a listening service.

Security still depends on factors such as:

- authentication,
- encryption,
- software patching,
- access-control rules,
- source restrictions,
- service configuration,
- logging and monitoring.

Legacy protocols such as:

```
Telnet
FTP
TFTP
HTTP
POP3
IMAP
```

may transmit sensitive information without encryption unless additional protection is used.

Encrypted alternatives or protected tunnels should generally be preferred where appropriate.

---

# Quick Reference

|Port|Protocol|Common Service|
|---|---|---|
|20|TCP|FTP Data|
|21|TCP|FTP|
|22|TCP|SSH|
|23|TCP|Telnet|
|25|TCP|SMTP|
|53|TCP / UDP|DNS|
|67|UDP|DHCP Server|
|68|UDP|DHCP Client|
|69|UDP|TFTP|
|80|TCP|HTTP|
|88|TCP / UDP|Kerberos|
|110|TCP|POP3|
|123|UDP|NTP|
|135|TCP|Microsoft RPC|
|137-139|TCP / UDP|NetBIOS|
|143|TCP|IMAP|
|161|UDP|SNMP|
|162|UDP|SNMP Trap|
|179|TCP|BGP|
|389|TCP / UDP|LDAP|
|443|TCP / UDP|HTTPS / HTTP3|
|445|TCP|SMB|
|464|TCP / UDP|Kerberos Password Change|
|465|TCP|SMTP Submission over TLS|
|500|UDP|IKE / IPsec|
|514|UDP|Syslog|
|546|UDP|DHCPv6 Client|
|547|UDP|DHCPv6 Server|
|587|TCP|SMTP Submission|
|636|TCP|LDAPS|
|853|TCP / UDP|Secure DNS transports|
|993|TCP|IMAPS|
|995|TCP|POP3S|
|1194|TCP / UDP|OpenVPN|
|1433|TCP|Microsoft SQL Server|
|1521|TCP|Oracle Database|
|1701|UDP|L2TP|
|1812|UDP|RADIUS Authentication|
|1813|UDP|RADIUS Accounting|
|1883|TCP|MQTT|
|2049|TCP / UDP|NFS|
|3260|TCP|iSCSI|
|3268|TCP|AD Global Catalog|
|3269|TCP|AD Global Catalog TLS|
|3306|TCP|MySQL / MariaDB|
|3389|TCP / UDP|RDP|
|4500|UDP|IPsec NAT-T|
|4789|UDP|VXLAN|
|5060|TCP / UDP|SIP|
|5061|TCP|SIP over TLS|
|5353|UDP|mDNS|
|5355|TCP / UDP|LLMNR|
|5432|TCP|PostgreSQL|
|5671|TCP|AMQPS|
|5672|TCP|AMQP|
|5900|TCP|VNC|
|5985|TCP|WinRM HTTP|
|5986|TCP|WinRM HTTPS|
|6379|TCP|Redis|
|6443|TCP|Kubernetes API|
|6514|TCP|Syslog over TLS|
|8080|TCP|HTTP Alternate|
|8443|TCP|HTTPS Alternate|
|8883|TCP|MQTT over TLS|
|9092|TCP|Kafka|
|9100|TCP|Raw Network Printing|
|9200|TCP|Elasticsearch HTTP API|
|9418|TCP|Git|
|11211|TCP / UDP|Memcached|
|27017|TCP|MongoDB|

---

# Reference Maintenance

Port assignments change over time.

When maintaining this Knowledge Base, this reference should periodically be checked against the current IANA registry:

```
IANA
Service Name and Transport Protocol Port Number Registry
```

[https://www.iana.org/assignments/service-names-port-numbers/](https://www.iana.org/assignments/service-names-port-numbers/)

The IANA registry should remain the source of truth for officially registered service-name and port-number assignments.

Vendor documentation should be used as the source of truth for product-specific default ports because software can use ports that are not uniquely represented by a single IANA service assignment.

---

# Key Takeaways

- Transport-layer ports use numbers from `0` through `65535`.
- System / Well-Known Ports occupy `0-1023`.
- User / Registered Ports occupy `1024-49151`.
- Dynamic / Private Ports occupy `49152-65535`.
- TCP and UDP have separate port namespaces.
- The same number can therefore be used by both TCP and UDP.
- A port number identifies a transport endpoint, not a physical network port.
- A port number does not prove which application protocol is actually being carried.
- Well-known ports make common network services easier to discover and configure.
- Registered ports are commonly associated with specific applications but are not mandatory for every implementation.
- Dynamic ports are commonly used as temporary client-side source ports.
- Enterprise technologies often require groups of ports rather than a single port.
- Some network protocols, such as IPsec ESP, use IP protocol numbers rather than TCP or UDP ports.
- Firewall rules should be based on the actual application and security requirements, not only on whether a port has an IANA registration.
- IANA maintains the authoritative registry for official service-name and port-number assignments.