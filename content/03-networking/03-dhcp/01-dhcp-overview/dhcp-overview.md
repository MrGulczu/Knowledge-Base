# DHCP Overview

## What Is DHCP?

**DHCP (Dynamic Host Configuration Protocol)** is a network protocol  
used to automatically provide devices with the network configuration  
they need to communicate.

When a new device connects to a network, it needs an IP configuration  
appropriate for that network. The fundamentals of IPv4 addresses, subnet masks, default gateways, and static versus dynamic addressing were covered earlier in [IP Addressing](../../01-fundamentals/04-ip-addressing/ip-addressing.md).

Without DHCP, this information would have to be configured manually on each device.

In a small network with only a few devices, manual configuration may be manageable. As the number of devices increases, manually maintaining  
their network configuration becomes increasingly difficult and prone to mistakes.

DHCP allows this configuration to be managed centrally and distributed  
automatically to clients.

## Why Do We Need DHCP?

Consider a network containing 50 employee laptops.

Without DHCP, an administrator would have to manually configure every  
laptop and keep track of which IP addresses were already assigned.

For example:

```
Laptop-01 → 192.168.10.20
Laptop-02 → 192.168.10.21
Laptop-03 → 192.168.10.22
Laptop-04 → 192.168.10.23
...
```

This may work with a small number of devices, but it becomes  
increasingly difficult to maintain as the network grows.

Accidentally assigning the same IP address to multiple devices can  
create an **IP address conflict**, potentially disrupting connectivity  
and making troubleshooting more difficult.

DHCP automates this process by managing address assignment for clients.

## DHCP Provides More Than an IP Address

Although DHCP is commonly associated with automatic IP address  
assignment, it can provide considerably more information.

A basic DHCP configuration might provide a client with:

```
IP address:      192.168.10.25
Subnet mask:     255.255.255.0
Default gateway: 192.168.10.1
DNS server:      192.168.10.10
```

Each piece of information serves a different purpose:

- **IP address** identifies the device on the IP network.
- **Subnet mask** determines which addresses belong to the local  
    network.
- **Default gateway** provides a path toward remote networks.
- **DNS server** tells the client where it can send DNS queries. The role  
    of DNS itself was covered earlier in  
    [DNS Fundamentals](../../02-dns/01-dns-fundamentals/dns-fundamentals.md).

DHCP can also distribute additional network parameters through **DHCP  
options**, which will be discussed separately.

## Basic DHCP Operation

When a DHCP-enabled device connects to an IPv4 network, it attempts to  
locate an available DHCP server.

At a simplified level, the process looks like this:

```
New Client
    |
    | "Is there a DHCP server?"
    | Broadcast
    v
DHCP Server
    |
    | Offers network configuration
    v
New Client
    |
    | Requests / accepts configuration
    v
Configured Client
```

The client initially uses broadcast communication because it does not  
yet know which DHCP server it should communicate with.

The exact message exchange and the **DORA process** will be covered in a  
dedicated topic.

## DHCP and Static Configuration

Using DHCP does not mean that every device in a network must receive  
dynamically changing addresses.

User endpoints such as laptops and phones are well suited for DHCP  
because they can automatically obtain configuration whenever they  
connect to a network.

Infrastructure devices often require predictable addresses.

A network might therefore use a combination such as:

```
Employee laptops/phones → DHCP
Servers                 → Static / predictable address
Routers and firewalls   → Static
Switch management       → Static
Printers                 → Static or DHCP reservation
```

For example, administrators and other systems may need to know where a  
router, server, switch, or printer can consistently be reached.

DHCP can also provide a device with the same address repeatedly through a **DHCP reservation**. Reservations will be discussed separately.

## Centralized Network Configuration

One major advantage of DHCP is that client network configuration can be managed centrally.

Consider an organization with 300 computers using the following DNS  
server:

```
192.168.10.10
```

The organization replaces it with:

```
192.168.10.20
```

If every computer is manually configured, the DNS configuration may need to be changed individually on each machine.

With DHCP, the administrator can change the appropriate DHCP  
configuration centrally. Clients can then receive the updated  
configuration through DHCP.

How and when existing clients receive updated information depends on  
DHCP leases and the renewal process, which will be discussed later.

## Moving Between Networks

DHCP is especially useful for mobile devices that regularly connect to  
different networks.

Consider a laptop configured with:

```
IP address: 192.168.10.25
Network:    192.168.10.0/24
```

The user moves the laptop from:

```
Office A
192.168.10.0/24
```

to:

```
Office B
10.20.30.0/24
```

If the laptop retained its static configuration from Office A, its  
address would not match the network used in Office B. Other  
configuration, such as its default gateway, could also be incorrect.

The user would normally be unable to use the new network until the  
configuration was changed.

With DHCP, the laptop can obtain configuration appropriate for the  
network to which it is currently connected.

```
Office A

Laptop
  |
  | DHCP
  v
192.168.10.25/24

        User moves to Office B

Office B

Laptop
  |
  | DHCP
  v
10.20.30.25/24
```

This allows devices to move between networks without requiring users or administrators to manually reconfigure their network settings each time.

## Key Takeaways

DHCP simplifies network administration by automatically providing  
clients with the configuration required to communicate on a network.

Instead of manually configuring every endpoint, administrators can  
centrally manage parameters such as IP addresses, subnet masks, default gateways, and DNS servers.

This becomes increasingly important as networks grow because it reduces administrative overhead, helps prevent configuration mistakes, and  
allows devices to move between networks more easily.

DHCP does not completely replace static addressing. Infrastructure and  
other devices that require predictable addresses may still use static  
configuration or DHCP reservations.

In the next topics, we will look more closely at the components involved in DHCP and how clients and DHCP servers actually communicate.