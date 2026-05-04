# 🌐 Networking for DevOps — Complete Notes
> Clean, structured, interview + real-world ready

---

## Table of Contents
1. [What is a Network?](#1-what-is-a-network)
2. [Types of Networks](#2-types-of-networks)
3. [Network Components](#3-network-components)
4. [Protocols](#4-protocols)
5. [IP Address](#5-ip-address)
6. [Types of IP Addresses](#6-types-of-ip-addresses)
7. [IPv4 Classes](#7-ipv4-classes)
8. [Loopback Address (Localhost)](#8-loopback-address-localhost)
9. [Network ID vs Host ID](#9-network-id-vs-host-id)
10. [Subnetting](#10-subnetting)
11. [CIDR (Classless Inter-Domain Routing)](#11-cidr)
12. [Network Models — OSI & TCP/IP](#12-network-models)
13. [Ports and Protocols](#13-ports-and-protocols)
14. [HTTP Basics](#14-http-basics)
15. [TCP vs UDP](#15-tcp-vs-udp)
16. [Routing](#16-routing)
17. [DNS (Domain Name System)](#17-dns)
18. [DHCP](#18-dhcp)
19. [Network Security & Services](#19-network-security--services)
20. [Network Troubleshooting Tools](#20-network-troubleshooting-tools)
21. [Additional Concepts](#21-additional-concepts)

---

## 1. What is a Network?

A **network** is formed when **two or more computers/devices** are connected through **wired or wireless communication** so they can **share data and resources**.

**Why Networks are Used:**
- Enable **communication** between devices (messages, requests, data)
- Share **resources** like printers, storage, servers, and internet connections
- Support **centralized management** of systems
- Allow **remote access** and collaboration

> ✅ The **Internet** is the world's largest network — often called the **"network of networks"**.

---

## 2. Types of Networks

Networks are categorized based on size and coverage.

| Type | Full Form | Description | Example |
|------|-----------|-------------|---------|
| **LAN** | Local Area Network | Small area — home, office, school | Wi-Fi, Ethernet |
| **MAN** | Metropolitan Area Network | Spans a city or large campus | City-wide ISP networks |
| **WAN** | Wide Area Network | Connects multiple LANs across long distances | Internet, enterprise global networks |
| **SONET** | Synchronous Optical Network | High-speed optical transmission, long-distance | Undersea internet cables |

---

## 3. Network Components

A DevOps engineer must understand basic networking hardware because cloud networking works on the same principles.

### 1. Switch
- A **Layer 2** device that connects devices **inside the same network**
- Forwards traffic using **MAC addresses**
- Smarter than a hub — sends data only to the intended device
- Used inside LAN networks

### 2. Router
- A **Layer 3** device that connects **different networks** and forwards traffic using **IP addresses**
- Example: Connecting a private network to the internet

### 3. Modem
- Used for **modulation/demodulation** — converts signals for internet communication
- Mostly used in ISP connectivity

### 4. Hub *(Obsolete)*
- Broadcasts data to **all ports** (very inefficient and noisy)
- Replaced by switches
- Operates at **Layer 1 (Physical Layer)**

### 5. NIC (Network Interface Card)
- Hardware component used to connect a device to a network
- Every NIC has a unique **MAC Address**
- Now built into most motherboards (wired + wireless)

### 6. Bridge *(Mostly Obsolete)*
- Connects two LANs and reduces unnecessary traffic compared to hubs
- Replaced by switches

---

## 4. Protocols

A **protocol** is a set of rules that defines **how data is transmitted and received** between devices in a network.

**Common Protocols:**
- `HTTP` / `HTTPS`
- `TCP` / `UDP`
- `IP`
- `DNS`
- `FTP`
- `SMTP` / `IMAP` / `POP3`
- `SSH`

---

## 5. IP Address

An **IP address** is a unique identifier given to a device on a network.

> ✅ Think of it as the **address of a device** so data can reach it correctly.

### IPv4
- **32-bit** address
- Format: `x.x.x.x`
- Example: `192.168.1.10`
- Total IPv4 addresses ≈ **4.3 billion** (running out → hence IPv6)
- Each address = 4 Octets (8 bits each) = 32 bits total

### IPv6
- **128-bit** address
- Written in hexadecimal groups
- Example: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- Provides a vastly larger address space (~340 undecillion addresses)
- Has built-in **IPsec** security support
- Uses **Anycast** and **Multicast** (no broadcast)
- Supports auto-configuration via **SLAAC** or **DHCPv6**

### IPv4 vs IPv6 Comparison

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Address Length | 32 bits (4 octets) | 128 bits (8 octets) |
| Format | Decimal, dots (192.168.0.1) | Hexadecimal, colons |
| Total Addresses | ~4.3 billion | ~340 undecillion |
| NAT Required | Yes | No |
| Security | External (IPsec optional) | Built-in IPsec |
| Broadcast | Supported | Not supported (uses Multicast/Anycast) |
| Configuration | Manual or DHCP | Auto-config (SLAAC) or DHCPv6 |

---

## 6. Types of IP Addresses

### 1. Public IP
- Used on the internet
- Assigned by ISP / cloud provider
- Globally reachable
- Example: Your website server IP

### 2. Private IP
- Used inside internal networks — **not routable on the public internet**
- Private IPv4 ranges:
  - `10.0.0.0/8`
  - `172.16.0.0/12`
  - `192.168.0.0/16`
- Example: EC2 private IP inside a VPC

### 3. Static IP
- Assigned manually or reserved
- Does not change
- Used for servers, database endpoints, VPN gateways

### 4. Dynamic IP
- Automatically assigned using **DHCP**
- Changes over time
- Used for laptops and home devices

### 5. Loopback IP
- Reserved for device to communicate with itself
- `127.0.0.1` → **localhost**

---

## 7. IPv4 Classes

IPv4 was traditionally divided into classes (managed by **IANA** — Internet Assigned Numbers Authority):

| Class | Range | Purpose | Default Subnet Mask |
|-------|-------|---------|---------------------|
| **A** | 1.0.0.0 – 126.0.0.0 | Large networks (big orgs) | 255.0.0.0 |
| **B** | 128.0.0.0 – 191.255.0.0 | Medium networks | 255.255.0.0 |
| **C** | 192.0.0.0 – 223.255.255.0 | Small networks (home/office) | 255.255.255.0 |
| **D** | 224.0.0.0 – 239.255.255.255 | Multicast | — |
| **E** | 240.0.0.0 – 255.255.255.255 | Experimental/Research | — |

**Private IP ranges by class:**
- Class A: `10.0.0.1` – `10.255.255.254`
- Class B: `172.16.0.1` – `172.31.255.254`
- Class C: `192.168.0.1` – `192.168.255.254`

> ✅ `0.0.0.0` has special meaning (default route / unknown address)
> ✅ `127.0.0.0/8` is reserved for loopback

---

## 8. Loopback Address (Localhost)

A **loopback address** allows a device to communicate with itself.

- `127.0.0.1` → **localhost**
- Any IP in the `127.x.x.x` range loops back to the same device

**Why it's used:**
- Testing local services without external networking
- Debugging applications
- Checking if a service is running locally

---

## 9. Network ID vs Host ID

Every IPv4 address contains:
- **Network ID** → identifies the network
- **Host ID** → identifies the device inside that network

| Class | Network ID | Host ID |
|-------|-----------|---------|
| A | 1st Octet | 2nd, 3rd, 4th |
| B | 1st + 2nd Octet | 3rd, 4th |
| C | 1st + 2nd + 3rd Octet | 4th |

**Connection rule:**
- Same Network ID → devices can talk **directly**
- Different Network ID → a **router is required**

---

## 10. Subnetting *(Core DevOps Skill)*

Subnetting divides a network into **smaller networks** for:
- Security
- Performance
- Efficient IP usage

### Example: Subnetting a /24 Network

```
Network: 192.168.1.0/24
Subnet Mask: 255.255.255.0
Total IPs: 256
Usable IPs: 254 (network + broadcast excluded)
```

**Split into two /25 networks:**
```
Subnet mask becomes: 255.255.255.128

Subnet 1: 192.168.1.0/25
Range: 192.168.1.0 – 192.168.1.127
Usable: 126 hosts

Subnet 2: 192.168.1.128/25
Range: 192.168.1.128 – 192.168.1.255
Usable: 126 hosts
```

**Benefits of Subnetting:**
- Reduces broadcast traffic
- Improves performance
- Improves isolation/security
- Helps structure cloud networking (public/private subnets)

---

## 11. CIDR

**CIDR (Classless Inter-Domain Routing)** replaced class-based networking and uses **prefix notation** like `/24`, `/16`.

Format: `IP/prefix`

Examples:
- `10.0.0.0/16` → 65,536 addresses
- `192.168.1.0/24` → 256 addresses

### Common CIDR Blocks

| Prefix | Netmask | Total Addresses | Meaning |
|--------|---------|-----------------|---------|
| /32 | 255.255.255.255 | 1 | Single host |
| /24 | 255.255.255.0 | 256 | Typical small subnet |
| /16 | 255.255.0.0 | 65,536 | Large subnet |
| /8 | 255.0.0.0 | 16,777,216 | Very large |
| /0 | 0.0.0.0 | Entire internet | Default route |

> ✅ `0.0.0.0/0` = **all destinations** (default internet route)

---

## 12. Network Models

Two major networking models are important in DevOps:

### A) OSI Model (7 Layers)

The OSI model explains how data moves from one device to another (Top → Bottom):

| Layer | Name | Protocols/Examples | Data Unit |
|-------|------|--------------------|-----------|
| L7 | **Application** | HTTP, FTP, SMTP | Data |
| L6 | **Presentation** | SSL/TLS, Encryption, Compression | Data |
| L5 | **Session** | Session management, NetBIOS | Data |
| L4 | **Transport** | TCP, UDP | Segment |
| L3 | **Network** | IP, Routing, ICMP | Packet |
| L2 | **Data Link** | MAC Addressing, Ethernet, ARP | Frame |
| L1 | **Physical** | Cables, signals, radio waves | Bit |

> 💡 Memory tip: **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing

**Layer notes:**
- **Hub** → Layer 1
- **Switch** → Layer 2
- **Router** → Layer 3

### B) TCP/IP Model (4 Layers)

This is what is actually used in real networking:

| TCP/IP Layer | Corresponds to OSI |
|-------------|-------------------|
| Application Layer | OSI L7 + L6 + L5 |
| Transport Layer | OSI L4 |
| Internet/Network Layer | OSI L3 |
| Network Access Layer | OSI L2 + L1 |

> ✅ Real-world internet runs on TCP/IP.

---

## 13. Ports and Protocols

A **port** identifies a service running on a machine.

> IP = server address | Port = specific application/service on that server

### Common Ports Cheat Sheet

| Service | Protocol | Port |
|---------|----------|------|
| HTTP | TCP | 80 |
| HTTPS | TCP | 443 |
| SSH | TCP | 22 |
| DNS | UDP/TCP | 53 |
| SMTP | TCP | 25 |
| FTP | TCP | 20/21 |
| SFTP | TCP | 22 |
| MySQL | TCP | 3306 |
| PostgreSQL | TCP | 5432 |
| Redis | TCP | 6379 |
| Kubernetes API | TCP | 6443 |
| Telnet | TCP | 23 |
| RDP | TCP | 3389 |

---

## 14. HTTP Basics

HTTP is a **client-server protocol** used for web communication.

**Important properties:**
- **Stateless** by default (server does not store client session)
- Works with requests and responses

### HTTP Methods

| Method | Purpose |
|--------|---------|
| `GET` | Fetch data |
| `POST` | Send data |
| `PUT` | Update resource |
| `DELETE` | Remove resource |
| `PATCH` | Partial update |

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| **200** | OK |
| **201** | Created |
| **301** | Permanent redirect |
| **302** | Temporary redirect |
| **400** | Bad request |
| **401** | Unauthorized |
| **403** | Forbidden |
| **404** | Not found |
| **500** | Internal server error |
| **502** | Bad gateway |
| **503** | Service unavailable |
| **504** | Gateway timeout |

### Cookies (Why login stays active)
Even though HTTP is stateless, login persists due to:
**Cookies + Sessions + Tokens**

Cookies store identifiers in the browser so the server can recognize you.

---

## 15. TCP vs UDP

Both operate at **Layer 4 (Transport Layer)**.

### TCP (Transmission Control Protocol)
- ✅ Reliable + connection-based
- ✅ Used when delivery must be guaranteed
- Slower due to overhead

**Examples:** Web browsing (HTTPS), Database connections, SSH

### TCP Three-Way Handshake
```
Client → Server : SYN
Server → Client : SYN-ACK
Client → Server : ACK
→ Connection established!
```

**Why TCP is Reliable:**
1. **Acknowledgement** — receiver confirms receipt
2. **Sequencing** — data divided into numbered segments
3. **Checksum** — detects corruption during transmission

### UDP (User Datagram Protocol)
- ✅ Faster + connectionless
- ✅ No guaranteed delivery
- Used where speed > reliability

**Examples:** DNS queries, Video streaming, Online gaming

### TCP vs UDP Comparison

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented | Connectionless |
| Reliability | Reliable (ACK) | Unreliable |
| Speed | Slower | Faster |
| Order | In-order delivery | No order guarantee |
| Use case | HTTP, SSH, FTP | DNS, Streaming, Gaming |
| Protocol number | 6 | 17 |

---

## 16. Routing

Routing decides how **packets** go from **source** to **destination** using a **route table**.

**Routing Rule Priority:**
- Most specific route wins first
- Example: `10.21.0.0/16` is evaluated before `10.0.0.0/8`
- Default route `0.0.0.0/0` matches everything else

**Default Route:**
- `0.0.0.0/0` → send traffic to internet gateway / NAT / firewall

---

## 17. DNS

**DNS (Domain Name System)** converts:
- Domain name → IP address

Example: `www.example.com` → `93.184.216.34`

> ✅ DNS is like the **"phonebook"** of the internet.

### How DNS Works (Simple Flow)
1. User types domain in browser
2. Query goes to **resolver DNS**
3. Resolver checks **cache**
4. If not found, asks:
   - Root DNS
   - TLD DNS (`.com`, `.org`)
   - Authoritative DNS
5. Returns IP to browser
6. Browser connects to server

### DNS Record Types

| Record | Meaning | Use |
|--------|---------|-----|
| **A** | IPv4 address | domain → IPv4 |
| **AAAA** | IPv6 address | domain → IPv6 |
| **CNAME** | Alias | maps one name to another |
| **MX** | Mail server | email routing |
| **NS** | Name server | DNS delegation |
| **TXT** | Text record | verification, SPF/DKIM |

---

## 18. DHCP

**DHCP (Dynamic Host Configuration Protocol)** automatically assigns network configuration to devices.

DHCP assigns:
- IP address
- Subnet mask
- Gateway
- DNS servers

Example: Connecting your laptop to Wi-Fi and getting an IP automatically.

### DORA Process (How DHCP Works)
1. **Discover** — Client broadcasts searching for DHCP server
2. **Offer** — Server responds with an available IP
3. **Request** — Client formally accepts the offered IP
4. **Acknowledge** — Server confirms IP assignment

**Why Use DHCP?**
- **Automation** — No manual IP assignment needed
- **Error Prevention** — Avoids duplicate IPs
- **Time-Saving** — Faster network setup
- **Central Management** — IT team manages IPs centrally

---

## 19. Network Security & Services

### Firewalls
Firewalls control **allowed and blocked traffic** using rules.

**Functions:**
1. Traffic Filtering — only allows legitimate traffic
2. Network Segmentation — isolates parts of the network
3. Protection — shields internal network from external threats

**Examples:**
- Allow SSH only from office IPs
- Block public access to internal databases

### Load Balancer
A load balancer distributes traffic across multiple servers.

**Benefits:**
- High availability
- Prevents overload
- Supports autoscaling
- Enables blue-green deployments

### VPN (Virtual Private Network)
VPN creates a **secure encrypted tunnel** between users and a private network.

**Used for:**
- Secure remote access
- Connecting offices
- Private cloud access

**VPN uses:**
- **Encryption** to protect data in transit
- **Authentication** to verify identities
- **Tunneling protocols** to securely transport data

**Advantages:** Scalability, low cost, security

### VLAN (Virtual Local Area Network)
A VLAN logically segments a physical network into multiple virtual networks.

**Why use VLANs:**
- Improve security by isolating sensitive data/devices
- Enhance scalability
- Reduce broadcast traffic
- Better traffic management

**Access Port vs Trunk Port:**
- **Access Port** — belongs to only one VLAN; device is unaware of VLAN
- **Trunk Port** — carries traffic for multiple VLANs (1 to 4094); used to connect switches/routers

### NAT (Network Address Translation)
NAT allows devices in a private network to access external networks by converting private IPs to public IPs.

**Types of NAT:**
1. **Static NAT** — one private IP → one public IP
2. **Dynamic NAT** — multiple private IPs → pool of public IPs
3. **PAT (Port Address Translation)** — multiple private IPs → one public IP using different ports (also called NAT overload)

---

## 20. Network Troubleshooting Tools

### 1. `ping`
Checks connectivity using ICMP.
```bash
ping google.com
```
Useful for: packet loss + latency check

### 2. `traceroute` / `tracert`
Shows route path to destination.
```bash
traceroute google.com
```
Useful for: identifying slow hops or failures

### 3. `telnet`
Checks if a port is reachable (basic connectivity test).
```bash
telnet example.com 443
```
> ⚠️ Not secure for remote login — use SSH instead

### 4. `curl`
Used to test HTTP endpoints.
```bash
curl http://example.com
curl -I https://example.com
```
Useful for: API testing + response headers

### 5. `dig`
DNS troubleshooting tool.
```bash
dig google.com
dig google.com A
```
Shows: TTL, records, authoritative servers

### 6. `netstat`
Shows active connections, listening ports, routing.
```bash
netstat -tulnp
netstat -r
```
Useful for: port listening + routing check

### 7. `nmap`
Scans systems and ports.
```bash
nmap -sn 192.168.1.0/24
nmap -A 192.168.1.10
```
Useful for: host discovery + port scanning
> ⚠️ Use only with permission

### 8. `ssh`
Secure remote login.
```bash
ssh user@server-ip
```
Used daily in DevOps.

### 9. `scp`
Securely copy files between machines.
```bash
scp file.txt user@server:/tmp/
```

---

## 21. Additional Concepts

### ARP (Address Resolution Protocol)
ARP maps a device's **IP address** to its **physical MAC address**.
- Used at the **Data Link Layer**
- When a device wants to communicate on the same local network, it uses ARP to find the recipient's MAC address

### MAC Address
- A **unique physical address** assigned to a network device
- **48 bits (6 bytes)**, written in hexadecimal (e.g., `00:1A:2B:3C:4D:5E`)
- First 3 bytes = manufacturer ID
- Last 3 bytes = device serial number

### RTT (Round Trip Time)
RTT is the time for a packet to travel from source to destination and back.
- Key metric for measuring network performance and latency

### Network Performance Metrics
| Metric | Description |
|--------|-------------|
| **Bandwidth** | Max data transfer rate (bits per second) |
| **Latency** | Time for signal to travel source → destination |
| **Reliability** | Consistency of successful data transmission |
| **Load** | Amount of data on the network at any given time |
| **MTU** | Maximum Transmission Unit — largest packet size |

### Unicast, Anycast, Multicast, Broadcast
| Type | Description |
|------|-------------|
| **Unicast** | One-to-one communication |
| **Anycast** | One-to-nearest (IPv6 only) |
| **Multicast** | One-to-many (specific group) |
| **Broadcast** | One-to-all devices on network |

### APIPA (Automatic Private IP Addressing)
Assigns a temporary IP in the `169.254.0.0/16` range when a device fails to receive an IP from a DHCP server.
- Device can communicate locally but **has no internet access**.

### CRC (Cyclic Redundancy Check)
A robust **error detection** technique based on binary division.
- A calculated remainder is appended to data before transmission
- Receiver performs the same calculation to detect discrepancies

### Network Topology Types

| Topology | Description | Pros | Cons |
|----------|-------------|------|------|
| **Bus** | All nodes on a single central cable | Simple, cost-effective | Single point of failure |
| **Star** | All devices connected to central hub/switch | Easy to manage | Hub failure = network down |
| **Ring** | Each node connected to two others in a circle | Simple data flow | One node failure breaks network |
| **Mesh** | Every node connected to multiple nodes | Highly fault-tolerant | Expensive and complex |
| **Tree** | Combination of Star + Bus | Scalable | Main bus failure impacts all |
| **Hybrid** | Combines multiple topologies | Flexible | Complex and expensive |

---

## 🔖 Quick Reference — DevOps Cheat Sheet

```
Network → IP Address → Subnet → Gateway → DNS → Internet

OSI Layers:  L7-App | L6-Pres | L5-Sess | L4-Trans | L3-Net | L2-DLink | L1-Phys
TCP/IP:      Application | Transport | Internet | Network Access

Private IPs: 10.x.x.x | 172.16-31.x.x | 192.168.x.x
Loopback:    127.0.0.1

Key Ports:   HTTP=80 | HTTPS=443 | SSH=22 | DNS=53 | MySQL=3306
CIDR:        /32=1 host | /24=256 | /16=65536 | /0=all

TCP = Reliable (3-way handshake: SYN → SYN-ACK → ACK)
UDP = Fast, no guarantee

DNS DORA equivalent: User → Resolver → Root → TLD → Auth → IP
DHCP DORA: Discover → Offer → Request → Acknowledge
```

---

*Notes compiled from: Networking for DevOps — Beginner → Advanced*
