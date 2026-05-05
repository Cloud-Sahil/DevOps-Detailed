# 🌐 Networking for DevOps — Complete Notes
> **Modular · Interview-Ready · Real-World DevOps**
> Covers: OSI Model · TCP/IP · Protocols · Ports · Subnetting · DNS · DHCP · VPN · NAT · VLAN · Security · Troubleshooting · Cloud Networking

---

## 📚 Table of Contents

| # | Section | Topics |
|---|---------|--------|
| 1 | [What is a Network?](#1-what-is-a-network) | Definition, Internet, Why networks |
| 2 | [Types of Networks](#2-types-of-networks) | LAN, MAN, WAN, SONET |
| 3 | [Network Components](#3-network-components) | Switch, Router, Hub, NIC, Bridge |
| 4 | [IP Address](#4-ip-address) | IPv4, IPv6, Format |
| 5 | [Types of IP Addresses](#5-types-of-ip-addresses) | Public, Private, Static, Dynamic, Loopback |
| 6 | [IPv4 Classes](#6-ipv4-classes) | Class A–E, IANA, Private ranges |
| 7 | [Loopback & Special Addresses](#7-loopback--special-addresses) | 127.0.0.1, 0.0.0.0, APIPA |
| 8 | [Network ID vs Host ID](#8-network-id-vs-host-id) | Octet breakdown, routing rules |
| 9 | [Subnetting](#9-subnetting) | CIDR, /24 to /25, benefits |
| 10 | [CIDR Reference](#10-cidr-reference) | Prefix table, notation |
| 11 | [OSI Model (7 Layers)](#11-osi-model-7-layers) | All 7 layers, protocols, data units |
| 12 | [TCP/IP Model (4 Layers)](#12-tcpip-model-4-layers) | Layers, comparison to OSI |
| 13 | [Protocols](#13-protocols) | TCP, UDP, ICMP, ARP |
| 14 | [Ports & Protocol Reference](#14-ports--protocol-reference) | Complete port table (70+ ports) |
| 15 | [HTTP Basics](#15-http-basics) | Methods, Status Codes, Cookies |
| 16 | [TCP vs UDP](#16-tcp-vs-udp) | Comparison, handshake, use cases |
| 17 | [Routing](#17-routing) | Route table, priority, default route |
| 18 | [DNS](#18-dns) | Flow, record types, PING |
| 19 | [DHCP](#19-dhcp) | DORA process, what it assigns |
| 20 | [Network Security & Services](#20-network-security--services) | Firewall, Load Balancer, VPN, VLAN, NAT |
| 21 | [Network Troubleshooting Tools](#21-network-troubleshooting-tools) | ping, traceroute, curl, dig, nmap, ssh |
| 22 | [Cloud Networking Concepts](#22-cloud-networking-concepts) | VPC, Subnets, CDN, IaC, SDN |
| 23 | [Additional Concepts](#23-additional-concepts) | ARP, MAC, RTT, Topology, CRC, Metrics |
| 24 | [IPv4 vs IPv6 Deep Dive](#24-ipv4-vs-ipv6-deep-dive) | Full comparison table |
| 25 | [Quick Reference Cheat Sheet](#25-quick-reference-cheat-sheet) | All key facts at a glance |

---

## 1. What is a Network?

A **network** is formed when **two or more computers/devices** are connected through **wired or wireless communication** so they can **share data and resources**.

```
  Device A ──────── Switch ──────── Device B
                      │
                   Router
                      │
                  Internet
```

**Why Networks are Used:**

| Purpose | Description |
|---------|-------------|
| Communication | Send messages, requests, data between devices |
| Resource sharing | Printers, storage, servers, internet connections |
| Centralized management | Manage systems from one place |
| Remote access | Work from anywhere |
| Collaboration | Teams working on same systems simultaneously |

> ✅ The **Internet** is the world's largest network — often called the **"network of networks"**.

---

## 2. Types of Networks

Networks are categorized based on **size and geographic coverage**.

```
  [Home/Office]     [City]         [Country/World]     [Undersea]
  ┌──────────┐   ┌─────────┐      ┌─────────────┐    ┌──────────┐
  │   LAN    │   │   MAN   │      │    WAN      │    │  SONET   │
  └──────────┘   └─────────┘      └─────────────┘    └──────────┘
   Wi-Fi/LAN    City ISP net     Internet/Enterprise  Fiber Optic
```

| Type | Full Form | Coverage | Examples |
|------|-----------|----------|---------|
| **LAN** | Local Area Network | Home, office, school | Wi-Fi, Ethernet |
| **MAN** | Metropolitan Area Network | City or large campus | City-wide ISP |
| **WAN** | Wide Area Network | Countries, continents | Internet, enterprise global networks |
| **SONET** | Synchronous Optical Network | Long-distance, submarine | Undersea internet cables |

---

## 3. Network Components

A DevOps engineer must understand basic networking hardware because **cloud networking works on the same principles**.

```
  Internet
     │
  [Modem]  ← ISP signal modulation/demodulation
     │
  [Router]  ← Layer 3 — connects networks using IP
     │
  [Switch]  ← Layer 2 — connects devices using MAC
  ┌──┼──┐
 PC1 PC2 PC3   ← Each has a NIC with unique MAC address
```

| Component | Layer | Function |
|-----------|-------|----------|
| **Switch** | Layer 2 | Connects devices in same network using **MAC addresses**. Sends data only to intended device (smart). |
| **Router** | Layer 3 | Connects **different networks** using **IP addresses**. Routes traffic between LAN and internet. |
| **Modem** | Layer 1 | **Modulation/Demodulation** — converts ISP signals for internet communication. |
| **Hub** *(Obsolete)* | Layer 1 | Broadcasts data to **ALL ports** — very inefficient. Replaced by switches. |
| **NIC** | Layer 2 | Network Interface Card — hardware that connects device to network. Every NIC has a unique **MAC Address**. Now built into most motherboards. |
| **Bridge** *(Mostly Obsolete)* | Layer 2 | Connects two LANs, reduces traffic vs hubs. Replaced by switches. |
| **Firewall** | Layer 3–7 | Security guard — monitors data in/out, blocks suspicious traffic. |

---

## 4. IP Address

An **IP address** is a **unique identifier** given to a device on a network.

> ✅ Think of it as the **home address of a device** — so data can reach it correctly.

### IPv4

```
  192    .   168    .    1    .   10
  ───────    ───────    ───    ───────
  Octet 1   Octet 2  Octet 3  Octet 4
  (8 bits)  (8 bits) (8 bits) (8 bits)
  ─────────────────────────────────────
               Total = 32 bits
```

- **32-bit** address (4 octets × 8 bits)
- Format: `x.x.x.x` (decimal, dot-separated)
- Example: `192.168.1.10`
- Total IPv4 addresses ≈ **4.3 billion** (running out → hence IPv6)

### IPv6

```
  2001:0db8:85a3:0000:0000:8a2e:0370:7334
  ──── ──── ──── ──── ──── ──── ──── ────
   8 groups of 4 hexadecimal digits
              Total = 128 bits
```

- **128-bit** address (8 groups of 4 hex digits)
- Total addresses: **~340 undecillion** (virtually unlimited)
- Built-in **IPsec** security
- No broadcast — uses **Anycast** and **Multicast**
- Auto-configuration via **SLAAC** or **DHCPv6**

---

## 5. Types of IP Addresses

```
  ┌─────────────────────────────────────────────────────┐
  │                  IP Address Types                   │
  ├─────────────┬──────────────┬──────────┬─────────────┤
  │  Public IP  │  Private IP  │ Static IP│  Dynamic IP │
  │  (Internet) │  (LAN only)  │(Manual)  │   (DHCP)    │
  └─────────────┴──────────────┴──────────┴─────────────┘
```

| Type | Description | Example |
|------|-------------|---------|
| **Public IP** | Used on the internet. Assigned by ISP/cloud provider. Globally reachable. | Your website server IP |
| **Private IP** | Inside internal networks — NOT routable on public internet. | EC2 private IP inside a VPC |
| **Static IP** | Assigned manually or reserved. Does not change. | Servers, DB endpoints, VPN gateways |
| **Dynamic IP** | Automatically assigned using DHCP. Changes over time. | Laptops, home devices |
| **Loopback IP** | Reserved for device to communicate with itself. | `127.0.0.1` (localhost) |

**Private IPv4 Ranges (RFC 1918):**

| Class | Range | CIDR |
|-------|-------|------|
| A | `10.0.0.1` – `10.255.255.254` | `10.0.0.0/8` |
| B | `172.16.0.1` – `172.31.255.254` | `172.16.0.0/12` |
| C | `192.168.0.1` – `192.168.255.254` | `192.168.0.0/16` |

---

## 6. IPv4 Classes

IPv4 was traditionally divided into classes managed by **IANA** (Internet Assigned Numbers Authority).

```
  Binary first bits determine the class:
  ┌─────────┬────────────────┬─────────────────────────┐
  │ Class A │ 0xxxxxxx       │ 1.0.0.0 – 126.0.0.0     │
  │ Class B │ 10xxxxxx       │ 128.0.0.0 – 191.255.0.0 │
  │ Class C │ 110xxxxx       │ 192.0.0.0 – 223.255.255.0│
  │ Class D │ 1110xxxx       │ 224.0.0.0 – 239.255.255.255│
  │ Class E │ 11110xxx       │ 240.0.0.0 – 255.255.255.255│
  └─────────┴────────────────┴─────────────────────────┘
```

| Class | Range | Purpose | Default Subnet Mask | Hosts per Network |
|-------|-------|---------|---------------------|-------------------|
| **A** | 1.0.0.0 – 126.0.0.0 | Large networks (big organizations) | 255.0.0.0 (/8) | ~16 million |
| **B** | 128.0.0.0 – 191.255.0.0 | Medium networks | 255.255.0.0 (/16) | ~65,534 |
| **C** | 192.0.0.0 – 223.255.255.0 | Small networks (home/office) | 255.255.255.0 (/24) | 254 |
| **D** | 224.0.0.0 – 239.255.255.255 | Multicast only | — | — |
| **E** | 240.0.0.0 – 255.255.255.255 | Experimental/Research | — | — |

> ✅ `0.0.0.0` = special address (default route / unknown)
> ✅ `127.0.0.0/8` = reserved for loopback

---

## 7. Loopback & Special Addresses

```
  Device
  ┌────────────────────────────────┐
  │  App → 127.0.0.1 → Same App   │
  │  (No actual network traffic)   │
  └────────────────────────────────┘
```

| Address | Purpose |
|---------|---------|
| `127.0.0.1` | **Localhost** — device communicates with itself |
| `127.x.x.x` | Entire range loops back to same device |
| `0.0.0.0` | Default route / unknown source address |
| `255.255.255.255` | Limited broadcast (entire local network) |
| `169.254.x.x` | **APIPA** — auto-assigned when DHCP fails |

### APIPA (Automatic Private IP Addressing)

When a device **cannot reach a DHCP server**, it auto-assigns itself an IP from `169.254.0.0/16`.

- Device can communicate locally
- **No internet access**
- Symptom of DHCP failure or network misconfiguration

---

## 8. Network ID vs Host ID

Every IPv4 address contains two parts:

```
  IP: 192.168.1.10

  Class C:
  ┌──────────────────┬───────────┐
  │   Network ID     │  Host ID  │
  │  192 . 168 . 1   │    .10    │
  │  (1st 3 octets)  │(4th octet)│
  └──────────────────┴───────────┘
```

| Class | Network ID | Host ID | Max Hosts |
|-------|-----------|---------|-----------|
| **A** | 1st Octet | 2nd, 3rd, 4th | ~16 million |
| **B** | 1st + 2nd Octet | 3rd, 4th | ~65,534 |
| **C** | 1st + 2nd + 3rd Octet | 4th | 254 |

**Connection Rules:**

| Scenario | Requirement |
|----------|-------------|
| Same Network ID | Devices talk **directly** |
| Different Network ID | **Router required** |

---

## 9. Subnetting

Subnetting divides a large network into **smaller, more manageable sub-networks**.

```
  192.168.1.0/24  (Original /24 — 256 IPs)
  ┌─────────────────────────────────────┐
  │          Subnet the network         │
  └──────────────┬──────────────────────┘
                 │
        ┌────────┴────────┐
        ▼                 ▼
  192.168.1.0/25     192.168.1.128/25
  (126 hosts)        (126 hosts)
  [Public Subnet]    [Private Subnet]
```

### Subnetting Example

```
  Network:      192.168.1.0/24
  Subnet Mask:  255.255.255.0
  Total IPs:    256
  Usable IPs:   254  (first = network, last = broadcast)

  Split into two /25 subnets:
  ┌────────────────────────────────────────────────────────┐
  │ Subnet 1: 192.168.1.0/25                               │
  │   Range:  192.168.1.0 → 192.168.1.127                  │
  │   Usable: 192.168.1.1 → 192.168.1.126  (126 hosts)     │
  ├────────────────────────────────────────────────────────┤
  │ Subnet 2: 192.168.1.128/25                             │
  │   Range:  192.168.1.128 → 192.168.1.255                │
  │   Usable: 192.168.1.129 → 192.168.1.254  (126 hosts)   │
  └────────────────────────────────────────────────────────┘
```

**Benefits of Subnetting:**

| Benefit | Description |
|---------|-------------|
| Security | Isolate departments/environments (public vs private subnet) |
| Performance | Reduce broadcast traffic within each subnet |
| IP Efficiency | Better use of available address space |
| Cloud structure | Public/private subnets in AWS VPC, Azure VNet |

---

## 10. CIDR Reference

**CIDR (Classless Inter-Domain Routing)** replaced class-based networking using **prefix notation** (`/24`, `/16`).

```
  Format: IP/prefix
  Example: 192.168.1.0/24
           └──────────────┘└──┘
               Network      Prefix length
                             (bits used for network)
```

### Complete CIDR Reference Table

| Prefix | Subnet Mask | Total IPs | Usable IPs | Common Use |
|--------|-------------|-----------|------------|------------|
| **/32** | 255.255.255.255 | 1 | 1 | Single host |
| **/31** | 255.255.255.254 | 2 | 2 | Point-to-point links |
| **/30** | 255.255.255.252 | 4 | 2 | Small point-to-point |
| **/29** | 255.255.255.248 | 8 | 6 | Very small subnet |
| **/28** | 255.255.255.240 | 16 | 14 | Small subnet |
| **/27** | 255.255.255.224 | 32 | 30 | Small subnet |
| **/26** | 255.255.255.192 | 64 | 62 | Medium subnet |
| **/25** | 255.255.255.128 | 128 | 126 | Medium subnet |
| **/24** | 255.255.255.0 | 256 | 254 | Typical small network |
| **/23** | 255.255.254.0 | 512 | 510 | |
| **/22** | 255.255.252.0 | 1,024 | 1,022 | |
| **/21** | 255.255.248.0 | 2,048 | 2,046 | |
| **/20** | 255.255.240.0 | 4,096 | 4,094 | |
| **/19** | 255.255.224.0 | 8,192 | 8,190 | |
| **/18** | 255.255.192.0 | 16,384 | 16,382 | |
| **/17** | 255.255.128.0 | 32,768 | 32,766 | |
| **/16** | 255.255.0.0 | 65,536 | 65,534 | Large network |
| **/8** | 255.0.0.0 | 16,777,216 | 16,777,214 | Very large |
| **/0** | 0.0.0.0 | All | All | Default internet route |

> ✅ `0.0.0.0/0` = **all destinations** (default internet route — send everything here)

---

## 11. OSI Model (7 Layers)

The **OSI (Open Systems Interconnection)** model is a conceptual framework that standardizes how different computer systems communicate.

```
  Sender                                    Receiver
  ┌───────────────────┐                  ┌───────────────────┐
  │ L7  Application   │ ◄── Data ──────► │ L7  Application   │
  │ L6  Presentation  │                  │ L6  Presentation  │
  │ L5  Session       │                  │ L5  Session       │
  │ L4  Transport     │ ◄── Segments ──► │ L4  Transport     │
  │ L3  Network       │ ◄── Packets ───► │ L3  Network       │
  │ L2  Data Link     │ ◄── Frames ────► │ L2  Data Link     │
  │ L1  Physical      │ ◄── Bits ──────► │ L1  Physical      │
  └───────────────────┘                  └───────────────────┘
```

| Layer | Name | Protocols / Examples | Data Unit | Device |
|-------|------|----------------------|-----------|--------|
| **L7** | Application | HTTP, HTTPS, FTP, SMTP, DNS, SSH, Telnet | Data | — |
| **L6** | Presentation | SSL/TLS, Encryption, Compression, MIME | Data | — |
| **L5** | Session | Session management, NetBIOS, RPC | Data | — |
| **L4** | Transport | TCP, UDP | Segment | — |
| **L3** | Network | IP, ICMP, ARP, OSPF, BGP | Packet | Router |
| **L2** | Data Link | MAC Addressing, Ethernet, ARP, PPP | Frame | Switch |
| **L1** | Physical | Cables, fiber, radio waves, signals | Bit | Hub |

> 💡 **Memory tip:** **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing

**Layer-wise protocols with ports:**

| Layer | Protocol | Port |
|-------|----------|------|
| L7 | HTTP | 80 |
| L7 | HTTPS | 443 |
| L7 | SMTP | 25 |
| L7 | FTP | 20, 21 |
| L7 | DNS | 53 |
| L7 | POP3 | 110 |
| L7 | IMAP | 143 |
| L6 | SSL/TLS | 443 (for HTTPS) |
| L6 | MIME | (email attachments) |
| L4 | TCP | — |
| L4 | UDP | — |
| L3 | ICMP | — (used by ping) |

---

## 12. TCP/IP Model (4 Layers)

The **TCP/IP model** is what is **actually used** in real-world internet networking.

```
  TCP/IP Model           OSI Equivalent
  ┌──────────────────┐   ┌─────────────────────────┐
  │ Application      │ = │ L7 + L6 + L5             │
  │ Layer            │   │ (App + Present + Session) │
  ├──────────────────┤   ├─────────────────────────┤
  │ Transport        │ = │ L4 (Transport)            │
  │ Layer            │   │ TCP, UDP                  │
  ├──────────────────┤   ├─────────────────────────┤
  │ Internet/Network │ = │ L3 (Network)              │
  │ Layer            │   │ IP, ICMP, ARP             │
  ├──────────────────┤   ├─────────────────────────┤
  │ Network Access   │ = │ L2 + L1                   │
  │ (Link) Layer     │   │ (Data Link + Physical)    │
  └──────────────────┘   └─────────────────────────┘
```

| TCP/IP Layer | Key Protocols | Function |
|-------------|---------------|----------|
| **Application** | HTTP, FTP, DNS, SMTP, SSH | End-user services and interfaces |
| **Transport** | TCP (reliable), UDP (fast) | End-to-end delivery, flow control, error correction |
| **Internet** | IP, ICMP, ARP, BGP | Logical addressing and routing |
| **Network Access** | Ethernet, Wi-Fi, PPP | Physical transmission |

> ✅ Real-world internet runs on TCP/IP. OSI is the teaching/troubleshooting model.

---

## 13. Protocols

A **protocol** is a set of rules that defines how data is **transmitted and received** between devices on a network.

### TCP (Transmission Control Protocol)

- Operates at **Transport Layer (L4)**
- **Connection-oriented** — establishes connection before data transfer
- **Reliable** — guarantees delivery, ordering, and error checking

```
  TCP Three-Way Handshake:

  Client                    Server
    │                         │
    │──── SYN ───────────────►│  "Hello, can we talk?"
    │                         │
    │◄─── SYN-ACK ────────────│  "Yes, I'm ready!"
    │                         │
    │──── ACK ───────────────►│  "Great, let's go!"
    │                         │
    │ ◄══ Connection Open ══► │
```

**Why TCP is Reliable:**

| Mechanism | How it works |
|-----------|-------------|
| **Acknowledgement** | Receiver confirms receipt of each segment |
| **Sequencing** | Data divided into numbered segments; receiver detects missing parts |
| **Checksum** | Detects data corruption during transmission |
| **Retransmission** | Resends lost or corrupted segments automatically |
| **Flow Control** | Prevents sender from overwhelming receiver |

### UDP (User Datagram Protocol)

- **Connectionless** — no handshake, no guaranteed delivery
- **Faster** than TCP due to less overhead
- Used where **speed > reliability**

```
  UDP "fire and forget":

  Client                    Server
    │                         │
    │──── Data Packet ───────►│  (no ACK)
    │──── Data Packet ───────►│  (no ACK)
    │──── Data Packet ───────►│  (no ACK)
```

### ICMP (Internet Control Message Protocol)

- Used for **network diagnostics**
- Not a transport protocol — carries control messages
- Used by `ping` and `traceroute`

### ARP (Address Resolution Protocol)

- Maps an **IP address** → **MAC address**
- Used at **Data Link Layer (L2)**
- Broadcasts "Who has this IP?" on local network

```
  ARP Process:
  Device A (192.168.1.1) wants to talk to Device B (192.168.1.2)
    │
    ├── ARP Broadcast: "Who has 192.168.1.2?"
    │
    └── Device B replies: "I have 192.168.1.2, my MAC is 00:1A:2B:3C:4D:5E"
    │
    └── Device A caches this MAC-to-IP mapping
```

---

## 14. Ports & Protocol Reference

A **port** identifies a specific service running on a device.

```
  IP Address = The building address
  Port       = The room/department within that building

  Example:  192.168.1.10 : 443
            └──────────────┘└───┘
                Server IP    HTTPS service
```

**Port Ranges:**

| Range | Type | Description |
|-------|------|-------------|
| 0–1023 | **Well-Known Ports** | Reserved for standard services (root required) |
| 1024–49151 | **Registered Ports** | Application-specific, registered with IANA |
| 49152–65535 | **Dynamic/Ephemeral** | Temporary client-side ports |

### Complete Ports Cheat Sheet (70+ Ports)

#### 🌐 Web Services

| Port | Protocol | Service | Notes |
|------|----------|---------|-------|
| 80 | TCP | **HTTP** | Standard web traffic |
| 443 | TCP | **HTTPS** | Secure web (TLS) |
| 8080 | TCP | HTTP Alternative | Dev/proxy servers |
| 8443 | TCP | HTTPS Alternative | Dev/alternate HTTPS |
| 3000 | TCP | HTTP Development | Node.js, React dev |
| 8000 | TCP | HTTP Development | Django, Python dev |

#### 📧 Email

| Port | Protocol | Service | Notes |
|------|----------|---------|-------|
| 25 | TCP | **SMTP** | Send email (server-to-server) |
| 465 | TCP | SMTPS | SMTP over SSL |
| 587 | TCP | SMTP Submission | Send email (client auth) |
| 110 | TCP | **POP3** | Retrieve email (download & delete) |
| 995 | TCP | POP3S | POP3 over SSL |
| 143 | TCP | **IMAP** | Retrieve email (sync) |
| 993 | TCP | IMAPS | IMAP over SSL |

#### 🔐 Remote Access & Security

| Port | Protocol | Service | Notes |
|------|----------|---------|-------|
| 22 | TCP | **SSH** | Secure remote login |
| 23 | TCP | **Telnet** | Remote login (unencrypted — avoid!) |
| 3389 | TCP | **RDP** | Windows Remote Desktop |
| 5900 | TCP | VNC | Remote desktop (non-Windows) |

#### 📁 File Transfer

| Port | Protocol | Service | Notes |
|------|----------|---------|-------|
| 20 | TCP | **FTP Data** | File data transfer |
| 21 | TCP | **FTP Control** | FTP connection control |
| 22 | TCP | **SFTP** | Secure file transfer (over SSH) |
| 69 | UDP | **TFTP** | Trivial FTP (firmware, boot) |
| 990 | TCP | FTPS | FTP over SSL |

#### 🌍 Network Services

| Port | Protocol | Service | Notes |
|------|----------|---------|-------|
| 53 | UDP/TCP | **DNS** | Domain name resolution |
| 67 | UDP | DHCP Server | Server receives client requests |
| 68 | UDP | DHCP Client | Client receives IP from server |
| 123 | UDP | **NTP** | Network Time Protocol |
| 161 | UDP | **SNMP** | Simple Network Management Protocol |
| 162 | UDP | SNMP Trap | SNMP notifications |
| 179 | TCP | BGP | Border Gateway Protocol (routing) |

#### 🗂️ Directory Services

| Port | Protocol | Service | Notes |
|------|----------|---------|-------|
| 389 | TCP/UDP | **LDAP** | Directory services (Active Directory) |
| 636 | TCP | **LDAPS** | LDAP over SSL |
| 88 | TCP/UDP | Kerberos | Authentication protocol |
| 445 | TCP | SMB | Windows file sharing |
| 137–139 | TCP/UDP | NetBIOS | Windows network browsing |

#### 🗄️ Database Ports

| Port | Protocol | Database | Notes |
|------|----------|---------|-------|
| 3306 | TCP | **MySQL / MariaDB** | Default MySQL |
| 5432 | TCP | **PostgreSQL** | Default Postgres |
| 1521 | TCP | Oracle DB | |
| 1433 | TCP | MSSQL | Microsoft SQL Server |
| 27017 | TCP | **MongoDB** | Default MongoDB |
| 6379 | TCP | **Redis** | In-memory data store |
| 9200 | TCP | Elasticsearch | HTTP API |
| 9300 | TCP | Elasticsearch | Node communication |
| 2181 | TCP | Zookeeper | Cluster coordination |
| 9092 | TCP | Kafka | Message broker |

#### ☸️ Container & Orchestration

| Port | Protocol | Service | Notes |
|------|----------|---------|-------|
| 2375 | TCP | Docker Daemon | Without TLS (insecure) |
| 2376 | TCP | Docker Daemon | With TLS (secure) |
| 6443 | TCP | **Kubernetes API** | kube-apiserver |
| 10250 | TCP | Kubelet | Node agent API |
| 10255 | TCP | Kubelet (read-only) | Read-only metrics |
| 30000–32767 | TCP | **NodePort Range** | K8s exposed services |
| 2379–2380 | TCP | etcd | K8s cluster state store |

#### 📊 Monitoring & Logging

| Port | Protocol | Service | Notes |
|------|----------|---------|-------|
| 9090 | TCP | **Prometheus** | Metrics collection |
| 3000 | TCP | **Grafana** | Dashboards |
| 5601 | TCP | Kibana | Elasticsearch dashboard |
| 5044 | TCP | Logstash | Log ingest (Beats) |
| 9411 | TCP | Zipkin | Distributed tracing |
| 14268 | TCP | Jaeger | Distributed tracing |

#### 🔒 VPN & Tunneling

| Port | Protocol | Service | Notes |
|------|----------|---------|-------|
| 500 | UDP | IKEv2 / IPsec | VPN key exchange |
| 4500 | UDP | IPsec NAT-T | VPN through NAT |
| 1194 | UDP | OpenVPN | Open source VPN |
| 1723 | TCP | PPTP | VPN (older, less secure) |
| 51820 | UDP | WireGuard | Modern fast VPN |

---

## 15. HTTP Basics

HTTP is a **client-server protocol** for web communication.

```
  Client (Browser)          Server
       │                       │
       │──── HTTP Request ─────►│
       │     GET /index.html    │
       │     Host: example.com  │
       │                       │
       │◄─── HTTP Response ─────│
       │     200 OK             │
       │     Content-Type: html │
       │     <html>...</html>   │
```

**Key Properties:**
- **Stateless** — server does not store client session between requests
- Works over TCP (HTTP on port 80, HTTPS on port 443)
- **HTTPS** = HTTP + TLS encryption

### HTTP Methods

| Method | Purpose | Idempotent? |
|--------|---------|-------------|
| `GET` | Fetch/retrieve data | ✅ Yes |
| `POST` | Send data, create resource | ❌ No |
| `PUT` | Replace/update entire resource | ✅ Yes |
| `PATCH` | Partial update | ❌ No |
| `DELETE` | Remove resource | ✅ Yes |
| `HEAD` | Like GET but no body (headers only) | ✅ Yes |
| `OPTIONS` | List allowed methods | ✅ Yes |

### HTTP Status Codes

```
  1xx — Informational   (request received, processing)
  2xx — Success         (request completed)
  3xx — Redirect        (resource moved)
  4xx — Client Error    (bad request from client)
  5xx — Server Error    (server failed)
```

| Code | Meaning | Common Cause |
|------|---------|-------------|
| **200** | OK | Request succeeded |
| **201** | Created | Resource created (POST) |
| **204** | No Content | Success, no body returned |
| **301** | Moved Permanently | URL changed forever |
| **302** | Found (Temp Redirect) | Temporary URL change |
| **304** | Not Modified | Client cache is valid |
| **400** | Bad Request | Malformed request syntax |
| **401** | Unauthorized | Authentication required |
| **403** | Forbidden | Authenticated but no permission |
| **404** | Not Found | Resource doesn't exist |
| **405** | Method Not Allowed | HTTP method not supported |
| **429** | Too Many Requests | Rate limit hit |
| **500** | Internal Server Error | General server crash |
| **502** | Bad Gateway | Proxy received bad response upstream |
| **503** | Service Unavailable | Server overloaded or down |
| **504** | Gateway Timeout | Upstream server timeout |

### Cookies & Sessions (Why login stays active)

Even though HTTP is **stateless**, login persists due to:

```
  Browser                      Server
     │                            │
     │── POST /login ────────────►│
     │   (username + password)    │
     │                            │
     │◄── Set-Cookie: token=abc ──│
     │    (stored in browser)     │
     │                            │
     │── GET /dashboard ─────────►│
     │   Cookie: token=abc        │
     │                            │
     │◄── 200 OK + User Data ─────│
```

**Flow:** Cookies → Sessions → JWT Tokens  
Cookies store identifiers in the browser so the server can recognize returning users.

---

## 16. TCP vs UDP

Both operate at **Layer 4 (Transport Layer)**.

```
  TCP — Reliable (Connection-Oriented)    UDP — Fast (Connectionless)
  ┌─────────────────────────────┐         ┌─────────────────────────┐
  │ Client ──SYN──► Server      │         │ Client ──Data──► Server │
  │ Client ◄─SYN-ACK── Server  │         │ Client ──Data──► Server │
  │ Client ──ACK──► Server      │         │ (No acknowledgement)    │
  │ ═══ DATA FLOW ═══           │         │ ═══ DATA FLOW ═══       │
  │ Client ◄─ACK── Server (ack) │         │ (Fire and forget)       │
  └─────────────────────────────┘         └─────────────────────────┘
```

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented | Connectionless |
| Reliability | ✅ Reliable (ACK + retransmit) | ❌ No guarantee |
| Speed | Slower (overhead) | Faster (no overhead) |
| Order | In-order delivery | No order guarantee |
| Header Size | 20 bytes minimum | 8 bytes |
| Flow Control | ✅ Yes | ❌ No |
| Error Checking | ✅ Checksum + retransmit | Checksum only |
| Protocol Number | 6 | 17 |
| Use Cases | HTTP, SSH, FTP, databases | DNS, Video, Gaming, VoIP |

**When to use TCP:**
- File download/upload (FTP, SFTP)
- Web browsing (HTTP/HTTPS)
- Database connections
- Email (SMTP, IMAP)
- SSH remote access

**When to use UDP:**
- DNS queries (speed critical)
- Video/audio streaming
- Online gaming
- VoIP calls
- DHCP, NTP, SNMP

---

## 17. Routing

Routing decides how **packets travel from source to destination** using a **route table**.

```
  Packet: Source 10.0.1.5 → Destination 8.8.8.8

  Route Table:
  ┌───────────────┬──────────────────┬──────────────────────────┐
  │ Destination   │ Next Hop         │ Action                   │
  ├───────────────┼──────────────────┼──────────────────────────┤
  │ 10.0.1.0/24   │ local            │ Direct (same subnet)     │
  │ 10.0.2.0/24   │ 10.0.1.1        │ Via internal router      │
  │ 0.0.0.0/0     │ Internet Gateway │ Default — goes to internet│
  └───────────────┴──────────────────┴──────────────────────────┘

  Packet to 8.8.8.8 → matches 0.0.0.0/0 → sent to Internet Gateway
```

**Routing Rule Priority:**
1. **Most specific route wins** (longest prefix match)
2. `10.21.0.0/16` is evaluated before `10.0.0.0/8`
3. `0.0.0.0/0` matches **everything else** (catch-all)

| Default Route | Meaning |
|--------------|---------|
| `0.0.0.0/0` | Send all unmatched traffic to internet gateway / NAT / firewall |

---

## 18. DNS

**DNS (Domain Name System)** converts human-readable domain names to IP addresses.

```
  User types: www.example.com
       │
       ▼
  ┌─────────────────────────────────────────────────────────┐
  │                    DNS Resolution Flow                  │
  │                                                         │
  │  Browser → Resolver DNS → (Cache Hit?) → Return IP     │
  │                │                                        │
  │                └─ Cache Miss? ──►  Root DNS Server      │
  │                                        │                │
  │                                   TLD DNS (.com)        │
  │                                        │                │
  │                               Authoritative DNS         │
  │                                        │                │
  │                              Returns IP: 93.184.216.34  │
  └─────────────────────────────────────────────────────────┘
       │
       ▼
  Browser connects to 93.184.216.34
```

**How DNS Works (Step by Step):**

| Step | Action |
|------|--------|
| 1 | User types domain in browser |
| 2 | Query goes to **Resolver DNS** (usually ISP or 8.8.8.8) |
| 3 | Resolver checks **local cache** |
| 4 | If not cached, asks **Root DNS Server** |
| 5 | Root redirects to **TLD DNS** (`.com`, `.org`) |
| 6 | TLD points to **Authoritative DNS** |
| 7 | Authoritative DNS returns the IP address |
| 8 | Browser connects to the IP |

### DNS Record Types

| Record | Full Name | Use | Example |
|--------|-----------|-----|---------|
| **A** | Address | Domain → IPv4 | `example.com → 93.184.216.34` |
| **AAAA** | IPv6 Address | Domain → IPv6 | `example.com → 2001:db8::1` |
| **CNAME** | Canonical Name | Alias one name to another | `www → example.com` |
| **MX** | Mail Exchange | Email server routing | `@ → mail.example.com` |
| **NS** | Name Server | DNS delegation | `example.com → ns1.dns.com` |
| **TXT** | Text | Verification, SPF, DKIM | `v=spf1 include:...` |
| **PTR** | Pointer | Reverse DNS (IP → Domain) | `34.216.184.93 → example.com` |
| **SRV** | Service | Service location | `_http._tcp.example.com` |
| **SOA** | Start of Authority | Zone info, serial number | Primary NS + admin email |
| **CAA** | Certificate Authority | Which CAs can issue certs | `0 issue "letsencrypt.org"` |

### PING

**PING** uses **ICMP** to test reachability of a host on an IP network.

```bash
$ ping google.com
PING google.com (142.250.195.46): 56 data bytes
64 bytes from 142.250.195.46: icmp_seq=0 ttl=117 time=12.4 ms
64 bytes from 142.250.195.46: icmp_seq=1 ttl=117 time=11.8 ms

--- google.com ping statistics ---
2 packets transmitted, 2 packets received, 0.0% packet loss
round-trip min/avg/max = 11.8/12.1/12.4 ms
```

---

## 19. DHCP

**DHCP (Dynamic Host Configuration Protocol)** automatically assigns network configuration to devices.

```
  DHCP DORA Process:

  Device                          DHCP Server
    │                                  │
    │─── DISCOVER (broadcast) ────────►│  "Anyone have an IP for me?"
    │                                  │
    │◄── OFFER ───────────────────────│  "I can offer you 192.168.1.50"
    │                                  │
    │─── REQUEST ────────────────────►│  "I'd like 192.168.1.50 please"
    │                                  │
    │◄── ACKNOWLEDGE ─────────────────│  "It's yours! Lease: 24 hours"
    │                                  │
    │ IP: 192.168.1.50                 │
    │ Mask: 255.255.255.0              │
    │ Gateway: 192.168.1.1             │
    │ DNS: 8.8.8.8                     │
```

**DORA Process:**

| Step | Name | Description |
|------|------|-------------|
| **D** | **Discover** | Client broadcasts to find DHCP servers |
| **O** | **Offer** | Server responds with available IP |
| **R** | **Request** | Client formally accepts the offered IP |
| **A** | **Acknowledge** | Server confirms IP assignment and lease time |

**What DHCP Assigns:**

| Parameter | Example |
|-----------|---------|
| IP Address | `192.168.1.50` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.1.1` |
| DNS Servers | `8.8.8.8`, `8.8.4.4` |
| Lease Time | `86400` seconds (24 hours) |

**Why Use DHCP?**

| Reason | Benefit |
|--------|---------|
| Automation | No manual IP assignment needed |
| Error Prevention | Avoids duplicate IP addresses |
| Time-Saving | Faster network setup |
| Central Management | IT manages all IPs from one place |

---

## 20. Network Security & Services

### 🔥 Firewalls

```
  Internet                    Internal Network
  ──────── ►  [FIREWALL]  ► ────────────────
               │
               │ Rules:
               │ ✅ Allow HTTPS (443) from any
               │ ✅ Allow SSH (22) from office IPs only
               │ ❌ Block port 3306 from internet
               │ ❌ Block all other inbound
```

| Firewall Function | Description |
|-------------------|-------------|
| Traffic Filtering | Only allows legitimate traffic based on rules |
| Network Segmentation | Isolates management network from user network |
| Protection | Shields internal network from external threats |
| Logging | Records connection attempts for audit |

**Examples:**
- Allow SSH only from office IPs
- Block public access to internal databases
- AWS Security Groups are cloud-based firewalls

### ⚖️ Load Balancer

```
  Client Requests
         │
         ▼
  [Load Balancer]
  ┌──────┴──────┐
  │      │      │
  ▼      ▼      ▼
 App1   App2   App3
(servers)
```

| Benefit | Description |
|---------|-------------|
| High Availability | If one server fails, others handle traffic |
| Prevents Overload | Distributes load across multiple servers |
| Autoscaling | Supports adding/removing servers automatically |
| Blue-Green Deploys | Route traffic between old and new versions |

**Load Balancing Algorithms:**

| Algorithm | How It Works |
|-----------|-------------|
| Round Robin | Each request goes to the next server in rotation |
| Least Connections | Send to server with fewest active connections |
| IP Hash | Same client always goes to same server |
| Weighted | Servers get traffic proportional to their capacity |

### 🔒 VPN (Virtual Private Network)

```
  Remote User ──[Encrypted Tunnel]──► Private Network
  (Home/Coffee shop)                   (Office/Cloud)
```

**VPN Uses:**

| Use Case | Description |
|----------|-------------|
| Secure Remote Access | Employees work from home securely |
| Office Connectivity | Connect branch offices |
| Private Cloud Access | Access cloud resources privately |

**VPN Protocols:**

| Protocol | Port | Notes |
|----------|------|-------|
| WireGuard | UDP 51820 | Fastest, modern |
| OpenVPN | UDP 1194 | Open source, flexible |
| IPsec/IKEv2 | UDP 500, 4500 | Enterprise standard |
| PPTP | TCP 1723 | Old, less secure |

### 🏷️ VLAN (Virtual Local Area Network)

```
  Physical Network (1 switch)
  ┌──────────────────────────────────────┐
  │                                      │
  │  VLAN 10: HR Dept    VLAN 20: Dev    │
  │  ┌────────────┐      ┌────────────┐  │
  │  │ PC1  PC2   │      │ PC3  PC4   │  │
  │  └────────────┘      └────────────┘  │
  │  (Isolated from each other)          │
  └──────────────────────────────────────┘
```

| VLAN Type | Description |
|-----------|-------------|
| **Access Port** | Belongs to one VLAN; device is unaware of VLAN tagging |
| **Trunk Port** | Carries traffic for multiple VLANs (1–4094); connects switches/routers |

**Why Use VLANs:**

| Reason | Benefit |
|--------|---------|
| Security | Isolate sensitive departments (HR, Finance) |
| Scalability | Group devices regardless of physical location |
| Broadcast Control | Reduce broadcast traffic |
| Traffic Management | Prioritize critical traffic |

### 🔄 NAT (Network Address Translation)

```
  Private Network           Internet
  ┌─────────────────┐      ┌──────────────┐
  │ 192.168.1.10   ─┼──────►              │
  │ 192.168.1.11   ─┼──────► [NAT Router] ──► Internet
  │ 192.168.1.12   ─┼──────►              │   (One Public IP)
  └─────────────────┘      └──────────────┘
  (Private IPs)              (Public IP)
```

| NAT Type | Description |
|----------|-------------|
| **Static NAT** | One private IP ↔ one public IP (1:1) |
| **Dynamic NAT** | Pool of private IPs ↔ pool of public IPs |
| **PAT / NAT Overload** | Many private IPs → one public IP using different ports |

> ✅ PAT (Port Address Translation) is most common in home routers — your entire home uses one public IP.

---

## 21. Network Troubleshooting Tools

### Quick Reference

| Tool | Protocol | Use Case | Key Command |
|------|----------|---------|-------------|
| `ping` | ICMP | Check connectivity + latency | `ping google.com` |
| `traceroute` | ICMP/UDP | Show path + identify slow hops | `traceroute google.com` |
| `telnet` | TCP | Check if port is reachable | `telnet host 443` |
| `curl` | HTTP/S | Test HTTP endpoints + headers | `curl -I https://example.com` |
| `dig` | DNS | DNS lookups + troubleshooting | `dig google.com A` |
| `nslookup` | DNS | DNS query (simpler than dig) | `nslookup google.com` |
| `netstat` | — | Active connections + ports | `netstat -tulnp` |
| `ss` | — | Socket statistics (netstat replacement) | `ss -tulnp` |
| `nmap` | — | Port scanning + host discovery | `nmap -sn 192.168.1.0/24` |
| `ssh` | SSH | Secure remote login | `ssh user@server-ip` |
| `scp` | SSH | Secure file copy | `scp file.txt user@host:/tmp/` |
| `tcpdump` | — | Capture + analyze packets | `tcpdump -i eth0` |
| `wireshark` | — | GUI packet analyzer | (GUI tool) |
| `iperf` | TCP/UDP | Measure network bandwidth | `iperf3 -s / -c server` |

### Detailed Usage

```bash
# 1. PING — Check connectivity
ping google.com
ping -c 4 8.8.8.8           # Send only 4 packets
ping -i 0.5 google.com      # Ping every 0.5 seconds

# 2. TRACEROUTE — Show route path
traceroute google.com
traceroute -n google.com    # No DNS resolution (faster)

# 3. TELNET — Test port connectivity
telnet example.com 443      # Test HTTPS port
telnet example.com 22       # Test SSH port

# 4. CURL — Test HTTP endpoints
curl http://example.com
curl -I https://example.com         # Headers only
curl -X POST https://api.example.com/data -d '{"key":"value"}'
curl -o output.html http://example.com  # Save to file

# 5. DIG — DNS troubleshooting
dig google.com              # Full DNS lookup
dig google.com A            # IPv4 record only
dig google.com AAAA         # IPv6 record only
dig google.com MX           # Mail server records
dig @8.8.8.8 google.com     # Query specific DNS server
dig +short google.com       # Just the IP

# 6. NETSTAT — Active connections
netstat -tulnp              # All listening ports (TCP+UDP)
netstat -r                  # Routing table

# 7. SS — Modern netstat
ss -tulnp                   # Listening ports
ss -s                       # Summary statistics

# 8. NMAP — Port scanning (use with permission!)
nmap -sn 192.168.1.0/24     # Discover live hosts
nmap -p 80,443,22 host      # Scan specific ports
nmap -A 192.168.1.10        # Aggressive scan

# 9. SSH — Remote login
ssh user@server-ip
ssh -i key.pem user@server  # With key file
ssh -p 2222 user@server     # Custom port

# 10. SCP — Secure copy
scp file.txt user@server:/tmp/
scp user@server:/tmp/file.txt .   # Download
scp -r folder/ user@server:/tmp/ # Copy directory

# 11. TCPDUMP — Packet capture
tcpdump -i eth0             # Capture on interface
tcpdump -i eth0 port 80     # Capture HTTP traffic
tcpdump -w capture.pcap     # Save to file

# 12. IPERF — Bandwidth test
iperf3 -s                   # Start server
iperf3 -c server-ip         # Run client test
```

---

## 22. Cloud Networking Concepts

### VPC (Virtual Private Cloud)

```
  AWS VPC: 10.0.0.0/16
  ┌────────────────────────────────────────────────┐
  │                  VPC                           │
  │  ┌──────────────────┐  ┌───────────────────┐  │
  │  │  Public Subnet   │  │  Private Subnet   │  │
  │  │  10.0.1.0/24     │  │  10.0.2.0/24      │  │
  │  │  [Web Server]    │  │  [Database]       │  │
  │  └────────┬─────────┘  └───────────────────┘  │
  │           │ Internet Gateway                   │
  └───────────┼────────────────────────────────────┘
              │
           Internet
```

| Component | Function |
|-----------|----------|
| **VPC** | Isolated virtual network |
| **Public Subnet** | Has route to internet via Internet Gateway |
| **Private Subnet** | No direct internet access |
| **Internet Gateway** | Allows public subnet to reach internet |
| **NAT Gateway** | Allows private subnet to initiate internet traffic |
| **Route Table** | Controls where subnet traffic goes |
| **Security Group** | Instance-level firewall (stateful) |
| **NACL** | Subnet-level firewall (stateless) |

### CDN (Content Delivery Network)

```
  Origin Server (US)
        │
        ├──► Edge Node (Europe)  ◄─── EU Users
        ├──► Edge Node (Asia)    ◄─── Asian Users
        └──► Edge Node (AU)      ◄─── Australian Users
```

CDN caches content at edge locations closer to users, reducing:
- Latency
- Load on origin server
- Bandwidth costs

### Infrastructure as Code (IaC)

```
  Developer writes code → CI/CD Pipeline → Infrastructure provisioned
  ┌──────────────────┐   ┌─────────────┐   ┌──────────────────────┐
  │  Terraform .tf   │──►│   Plan      │──►│  AWS VPC, EC2, RDS   │
  │  Ansible YAML    │   │   Apply     │   │  K8s cluster         │
  │  Pulumi code     │   │   Destroy   │   │  Load balancers      │
  └──────────────────┘   └─────────────┘   └──────────────────────┘
```

### Service Discovery

In microservices/containers, services need to find each other dynamically.

| Tool | Use Case |
|------|---------|
| **Consul** | Service registry + health checking |
| **Kubernetes DNS** | Internal service discovery in K8s |
| **AWS Route 53** | DNS-based service discovery |
| **Istio/Envoy** | Service mesh with discovery |

### SDN (Software Defined Networking)

- Decouples **network control plane** from **data forwarding plane**
- Makes network configuration programmable
- Enables dynamic, cloud-native infrastructure
- Used in: OpenFlow, Kubernetes CNI plugins, cloud provider virtual networks

---

## 23. Additional Concepts

### ARP (Address Resolution Protocol)

```
  Device A wants to reach Device B on same LAN:
  Device A: "Who has 192.168.1.2? Tell 192.168.1.1"  (ARP Broadcast)
  Device B: "192.168.1.2 is at MAC 00:1A:2B:3C:4D:5E" (ARP Reply)
  Device A: caches this MAC → IP mapping
```

### MAC Address

```
  00 : 1A : 2B : 3C : 4D : 5E
  ──────────   ────────────────
  Manufacturer   Device Serial Number
  (OUI - 3 bytes)   (3 bytes)

  Total = 48 bits = 6 bytes
```

| Property | Value |
|----------|-------|
| Length | 48 bits (6 bytes) |
| Format | Hex: `00:1A:2B:3C:4D:5E` |
| First 3 bytes | OUI (manufacturer) |
| Last 3 bytes | Device serial |
| Scope | Layer 2 (local network only) |

### Network Performance Metrics

| Metric | Description | Unit |
|--------|-------------|------|
| **Bandwidth** | Maximum data transfer rate | Mbps, Gbps |
| **Latency** | Time for signal source → destination | ms |
| **RTT (Round Trip Time)** | Latency × 2 (there + back) | ms |
| **Throughput** | Actual data transferred | Mbps |
| **Jitter** | Variation in latency | ms |
| **Packet Loss** | % of packets that don't arrive | % |
| **Load** | Amount of data on network at a time | % utilization |
| **MTU** | Maximum Transmission Unit (largest packet) | bytes (default: 1500) |

### Network Topology Types

```
  Bus:         Star:          Ring:
  ─────────    ───[HUB]───    ─ A ─ B ─
  A  B  C  D   A   B   C  D    |       |
                              D ─────── C

  Mesh:        Tree:          Hybrid:
  A─B─C        [Root]         Mix of
  │ │ │        ┌─┴─┐          above
  D─E─F       [A] [B]         topologies
                ├─┤  ├─┤
               C  D  E  F
```

| Topology | Pros | Cons | Best For |
|----------|------|------|----------|
| **Bus** | Simple, cheap | Single point of failure | Small/temp networks |
| **Star** | Easy to manage | Hub failure = down | Office networks |
| **Ring** | Simple flow | One failure breaks loop | Token Ring (legacy) |
| **Mesh** | Highly fault-tolerant | Expensive, complex | Critical infrastructure |
| **Tree** | Scalable | Main bus failure impacts all | Large campus |
| **Hybrid** | Flexible | Complex and expensive | Enterprise networks |

### Communication Types

| Type | Description | Example |
|------|-------------|---------|
| **Unicast** | One-to-one | Visiting a website |
| **Broadcast** | One-to-all on network | ARP request |
| **Multicast** | One-to-group | Video streaming group |
| **Anycast** | One-to-nearest | DNS, CDN (IPv6) |

### CRC (Cyclic Redundancy Check)

- **Error detection** technique at Data Link Layer
- Sender calculates CRC value and appends to frame
- Receiver recalculates and compares — mismatch = corruption
- More robust than simple checksums

### Encryption in Networking

| Type | Description | Use Case |
|------|-------------|---------|
| **Symmetric** | Same key for encrypt + decrypt | Fast, used in bulk data (AES) |
| **Asymmetric** | Public + private key pair | TLS handshake, SSH, SSL |
| **TLS/SSL** | Transport Layer Security | HTTPS, SMTPS, IMAPS |
| **IPsec** | Network layer encryption | VPN tunnels |
| **WireGuard** | Modern VPN protocol | Fast, lightweight VPN |

### IDS/IPS (Intrusion Detection/Prevention)

| Feature | IDS | IPS |
|---------|-----|-----|
| Full Form | Intrusion Detection System | Intrusion Prevention System |
| Action | Detects and alerts | Detects and blocks |
| Placement | Out-of-band (passive) | Inline (active) |
| Risk | False positive = missed alert | False positive = blocked traffic |

---

## 24. IPv4 vs IPv6 Deep Dive

| Feature | IPv4 | IPv6 |
|---------|------|------|
| **Address Length** | 32 bits (4 octets) | 128 bits (8 groups) |
| **Format** | Decimal, dots: `192.168.0.1` | Hex, colons: `2001:db8::1` |
| **Total Addresses** | ~4.3 billion | ~340 undecillion |
| **NAT Required** | Yes (address exhaustion) | No (enough addresses) |
| **Security** | External IPsec (optional) | Built-in IPsec |
| **Broadcast** | Supported | Not supported |
| **Multicast** | Supported | Enhanced multicast |
| **Anycast** | Limited | Native support |
| **Configuration** | Manual or DHCP | Auto (SLAAC) or DHCPv6 |
| **Header Size** | 20 bytes (variable) | 40 bytes (fixed, simpler) |
| **Fragmentation** | By routers and hosts | Only by sender |
| **Checksum** | In header | Removed (handled by L4) |
| **Routing** | Classful/CIDR | More efficient with hierarchy |

**IPv6 Advantages:**
1. **Massive address space** — solves IPv4 exhaustion
2. **Built-in security** — IPsec mandatory in spec
3. **Simplified header** — faster router processing
4. **No NAT needed** — end-to-end connectivity
5. **Auto-configuration** — SLAAC for automatic IP
6. **Anycast support** — route to nearest node

---

## 25. Quick Reference Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════╗
║           NETWORKING FOR DEVOPS — QUICK REFERENCE               ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  FLOW:  Network → IP → Subnet → Gateway → DNS → Internet        ║
║                                                                  ║
║  OSI:   L7-App | L6-Pres | L5-Sess | L4-Trans |                 ║
║         L3-Net | L2-DLink | L1-Phys                             ║
║                                                                  ║
║  TCP/IP: Application | Transport | Internet | Network Access    ║
║                                                                  ║
║  PRIVATE IPs:  10.x.x.x | 172.16-31.x.x | 192.168.x.x         ║
║  LOOPBACK:     127.0.0.1  (localhost)                           ║
║  APIPA:        169.254.x.x  (no DHCP available)                 ║
║  DEFAULT ROUTE: 0.0.0.0/0  (internet)                           ║
║                                                                  ║
║  KEY PORTS:                                                      ║
║    HTTP=80    HTTPS=443    SSH=22     DNS=53                     ║
║    FTP=21     SMTP=25      MySQL=3306 K8s=6443                  ║
║    Redis=6379 Postgres=5432 MongoDB=27017                       ║
║    Prometheus=9090 Grafana=3000                                  ║
║                                                                  ║
║  CIDR:  /32=1 | /24=256 | /16=65536 | /8=16M | /0=all          ║
║                                                                  ║
║  TCP = Reliable (SYN → SYN-ACK → ACK → Data + ACKs)            ║
║  UDP = Fast, no guarantee (fire and forget)                      ║
║                                                                  ║
║  DNS DORA: User→Resolver→Root→TLD→Authoritative→IP→Browser     ║
║  DHCP DORA: Discover → Offer → Request → Acknowledge            ║
║                                                                  ║
║  NAT Types: Static (1:1) | Dynamic (pool) | PAT (many:1)        ║
║  VLAN: Access Port (1 VLAN) | Trunk Port (multiple VLANs)       ║
║                                                                  ║
║  TROUBLESHOOTING:                                                ║
║    ping (reachability) | traceroute (path) | dig (DNS)          ║
║    curl (HTTP) | netstat/ss (ports) | nmap (scanning)           ║
╚══════════════════════════════════════════════════════════════════╝
```

### Interview Questions Summary

| Topic | Key Points to Mention |
|-------|----------------------|
| OSI vs TCP/IP | OSI=7 layers (theoretical), TCP/IP=4 layers (practical) |
| TCP vs UDP | TCP=reliable+handshake, UDP=fast+connectionless |
| DNS | Phonebook of internet, DORA flow, A/AAAA/CNAME/MX records |
| DHCP | Auto IP assignment, DORA process |
| Subnetting | Divide networks, /24=256 IPs, 2 reserved (network+broadcast) |
| NAT | Private → Public IP translation, PAT most common |
| VPN | Encrypted tunnel, WireGuard/OpenVPN/IPsec |
| VLAN | Logical segmentation, access vs trunk ports |
| Firewall | Traffic filtering, Security Groups in cloud |
| Load Balancer | Distribute traffic, HA, autoscaling support |
| IPv6 | 128-bit, built-in IPsec, no NAT needed, SLAAC |
| Routing | Longest prefix match wins, 0.0.0.0/0 = default route |

---

*Notes compiled and enhanced from: Networking for DevOps — Beginner → Advanced*
*Sources: Images (OCR), existing notes document, additional DevOps networking knowledge*
*Topics: OSI · TCP/IP · Ports (70+) · Subnetting · DNS · DHCP · VPN · NAT · VLAN · Security · Cloud Networking*
