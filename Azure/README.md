# Microsoft Azure — Complete Study Guide

> **Track:** Cloud Computing | **Level:** Beginner → Advanced  
> **Covers:** Account Setup · Core Infrastructure · VMs · Storage · Networking · Security  
> **Last Updated:** May 2026

---

## Table of Contents

1. [What is Microsoft Azure?](#1-what-is-microsoft-azure)
2. [Azure Free Account Setup](#2-azure-free-account-setup)
3. [Azure Portal Overview](#3-azure-portal-overview)
4. [Core Azure Concepts](#4-core-azure-concepts)
5. [Azure Regions & Availability Zones](#5-azure-regions--availability-zones)
6. [Virtual Machines (VMs)](#6-virtual-machines-vms)
7. [VM Deployment Types](#7-vm-deployment-types)
8. [VM Availability Options](#8-vm-availability-options)
9. [Connecting to Virtual Machines](#9-connecting-to-virtual-machines)
10. [VM Extensions](#10-vm-extensions)
11. [Azure Storage](#11-azure-storage)
12. [Azure Blob Storage — Deep Dive](#12-azure-blob-storage--deep-dive)
13. [Storage Tiers vs AWS S3 Classes](#13-storage-tiers-vs-aws-s3-classes)
14. [Azure Networking](#14-azure-networking)
15. [Azure Security](#15-azure-security)
16. [Azure Health & Monitoring](#16-azure-health--monitoring)
17. [Azure vs AWS — Full Comparison](#17-azure-vs-aws--full-comparison)
18. [Cost Management & Billing](#18-cost-management--billing)
19. [Key Terminologies Glossary](#19-key-terminologies-glossary)

---

## 1. What is Microsoft Azure?

**Microsoft Azure** is a **cloud computing platform** owned and operated by Microsoft. It is one of the top 3 global cloud providers alongside AWS (Amazon) and GCP (Google Cloud).

### Azure Service Categories

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Microsoft Azure Services                         │
├─────────────────────┬───────────────────────┬───────────────────────┤
│      COMPUTE        │       STORAGE         │      NETWORKING       │
│  Virtual Machines   │  Blob Storage         │  Virtual Network      │
│  App Service        │  File Share           │  Load Balancer        │
│  Azure Functions    │  Managed Disks        │  VPN Gateway          │
│  Container Apps     │  Table Storage        │  ExpressRoute         │
│  AKS (Kubernetes)   │  Queue Storage        │  Azure DNS            │
├─────────────────────┼───────────────────────┼───────────────────────┤
│    DATABASES        │   AI / ML             │     SECURITY          │
│  Azure SQL          │  Azure OpenAI         │  Azure AD (Entra ID)  │
│  Cosmos DB          │  Cognitive Services   │  Key Vault            │
│  Azure MySQL        │  ML Studio            │  Defender for Cloud   │
│  Azure PostgreSQL   │  Bot Service          │  Sentinel (SIEM)      │
├─────────────────────┼───────────────────────┼───────────────────────┤
│    DEVOPS           │   ANALYTICS           │     INTEGRATION       │
│  Azure DevOps       │  Azure Synapse        │  Logic Apps           │
│  Azure Pipelines    │  Data Factory         │  Service Bus          │
│  Container Registry │  Power BI Embedded    │  Event Grid           │
└─────────────────────┴───────────────────────┴───────────────────────┘
```

### Why Azure?

| Reason | Description |
|--------|-------------|
| **Hybrid Cloud Leader** | Best integration with on-premise Windows/Active Directory |
| **Enterprise Trust** | Used by 95% of Fortune 500 companies |
| **Microsoft Ecosystem** | Deep integration with Office 365, Teams, GitHub |
| **Global Reach** | 60+ regions — more than any other cloud provider |
| **Compliance** | 90+ compliance certifications (GDPR, HIPAA, ISO 27001) |
| **AI Leadership** | Exclusive partnership with OpenAI (GPT models) |

---

## 2. Azure Free Account Setup

### Free Tier Breakdown

| Offer Type | Details |
|-----------|---------|
| **Free Credit** | $200 USD valid for 30 days |
| **Always Free** | 55+ services free forever (with limits) |
| **12-Month Free** | Popular services free for first 12 months |

> After 30 days or $200 credit exhausted — only "Always Free" services remain unless upgraded.

### Prerequisites

```
Required before signup:
─────────────────────────────────────────────────────
[✓] Valid email address (Microsoft/Outlook preferred)
[✓] Phone number (for OTP identity verification)
[✓] Credit/Debit card (identity check only — NOT charged)
[✓] Stable internet connection
[✓] Modern browser (Chrome, Edge, Firefox)
```

### Step-by-Step Account Creation

```
STEP 1: Visit https://azure.microsoft.com/en-us/free
        Click "Start free" button

STEP 2: Sign In or Create Microsoft Account
        ├── Existing account → Enter email + password
        └── New account → Create one! → Enter email → Set password → Verify OTP

STEP 3: Fill Profile Details
        Country/Region  → Select your country (India, etc.)
        First Name      → Your first name
        Last Name       → Your last name
        Phone Number    → Enter mobile for OTP
        Click "Text me" → Enter OTP → Verify

STEP 4: Credit/Debit Card Verification (Identity ONLY)
        Card Number     → 16-digit number
        Expiry Date     → MM/YY
        CVV             → 3-digit code
        Billing Address → Your address
        ⚠️ Azure holds ₹2/$1 temporarily — auto-reversed in days

STEP 5: Agree to Terms
        Check: "I agree to the subscription agreement..."
        Click: "Sign up"

STEP 6: Done!
        Click "Go to the Azure portal" → portal.azure.com
```

### Always-Free Services (No Expiry)

| Service | Free Limit |
|---------|-----------|
| Azure App Service | 10 web apps |
| Azure Functions | 1 million requests/month |
| Azure Cosmos DB | 1,000 RU/s + 25 GB storage |
| Azure SQL Database | 32 GB storage (Basic tier) |
| Azure Blob Storage | 5 GB LRS storage |
| Azure DevOps | 5 users, unlimited private repos |
| Azure Static Web Apps | 100 GB bandwidth/month |
| Azure Key Vault | 10,000 transactions/month |
| Azure Monitor | Basic metrics & logs |

### 12-Month Free Services

| Service | Free Limit |
|---------|-----------|
| Linux VM (B1s) | 750 hours/month |
| Windows VM (B1s) | 750 hours/month |
| Azure SQL Database | 250 GB/month |
| Azure Blob Storage | 5 GB + 2M read ops |
| Azure Files | 100 GB storage |
| Azure Bandwidth | 15 GB outbound |

---

## 3. Azure Portal Overview

### Portal Layout

```
┌──────────────────────────────────────────────────────────────────┐
│  🔷 Microsoft Azure        [Search bar]          [Your Name] [?] │
├──────────────────────────────────────────────────────────────────┤
│  🏠 Home  📊 Dashboard  📋 All Services  📌 Favorites           │
├────────────────┬─────────────────────────────────────────────────┤
│                │                                                  │
│  Left          │           Main Content Area                      │
│  Navigation    │                                                  │
│  Panel         │   [Create a resource]  [Recent resources]        │
│                │                                                  │
│  + Create      │   [Subscriptions]  [Resource Groups]            │
│    Resource    │                                                  │
│                │   [Cost Management]  [Azure Active Directory]   │
│  All Services  │                                                  │
│                │                                                  │
│  Favorites:    │                                                  │
│  - VMs         │                                                  │
│  - Storage     │                                                  │
│  - Networking  │                                                  │
└────────────────┴─────────────────────────────────────────────────┘
```

### Key Portal Areas

| Area | Path | Purpose |
|------|------|---------|
| **Home** | portal.azure.com | Quick access to recent resources |
| **Resource Groups** | All services → Resource Groups | Organize resources by project |
| **Subscriptions** | Search "Subscriptions" | View billing and credits |
| **All Services** | Left panel | Browse all 200+ Azure services |
| **Cost Management** | Search "Cost Management + Billing" | Monitor spending and credits |
| **Cloud Shell** | `>_` icon in top bar | Browser-based Bash/PowerShell terminal |
| **Azure AD (Entra ID)** | Search "Azure Active Directory" | User and identity management |

### Important URLs

| URL | Purpose |
|-----|---------|
| `portal.azure.com` | Main Azure Portal |
| `azure.microsoft.com/free` | Free account signup |
| `learn.microsoft.com` | Official Microsoft Learning |
| `azure.microsoft.com/pricing` | Azure Pricing Calculator |
| `status.azure.com` | Azure Service Health Status |

---

## 4. Core Azure Concepts

### Subscription

A **Subscription** is the billing unit in Azure. Everything you create belongs to a subscription. Free account = "Free Trial" subscription.

```
Account (Microsoft Email)
  └── Subscription: Free Trial  ($200 credit)
        └── All Azure resources you create
```

### Resource Group

A **Resource Group** is a logical container (like a folder) for organizing related Azure resources.

```
Best Practice Structure:
──────────────────────────────────────────────────────
Subscription: Free Trial
└── Resource Group: webapp-production-rg
      ├── Virtual Machine: webapp-vm-01
      ├── Storage Account: webappstorage123
      ├── Virtual Network: webapp-vnet
      ├── NSG: webapp-nsg
      └── Public IP: webapp-pip
```

**Why use Resource Groups:**
- Delete ALL resources of a project with one click
- Apply permissions (RBAC) to all resources at once
- Tag for cost tracking (`Project: MyApp`, `Env: Production`)
- View all costs for one project together

### Region

Azure has data centers around the world called **Regions**. Choose the region **closest to your users** for best performance.

```
Popular Regions:
─────────────────────────────────────────────────────────
India:      Central India (Pune)     ← 3 AZs, best for India
            West India (Mumbai)       ← No AZs
            South India (Chennai)     ← Limited services

US:         East US, West US 2        ← Most services available
Europe:     West Europe, North Europe
Asia Pac:   Southeast Asia (Singapore), East Asia (HK)
```

### Resource

Any service you create in Azure (VM, database, storage, etc.) is a **Resource**. Every resource has:

```
Resource Properties:
─────────────────────────────────
Name:             my-virtual-machine
Type:             Microsoft.Compute/virtualMachines
Resource Group:   myapp-rg
Location/Region:  Central India
Subscription:     Free Trial
Tags:             Env=Production, Team=Backend
```

### Tags

Tags are key-value labels applied to resources for:
- Cost allocation tracking
- Filtering and searching resources
- Applying automation policies

```
Common Tag Examples:
  Environment = Production / Development / Testing
  Team        = Backend / Frontend / DevOps
  Project     = MyApp / DataPipeline
  Owner       = teamlead@company.com
  CostCenter  = Engineering-001
```

---

## 5. Azure Regions & Availability Zones

### What is an Availability Zone?

An **Availability Zone (AZ)** is a **physically separate data center** within a single Azure Region. Each zone has its own power, cooling, and networking — completely independent.

```
Azure Region: Central India (Pune)
┌──────────────────────────────────────────────────────┐
│                                                      │
│   Zone 1           Zone 2           Zone 3          │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐    │
│  │  DC-A    │     │  DC-B    │     │  DC-C    │    │
│  │          │     │          │     │          │    │
│  │ Own Power│     │ Own Power│     │ Own Power│    │
│  │ Own Cool │     │ Own Cool │     │ Own Cool │    │
│  │ Own Net  │     │ Own Net  │     │ Own Net  │    │
│  └──────────┘     └──────────┘     └──────────┘    │
│                                                      │
│  ←────── Connected by low-latency fiber ──────────► │
└──────────────────────────────────────────────────────┘

If Zone 1 entire DC fails → Zone 2 & 3 still fully operational ✅
```

### Indian Azure Regions

| Region | Location | AZs Available | Best For |
|--------|----------|:------------:|---------|
| **Central India** | Pune | ✅ 3 AZs | Production workloads — full HA |
| **West India** | Mumbai | ❌ None | Closest to Mumbai users, no zone redundancy |
| **South India** | Chennai | ❌ None | Chennai-based users, limited services |

### Region Selection Rule

```
Decision Guide:
─────────────────────────────────────────────────────────────────
Users in India + Need High Availability  → Central India (Pune)
Users in Western India + Cost priority   → West India (Mumbai)
Compliance requires data in India        → Any India region
Global/International users               → East US, West Europe
Dev/Test (any region is fine)            → Pick cheapest

Latency Benchmark:
  Pune → Central India = ~5ms
  Mumbai → West India = ~5ms
  User in Europe → West Europe = ~10ms
  User in US → East US = ~15ms
```

### SLA by Availability Configuration

| Configuration | SLA Uptime | Downtime/Year |
|--------------|:----------:|:-------------:|
| Single VM (Premium SSD) | 99.9% | 8.7 hours |
| Availability Set (2+ VMs) | 99.95% | 4.4 hours |
| Availability Zone (2+ zones) | 99.99% | 52 minutes |
| Multi-Region Active-Active | 99.999% | 5 minutes |

---

## 6. Virtual Machines (VMs)

### What is a Virtual Machine?

A **Virtual Machine** is a software-based computer running on Azure physical hardware. You get full control over the OS, software, network, and storage — just like a physical server.

```
Virtual Machine Anatomy:
──────────────────────────────────────────────────────────────
Virtual Machine
├── vCPU          → Processing power (1, 2, 4, 8, 16, 32 cores)
├── RAM           → Memory (1 GB to 768 GB depending on size)
├── OS Disk       → Boot disk (Windows or Linux OS installed here)
│     └── Type: Standard HDD / Standard SSD / Premium SSD
├── Data Disks    → Additional storage disks (optional, up to 64)
├── NIC           → Network Interface Card (has Private IP)
│     └── Can also have Public IP attached
├── NSG           → Network Security Group (firewall rules)
└── VNet          → Virtual Network (private network it belongs to)
```

### VM Size Naming Convention

```
Standard_D4s_v5
    │    │ │  │
    │    │ │  └── Version (v5 = 5th gen hardware)
    │    │ └───── s = Supports Premium SSD
    │    └──────── 4 = 4 vCPUs
    └───────────── D = General Purpose family
```

### VM Size Families

| Series | Optimized For | vCPU Range | Use Cases |
|--------|-------------|:----------:|-----------|
| **B-series** | Burstable (dev/test) | 1–32 | Small apps, dev envs, FREE TIER (B1s) |
| **D-series** | General Purpose | 2–96 | Web servers, enterprise apps |
| **E-series** | Memory Optimized | 2–96 | Databases, in-memory caching (Redis) |
| **F-series** | Compute Optimized | 2–72 | Gaming servers, batch processing |
| **L-series** | Storage Optimized | 8–80 | Big data, NoSQL, data warehouses |
| **N-series** | GPU Optimized | 6–64 | ML training, AI inference, rendering |
| **M-series** | Memory Extreme | 8–416 | SAP HANA, large databases |

### Free Tier VM Spec

```
Size:   Standard_B1s
vCPU:   1
RAM:    1 GB
Disk:   30 GB (OS disk)
Free:   750 hours/month for 12 months
OS:     Linux OR Windows (not both simultaneously on same subscription)
```

### AWS EC2 to Azure VM Mapping

| Azure VM Family | AWS EC2 Equivalent | Type |
|----------------|-------------------|------|
| B-series | T-series (T2, T3) | Burstable |
| D-series | M-series (M5, M6i) | General Purpose |
| E-series | R-series (R5, R6i) | Memory Optimized |
| F-series | C-series (C5, C6i) | Compute Optimized |
| N-series (GPU) | P-series, G-series | GPU |

---

## 7. VM Deployment Types

### Type 1 — Single Virtual Machine

One standalone VM you fully control. Simple but has single point of failure.

```
Use Case:        Dev/Test, Learning, Personal Projects
Reliability:     ❌ No HA — if VM goes down, app is down
Cost:            ✅ Cheapest
Management:      Manual — you manage OS, updates, networking
SLA:             99.9%

You → VM → Your Application
```

### Type 2 — VM Scale Set (VMSS)

Group of identical VMs that auto-scale based on demand. Azure's answer to AWS Auto Scaling Groups.

```
Low Traffic:                  High Traffic (CPU spike):

    [LB]                          [LB]
     │                             │
  [VM1][VM2]         [VM1][VM2][VM3][VM4][VM5]
                          ↑ Auto-added by Azure

Scale Rules:
  CPU > 70% for 5 min  →  Add 2 VMs
  CPU < 20% for 10 min →  Remove 1 VM
  Min: 2 VMs    Max: 10 VMs
```

**VMSS Benefits:**
- Auto-scaling up and down based on demand
- Load balanced automatically
- VMs spread across zones automatically
- Self-healing — failed VMs replaced automatically
- Manage 1 to 1,000 VMs as a single unit

### Type 3 — Presets (Preconfigured Templates)

Ready-made VM configurations optimized for specific workloads. No need to manually choose every setting.

| Preset | CPU:RAM Ratio | Best For |
|--------|:------------:|---------|
| General Purpose | 1:4 | Web servers, enterprise apps |
| Compute Optimized | 1:2 | Gaming, batch, HPC |
| Memory Optimized | 1:8 | Databases, in-memory caching |
| Storage Optimized | High IOPS | Big data, data warehouses |
| GPU Optimized | GPU + CPU | ML training, rendering |

### Type 4 — Hybrid (On-Premise + Cloud)

Combines company on-premise physical servers with Azure cloud VMs.

```
Company Office (On-Premise)                 Microsoft Azure
┌───────────────────────────┐               ┌──────────────────────┐
│  Physical Servers         │               │  Azure VMs           │
│  Active Directory         │◄──VPN/ER─────►│  Azure AD (Entra ID) │
│  Local Databases          │               │  Azure Storage       │
│  Legacy Apps              │               │  Azure Backup        │
└───────────────────────────┘               └──────────────────────┘
                    Hybrid Cloud Setup

Key Services:
  Azure Arc           → Manage on-premise servers FROM Azure portal
  Azure Site Recovery → Replicate on-prem servers to Azure (DR)
  Azure Migrate       → Migration tool for on-prem to Azure
  Azure ExpressRoute  → Dedicated private circuit (not internet)
  Azure VPN Gateway   → Encrypted VPN tunnel over internet
```

---

## 8. VM Availability Options

When creating a VM, you choose how it's protected from failures.

### Option 1 — No Infrastructure Redundancy

Single VM, no protection. For dev/test only.

```
[VM-1]
  └── If physical hardware fails → App is DOWN ❌
SLA: 99.9%   Cost: Lowest   Use: Dev/Test/Learning
```

### Option 2 — Availability Zone

VM placed in a specific physical data center (zone). Deploy multiple VMs across zones for protection.

```
Region: Central India
┌────────────────────────────────────────────────────┐
│  Zone 1 (DC-A)   Zone 2 (DC-B)    Zone 3 (DC-C)  │
│    [VM-1]           [VM-2]            [VM-3]       │
└────────────────────────────────────────────────────┘
Zone 1 entire DC fails → VM-2 and VM-3 still running ✅
SLA: 99.99%   Cost: Higher   Use: Production workloads
```

### Option 3 — VM Scale Set (VMSS Across Zones)

Azure automatically distributes VMSS VMs across all available zones.

```
VMSS auto-distribution:
Zone 1     Zone 2     Zone 3
[VM-1]     [VM-2]     [VM-3]
[VM-4]     [VM-5]     [VM-6]
  ↑ Azure places VMs automatically — no manual zone assignment

SLA: 99.99%   Cost: High   Use: High-traffic production
```

### Option 4 — Availability Set

Protects VMs from hardware failures **within** a single data center. Uses Fault Domains and Update Domains.

```
One Data Center (e.g., West India — no AZs available):
┌──────────────────────────────────────────────────────┐
│                                                      │
│  Fault Domain 0              Fault Domain 1          │
│  ┌──────────────────┐       ┌──────────────────┐    │
│  │   Rack A         │       │   Rack B         │    │
│  │   Own Power/Net  │       │   Own Power/Net  │    │
│  │   [VM-1] [VM-3]  │       │   [VM-2] [VM-4]  │    │
│  └──────────────────┘       └──────────────────┘    │
│                                                      │
│  If Rack A fails → VM-2 & VM-4 still running ✅     │
│  If entire DC fails → ALL VMs go down ❌            │
└──────────────────────────────────────────────────────┘
```

| Concept | Definition |
|---------|-----------|
| **Fault Domain** | VMs sharing same physical rack (power + network switch). Max 3. |
| **Update Domain** | VMs rebooted together during Azure maintenance. Max 20. |

**Key rule:** Use Availability Sets in regions WITHOUT AZs (West India, South India).

### Availability Options — Comparison

| Feature | No Redundancy | Availability Set | Availability Zone | VMSS |
|---------|:------------:|:----------------:|:-----------------:|:----:|
| HW failure protection | ❌ | ✅ (within DC) | ✅ | ✅ |
| DC failure protection | ❌ | ❌ | ✅ | ✅ |
| Auto-scaling | ❌ | ❌ | ❌ | ✅ |
| All regions | ✅ | ✅ | ❌ (AZ only) | ✅ |
| SLA | 99.9% | 99.95% | 99.99% | 99.99% |
| Cost | Lowest | Low | Medium | High |
| Best For | Dev/Test | Legacy/Non-AZ | Production | High-traffic |

---

## 9. Connecting to Virtual Machines

### Method 1 — Azure Bastion (Most Secure)

Browser-based SSH/RDP directly from Azure Portal — VM needs NO public IP.

```
Your Browser → Azure Portal → Azure Bastion → VM (Private IP only)
                               (Managed jump host)

Path: VM → Connect → Bastion → Enter username + key/password
```

| Property | Value |
|----------|-------|
| OS Support | Linux + Windows |
| Public IP needed | ❌ No |
| Software needed | None (browser only) |
| Cost | ~$0.19/hour (paid service) |
| Security | Highest — TLS encrypted, Azure AD protected |
| Best For | Production VMs |

### Method 2 — SSH via CLI (Linux VMs)

Standard SSH connection from local terminal or Azure Cloud Shell.

```bash
# Basic SSH
ssh azureuser@<public-ip>

# SSH with private key (.pem file from Azure)
ssh -i ~/.ssh/mykey.pem azureuser@20.195.123.45

# Via Azure CLI
az login
az ssh vm --resource-group MyRG --name MyVM

# SSH key authentication diagram:
Your Machine            Azure Linux VM
┌──────────────┐        ┌─────────────────────────┐
│ Private Key  │◄──────►│ Public Key in            │
│ (id_rsa.pem) │ match  │ ~/.ssh/authorized_keys  │
└──────────────┘        └─────────────────────────┘
```

### Method 3 — RDP (Windows VMs)

Full graphical Windows desktop connection.

```
Your Machine              Azure Windows VM
┌──────────────────┐      ┌────────────────────────┐
│ Remote Desktop   │◄────►│  Windows Server Desktop│
│ App (mstsc)      │ 3389 │  Full GUI interface    │
└──────────────────┘      └────────────────────────┘

Steps:
  Windows: Win+R → mstsc → Enter Public IP → Login
  Mac:     Download "Microsoft Remote Desktop" from App Store
```

### Method 4 — Serial Console (Emergency)

Direct hardware-level access — works even when OS is crashed or network is down.

```
Normal SSH/RDP fails because: OS crashed / wrong firewall / boot loop

Serial Console:
  Azure Portal → VM → Help (left sidebar) → Serial Console
  → Direct terminal to VM hardware
  → Works REGARDLESS of OS/network state

Use ONLY for:
  - VM stuck in boot loop
  - Locked out (wrong NSG/firewall rules)
  - OS crashed — need diagnostics
  - Network configuration broken
  - Emergency recovery
```

### MobaXterm (Windows Tool)

**MobaXterm** is a Windows terminal combining SSH, RDP, SFTP, and X11 forwarding in one tool.

```
Connection Methods in MobaXterm:

1. Key-Based SSH:
   Session → SSH → Remote Host: <VM IP>
   Check: "Use private key" → Browse to .pem file
   Username: azureuser → OK (no password needed)

2. Password-Based SSH:
   Session → SSH → Remote Host → Enter username
   MobaXterm prompts for password

3. RDP (Windows VM GUI):
   Session → RDP → Remote Host: <VM IP>
   Enter username + password → Full Windows desktop

4. X11 Forwarding (Linux GUI apps):
   Session → SSH → Enable X11 forwarding
   GUI apps from Linux VM render on your Windows screen

MobaXterm Advantages:
  ✅ Built-in SFTP file browser (drag & drop files)
  ✅ Multiple sessions in tabs
  ✅ Saves all session configs
  ✅ Built-in X11 server (no Xming needed)
  ✅ Free for personal use
```

### Connection Methods — Comparison

| Method | OS | Public IP | Cost | Use Case |
|--------|-------|:---------:|------|---------|
| **Bastion** | Linux + Windows | ❌ No | ~$0.19/hr | Secure production |
| **SSH CLI** | Linux only | ✅ Yes | Free | Daily Linux mgmt |
| **RDP** | Windows only | ✅ Yes | Free | Windows GUI access |
| **Serial Console** | Both | ❌ No | Free | Emergency only |

---

## 10. VM Extensions

### What are VM Extensions?

VM Extensions are **lightweight software agents** that run inside your VM to automate post-deployment tasks — without you manually logging in.

```
Traditional Approach:               With Extensions:
Deploy VM                           Deploy VM
    ↓                                   ↓
SSH into VM                         Attach Extension (Portal/CLI/ARM)
    ↓                                   ↓
Manually run commands               Extension runs AUTOMATICALLY
    ↓                                   ↓
Install software                    Software installed silently
                                    No login required
```

### How Extensions Execute

```
Azure Portal / CLI / ARM Template / Terraform
              │
              │ (sends extension configuration)
              ▼
    Azure VM Agent (pre-installed in all Azure VMs)
              │
              │ (downloads + executes extension)
              ▼
    Extension runs as root/SYSTEM (background process)
              │
              ▼
    Status reported back → Azure Portal
```

### Common VM Extensions

| Extension | OS | Purpose |
|-----------|----|---------| 
| **Custom Script Extension** | Linux + Windows | Run any shell/PowerShell script on VM |
| **Azure Monitor Agent** | Linux + Windows | Send logs/metrics to Azure Monitor |
| **Microsoft Antimalware** | Windows | Windows Defender + AV policy management |
| **DSC (Desired State Config)** | Windows | Enforce OS configuration compliance |
| **Key Vault Extension** | Linux + Windows | Auto-rotate + inject Key Vault certificates |
| **Azure AD Login Extension** | Linux + Windows | Login with Azure AD credentials |
| **Diagnostics Extension** | Linux + Windows | Boot diagnostics, crash dumps |
| **NVIDIA GPU Driver** | Linux + Windows | Auto-install GPU drivers (N-series VMs) |
| **Dependency Agent** | Linux + Windows | Map network connections (Azure Monitor) |

### Custom Script Extension Example

```bash
#!/bin/bash
# This script runs automatically via Custom Script Extension
# Azure executes this silently — no one needs to log in

apt-get update -y
apt-get install -y nginx docker.io
systemctl start nginx
systemctl enable nginx
systemctl start docker

echo "<h1>Deployed via Azure VM Extension — Zero Touch!</h1>" \
  > /var/www/html/index.html
```

Store this in Azure Blob Storage or GitHub — reference in extension config — Azure runs it automatically.

---

## 11. Azure Storage

### Five Azure Storage Services

```
Storage Account (mystorageaccount123)
├── Blob Storage        → Object storage (S3 equivalent)
│     ├── Container: images
│     │     ├── photo1.jpg
│     │     └── backup.zip
│     └── Container: logs
│           └── app-log-2026.txt
│
├── File Share          → Managed network file system (EFS equivalent)
│     └── shared-drive/ (mountable on VMs via SMB/NFS)
│
├── Queue Storage       → Message queue (SQS equivalent)
│     └── order-queue (messages between app components)
│
├── Table Storage       → NoSQL key-value store (DynamoDB basic equiv.)
│     └── usersTable
│
└── Disk Storage        → Block storage for VMs (EBS equivalent)
      └── myvm-os-disk (attached to VM, bootable)
```

### Storage Account Naming Rules

```
Rules:
  ✅ 3 to 24 characters
  ✅ Lowercase letters and numbers only
  ✅ Must be globally unique across ALL of Azure
  ❌ No hyphens, underscores, uppercase

Valid:   mystorageaccount123
Invalid: My-Storage_Account  (uppercase + special chars)
```

### Storage Account Types

| Type | Services | Use Case |
|------|----------|---------|
| **Standard GPv2** ← Recommended | Blob, File, Queue, Table | Most scenarios |
| **Premium Block Blobs** | Blob only | Low-latency blob workloads |
| **Premium File Shares** | File only | High-performance file shares |
| **Premium Page Blobs** | Page Blobs | VHD disks for Azure VMs |

### AWS to Azure Storage Mapping

| AWS Service | Azure Equivalent | Type |
|-------------|----------------|------|
| S3 | Azure Blob Storage | Object storage |
| EBS | Azure Managed Disk | Block storage (bootable) |
| EFS | Azure File Share | Network file system |
| SQS | Azure Queue Storage | Message queuing |
| DynamoDB (basic) | Azure Table Storage | NoSQL key-value |
| DynamoDB (advanced) | Azure Cosmos DB | Globally distributed NoSQL |
| S3 Glacier | Azure Archive Tier | Long-term cold archive |
| CloudFront + S3 | Azure CDN + Blob | CDN with object storage |

---

## 12. Azure Blob Storage — Deep Dive

### What is Blob Storage?

**Blob** = **Binary Large Object** — any file of any format. Blob Storage stores massive amounts of unstructured data.

```
Blob Storage Structure:
────────────────────────────────────────────────────────────
Storage Account  →  Container  →  Blobs (files)
(mystorageacct)     (images)       (photo1.jpg, logo.png)
                    (backups)      (backup-2026-01.zip)
                    (logs)         (app-error-2026.log)

AWS equivalent:
S3 Bucket  ≈  Container
S3 Object  ≈  Blob
```

### Three Blob Types

| Blob Type | Description | Best For |
|-----------|-------------|---------|
| **Block Blob** | Data stored as blocks; streaming optimized | Files, images, videos, backups (most common) |
| **Append Blob** | Optimized for append-only operations | Log files, audit trails |
| **Page Blob** | 512-byte pages; random read/write | VHD files for VM OS disks |

### Why Blob Storage CANNOT Be a Root Volume (OS Disk)

```
Block Storage (Managed Disk / EBS):   Object Storage (Blob / S3):
─────────────────────────────────     ────────────────────────────
Direct block-level I/O               HTTP REST API access
Bootable ✅                           Bootable ❌
Has file system (ext4, NTFS)         No file system
Random read/write (OS needs this)    Whole-object replace only
High IOPS (fast)                     Higher latency
VM can boot from it                  Cannot install/run OS from it

Rule: OS needs millisecond block-level I/O for boot and runtime.
      Object storage works via HTTP — fundamentally incompatible as boot device.
```

### Key Blob Storage Concepts

```
SAS Token (Shared Access Signature):
  Time-limited URL granting access to a specific blob
  Example: https://storage.blob.core.windows.net/images/photo.jpg?sv=2023&se=2026-12-31&sig=abc123
  Use case: Share private blobs temporarily without exposing credentials

Blob Versioning:
  Every overwrite creates a new version
  Can restore previous versions
  Similar to S3 Object Versioning

Soft Delete:
  Blobs moved to "deleted" state for N days before permanent removal
  Recovery window for accidental deletions

Static Website Hosting:
  Enable on storage account → host HTML/CSS/JS
  URL: https://mystorageacct.z13.web.core.windows.net
  Similar to S3 Static Website Hosting
```

---

## 13. Storage Tiers vs AWS S3 Classes

### The Core Concept

```
Data Access Frequency Over Time:

Frequent ────────────────────────────────────► Rare/Never
   │                                                │
[Hot]     →     [Cool]     →    [Cold]    →   [Archive]
   │                                                │
Expensive                                       Cheapest
Immediate                               Hours to retrieve
```

### Azure Blob Storage Tiers

| Tier | Storage Cost | Access Cost | Retrieval | Min Duration | Best For |
|------|:----------:|:----------:|:---------:|:------------:|---------|
| **Hot** | Highest | Lowest | Instant | None | Active data, daily access |
| **Cool** | ~50% less than Hot | Higher | Instant | 30 days | Monthly access, short backups |
| **Cold** | Lower than Cool | Higher | Instant | 90 days | Quarterly access, compliance |
| **Archive** | Lowest (90% cheaper) | Highest | **Hours** (rehydration) | 180 days | Annual access, regulatory |

### Archive Rehydration

```
Archive Tier blob = OFFLINE — cannot be read directly

To read an archived blob:
  1. Change tier to Hot or Cool (rehydrate)
  2. Wait for rehydration:
     Standard priority → up to 15 hours
     High priority     → under 1 hour (higher cost)
  3. Now the blob is readable

⚠️ Early deletion fees:
  Cool:    30-day minimum → delete early = charged for full 30 days
  Cold:    90-day minimum → delete early = charged for full 90 days
  Archive: 180-day minimum → delete early = charged for full 180 days
```

### AWS S3 Storage Classes Mapping

| Azure Tier | AWS S3 Equivalent | Key Difference |
|-----------|-----------------|----------------|
| Hot | S3 Standard | Same |
| Cool | S3 Standard-IA | Same |
| Cold | S3 Glacier Instant Retrieval | Same |
| Archive | S3 Glacier Flexible / Deep Archive | Azure = up to 15hr; AWS Deep Archive = 12-48hr |
| N/A | S3 One Zone-IA | Azure has no single-zone equivalent |
| Lifecycle Policies | S3 Intelligent-Tiering (AI) | Azure = rule-based; AWS = ML-driven |

### Lifecycle Management — Auto Tier Transitions

```
Example Azure Lifecycle Policy:
────────────────────────────────────────────────────────────────
If blob not accessed for  30 days  → Move to Cool tier
If blob not accessed for  90 days  → Move to Cold tier
If blob not accessed for 365 days  → Move to Archive tier
If blob older than       7 years   → Delete permanently

Why this matters:
  - Zero manual data management
  - Automatic cost optimization
  - Enforces data retention compliance
  - AWS equivalent: S3 Lifecycle Rules
```

---

## 14. Azure Networking

### Virtual Network (VNet)

Azure VNet is your private network in Azure — equivalent to AWS VPC.

```
VNet: 10.0.0.0/16
│
├── Subnet: frontend  (10.0.1.0/24)
│     ├── VM: web-server-01  (10.0.1.4)
│     └── VM: web-server-02  (10.0.1.5)
│
├── Subnet: backend   (10.0.2.0/24)
│     ├── VM: app-server-01  (10.0.2.4)
│     └── VM: app-server-02  (10.0.2.5)
│
└── Subnet: database  (10.0.3.0/24)
      └── Azure SQL Database  (10.0.3.4)
```

### Network Security Group (NSG)

NSG = Firewall rules for VMs and subnets. Equivalent to AWS Security Groups.

```
NSG Rule Example (Web Server):
────────────────────────────────────────────────────────
Direction  Priority  Port   Protocol  Source          Action
─────────  ────────  ─────  ────────  ──────────────  ──────
Inbound    100       80     TCP       Any (0.0.0.0/0) Allow  ← HTTP
Inbound    110       443    TCP       Any             Allow  ← HTTPS
Inbound    120       22     TCP       My IP only      Allow  ← SSH (restricted!)
Inbound    65000     Any    Any       Any             Deny   ← Block all else

Outbound   100       Any    Any       Any             Allow  ← Allow all outbound

Lower priority number = higher priority (100 is checked before 200)
```

### Load Balancer Types

| Type | Layer | AWS Equivalent | Use Case |
|------|:-----:|---------------|---------|
| **Azure Load Balancer** | Layer 4 (TCP/UDP) | AWS Classic ELB | Simple TCP load balancing |
| **Application Gateway** | Layer 7 (HTTP/HTTPS) | AWS ALB | URL-based routing, SSL termination, WAF |
| **Traffic Manager** | DNS-based | AWS Route 53 (LB mode) | Global traffic distribution |
| **Azure Front Door** | Global CDN+LB | AWS CloudFront + ALB | Global HTTP load balancing + CDN |

---

## 15. Azure Security

### Azure Active Directory (Azure AD / Entra ID)

Azure's identity and access management service. Equivalent to AWS IAM.

```
Azure AD Concepts:
  Tenant    → Your organization's Azure AD directory
  User      → Individual identity (employee, service account)
  Group     → Collection of users
  App Reg   → Register apps to authenticate via Azure AD
  RBAC      → Role-Based Access Control

Common Built-in Roles:
  Owner          → Full control including assign permissions to others
  Contributor    → Create/manage all resources, NO permission management
  Reader         → View everything, NO changes
  User Access    → Manage user access only, NO resource changes
  Administrator
```

### Role-Based Access Control (RBAC)

```
Assignment structure:
Who (Principal)   +   What (Role)    +   Scope
────────────────      ─────────────      ────────────────────────
User/Group/       +   Owner/          +  Subscription /
Service Principal     Contributor /       Resource Group /
                      Reader /            Individual Resource
                      Custom Role

Example:
  John (User) + Contributor + Resource Group: webapp-rg
  → John can create/delete resources in webapp-rg only
```

### Azure Key Vault

Secure store for secrets, keys, and certificates.

```
Key Vault stores:
  Secrets     → Connection strings, API keys, passwords
  Keys        → Encryption/decryption keys (HSM-backed)
  Certificates → TLS/SSL certificates (auto-rotate)

Access:
  Via RBAC roles (preferred)
  Via Access Policies (legacy)

Best Practice:
  Never store secrets in app config files or code
  Always reference Key Vault: @Microsoft.KeyVault(SecretUri=...)
```

---

## 16. Azure Health & Monitoring

### Resource Health

Shows health status of YOUR specific Azure resource.

```
Navigation: VM → Help (left sidebar) → Resource Health

Status Values:
  Available    → Resource running normally
  Unavailable  → Resource down (Azure platform issue)
  Degraded     → Running but reduced performance
  Unknown      → Status cannot be determined (transient)

Platform-initiated issues → Microsoft responsible for fix
User-initiated issues     → You responsible for fix
```

### Monitoring Tools Comparison

| Tool | Scope | Question It Answers |
|------|-------|---------------------|
| **Resource Health** | Your specific resource | "Is MY VM healthy right now?" |
| **Service Health** | Azure platform globally | "Is there an Azure outage in my region?" |
| **Azure Monitor** | Custom metrics + logs | "What was my VM CPU at 3AM yesterday?" |
| **Activity Log** | All operations on resource | "Who deleted my storage account and when?" |
| **Metrics** | Numeric performance data | "Average CPU % over last 24 hours?" |
| **Log Analytics** | Query logs at scale | "Show all 404 errors from my VMs this week" |
| **Application Insights** | App-level monitoring | "Which API endpoint is slowest?" |

### Health Probe (Load Balancer)

```
How Load Balancer decides where to send traffic:

Every 5 seconds:
Load Balancer → sends HTTP GET /health to each VM

VM-1 responds 200 OK  → Healthy ✅ → Gets traffic
VM-2 responds timeout → Unhealthy ❌ → Removed from rotation

Traffic ONLY goes to healthy VMs — automatic failover
```

---

## 17. Azure vs AWS — Full Comparison

| Category | Azure Service | AWS Equivalent |
|----------|--------------|---------------|
| **Compute** | Virtual Machine | EC2 Instance |
| **Auto Scale** | VM Scale Set | Auto Scaling Group |
| **Serverless** | Azure Functions | AWS Lambda |
| **Containers** | AKS | EKS |
| **App Platform** | App Service | Elastic Beanstalk |
| **Object Storage** | Blob Storage | S3 |
| **Block Storage** | Managed Disk | EBS |
| **File Storage** | Azure Files | EFS |
| **Queues** | Queue Storage | SQS |
| **NoSQL** | Cosmos DB | DynamoDB |
| **Relational DB** | Azure SQL | RDS |
| **Private Network** | VNet | VPC |
| **Firewall** | NSG | Security Group |
| **DNS** | Azure DNS | Route 53 |
| **CDN** | Azure CDN / Front Door | CloudFront |
| **Load Balancer L4** | Azure Load Balancer | Classic ELB / NLB |
| **Load Balancer L7** | Application Gateway | ALB |
| **Identity** | Azure AD (Entra ID) | IAM |
| **Secret Store** | Key Vault | Secrets Manager |
| **CI/CD** | Azure DevOps / Pipelines | CodePipeline |
| **Monitoring** | Azure Monitor | CloudWatch |
| **Logging** | Log Analytics | CloudWatch Logs |
| **IaC** | ARM Templates / Bicep | CloudFormation |
| **Multi-cloud IaC** | Terraform (supported) | Terraform (supported) |
| **Secure VM Access** | Azure Bastion | Systems Manager Session Manager |
| **Hybrid** | Azure Arc | AWS Outposts |
| **VM Placement** | Availability Set | Placement Group |
| **AI/ML** | Azure ML + OpenAI | SageMaker + Bedrock |
| **Backup** | Azure Backup | AWS Backup |

---

## 18. Cost Management & Billing

### Check Remaining Credits

```
Path: portal.azure.com → Search "Cost Management + Billing"
      → Subscriptions → Free Trial
      → View remaining $200 credit and usage
```

### Set Budget Alerts

```
Path: Cost Management + Billing → Budgets → + Add

Recommended setup for free account:
  Budget: $5
  Alert at 50% → Email notification
  Alert at 80% → Email notification
  Alert at 100% → Email notification

This way: you get warned BEFORE you're charged anything
```

### Tips to Avoid Accidental Charges

```
Top Rules for Beginners:
────────────────────────────────────────────────────────────────
1. Always use FREE TIER sizes (B1s VM, Standard storage)
2. STOP (Deallocate) VMs when not using them
   → "Stopped" alone is NOT enough — must be "Stopped (Deallocated)"
   → Stopped but NOT deallocated still charges for compute!
3. DELETE resource groups when done with a project
   → Deletes ALL resources inside automatically
4. Only B1s VM is free → Any larger size charges immediately
5. Use Standard HDD, NOT Premium SSD (free tier uses Standard)
6. Set $1 budget alert → notified at first penny of charges
7. Check 12-month expiry → set calendar reminder before it ends
8. Disk still charges even when VM is stopped
   → Deallocate VM to stop compute, but disk still charges a little

Free tier VM must be Stopped (Deallocated) to stop compute billing:
  Running VM         → Compute + Disk charged
  Stopped (VM)       → Compute + Disk charged ← COMMON MISTAKE
  Stopped (Dealloc.) → Only Disk charged (tiny) ← CORRECT way to stop
  Deleted            → Nothing charged
```

---

## 19. Key Terminologies Glossary

| Term | Definition |
|------|-----------|
| **Azure** | Microsoft's cloud computing platform |
| **Subscription** | Billing unit in Azure — all resources belong to one |
| **Resource Group** | Logical container to organize related Azure resources |
| **Region** | Geographic location of Azure data centers |
| **Availability Zone** | Physically separate data center within a region |
| **Availability Set** | HA option: distributes VMs across fault/update domains in one DC |
| **Fault Domain** | VMs sharing same physical rack (power + network) |
| **Update Domain** | VMs rebooted together during Azure planned maintenance |
| **VM** | Virtual Machine — software-based computer on Azure hardware |
| **VMSS** | VM Scale Set — auto-scaling group of identical VMs |
| **NSG** | Network Security Group — firewall rules for VMs/subnets |
| **VNet** | Virtual Network — private network in Azure (AWS VPC equivalent) |
| **Subnet** | Subdivision of a VNet |
| **Public IP** | IP address accessible from the internet |
| **Private IP** | IP address only accessible within the VNet |
| **SSH** | Secure Shell — encrypted protocol to connect to Linux servers |
| **RDP** | Remote Desktop Protocol — GUI connection to Windows servers |
| **Azure Bastion** | Managed secure browser-based VM access (no public IP needed) |
| **Serial Console** | Emergency low-level VM access via serial port |
| **VM Extension** | Agent running automation/monitoring on VM without manual login |
| **Azure Blob** | Binary Large Object — any file stored in Azure Blob Storage |
| **Container** | Top-level organizer in Blob Storage (S3 Bucket equivalent) |
| **Storage Account** | Azure resource holding all storage services under one namespace |
| **Root Volume** | Primary OS disk of a VM — must be block storage |
| **Block Storage** | Storage with addressable blocks — bootable (Managed Disk, EBS) |
| **Object Storage** | HTTP-based file storage — non-bootable (Blob, S3) |
| **Hot Tier** | Highest cost, lowest access cost, instant retrieval |
| **Cool Tier** | 30-day min, lower storage cost, higher access cost |
| **Cold Tier** | 90-day min, lower than Cool, instant retrieval |
| **Archive Tier** | Lowest cost, offline storage, requires rehydration (hours) |
| **Rehydration** | Process of moving blob from Archive to Hot/Cool before reading |
| **Lifecycle Policy** | Automated rules to move/delete data based on age or access |
| **SAS Token** | Shared Access Signature — time-limited URL to access a blob |
| **Azure AD / Entra ID** | Microsoft's cloud identity and access management (IAM) |
| **RBAC** | Role-Based Access Control — assign permissions by role |
| **Key Vault** | Secure store for secrets, encryption keys, certificates |
| **Azure Monitor** | Platform for metrics, logs, and alerts |
| **Resource Health** | Azure portal feature showing health of a specific resource |
| **SLA** | Service Level Agreement — Microsoft's uptime guarantee |
| **Hybrid Cloud** | Combination of on-premise servers and Azure cloud |
| **Azure Arc** | Manage on-premise servers from Azure portal |
| **ExpressRoute** | Dedicated private circuit from office to Azure (not internet) |
| **MobaXterm** | Windows SSH/RDP terminal with GUI, SFTP, X11 forwarding |
| **Deallocated** | VM stopped and compute resources released — no compute charge |
| **Spot VM** | Low-cost VM that Azure can evict when needed (up to 90% cheaper) |
| **Reserved Instance** | Pre-purchase VM capacity for 1–3 years for up to 72% discount |

---

*Happy Cloud Learning! ☁️ — portal.azure.com — learn.microsoft.com*
