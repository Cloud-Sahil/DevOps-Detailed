# Microsoft Azure — Interview Questions

> **Coverage:** Simple · Advantage-Based · Scenario-Based  
> **Total Questions:** 45  
> **Level:** Beginner → Production Engineer

---

## Table of Contents

1. [Simple / Conceptual Questions](#1-simple--conceptual-questions) — Q1–Q15
2. [Advantage-Based Questions](#2-advantage-based-questions) — Q16–Q30
3. [Scenario-Based Questions](#3-scenario-based-questions) — Q31–Q45
4. [Quick-Fire Q&A](#4-quick-fire-qa)
5. [Interview Tips](#5-interview-tips)

---

## 1. Simple / Conceptual Questions

*Tests whether you understand fundamental Azure concepts clearly.*

---

**Q1. What is Microsoft Azure?**

**A:** Microsoft Azure is a cloud computing platform owned and operated by Microsoft. It provides a wide range of cloud services including compute (Virtual Machines, App Services), storage (Blob, Files, Disks), databases (SQL, Cosmos DB), networking, AI/ML, and DevOps tools. Azure competes with AWS (Amazon) and GCP (Google Cloud) as one of the top 3 global cloud providers. It is especially popular in enterprises due to its deep integration with Microsoft products like Active Directory, Office 365, and Windows Server.

**Key points:** Open-source friendly · 60+ global regions · 90+ compliance certifications · Used by 95% of Fortune 500

---

**Q2. What is the difference between a Subscription, a Resource Group, and a Resource in Azure?**

**A:**

| Term | What it is | Analogy |
|------|-----------|---------|
| **Subscription** | Billing unit — all costs are tracked here | Company credit card account |
| **Resource Group** | Logical container to organize related resources | Project folder |
| **Resource** | Any service you create (VM, storage, DB) | File inside the folder |

```
Subscription: Free Trial
  └── Resource Group: webapp-prod-rg
        ├── Resource: Virtual Machine (web-server)
        ├── Resource: Storage Account (webappstorage)
        └── Resource: Virtual Network (webapp-vnet)
```

Best practice: One Resource Group per project. Deleting a resource group deletes everything inside it.

---

**Q3. What is an Azure Region and an Availability Zone? How are they different?**

**A:**

- **Region:** A geographic location where Microsoft has data centers (e.g., Central India = Pune, East US = Virginia). Azure has 60+ regions globally.

- **Availability Zone (AZ):** A physically separate data center inside a region. Each zone has its own power, cooling, and networking. Central India has 3 AZs.

```
Region: Central India
  ├── Zone 1 (DC-A) — Own power, cooling, network
  ├── Zone 2 (DC-B) — Completely separate
  └── Zone 3 (DC-C) — Completely separate
```

**Key difference:** A region is a city-level location. An AZ is a building-level separation within that city.

Not every region has AZs — Central India (Pune) has 3 AZs, West India (Mumbai) has none.

---

**Q4. What is the difference between Availability Sets and Availability Zones?**

**A:**

| Feature | Availability Set | Availability Zone |
|---------|:---------------:|:-----------------:|
| Scope | Within ONE data center | Across separate data centers |
| Protects from | Rack failure (hardware) | Entire DC failure |
| Regions | All Azure regions | Only AZ-enabled regions |
| SLA | 99.95% | 99.99% |
| Cost | Free (pay only for VMs) | Higher (multiple VMs across zones) |
| Best for | Non-AZ regions (Mumbai, Chennai) | Production in AZ regions (Pune) |

Availability Sets use **Fault Domains** (separate racks) and **Update Domains** (separate maintenance batches) within a single data center.

---

**Q5. What are the four methods to connect to an Azure VM? When would you use each?**

**A:**

| Method | OS | Best For |
|--------|----|---------| 
| **Azure Bastion** | Linux + Windows | Secure production access — no public IP needed |
| **SSH** | Linux only | Daily Linux VM management — free, fast |
| **RDP** | Windows only | Windows VM GUI access |
| **Serial Console** | Both | Emergency access when OS/network is down |

**When to use each:**
- Bastion → production VMs where security is critical (extra cost ~$0.19/hr)
- SSH → any Linux dev/test/production VM with public IP
- RDP → Windows Server administration
- Serial Console → VM stuck in boot loop, wrong firewall rules, OS crash recovery

---

**Q6. What is the difference between Blob Storage and Managed Disk in Azure?**

**A:**

| Feature | Blob Storage (Object) | Managed Disk (Block) |
|---------|:--------------------:|:-------------------:|
| Access Method | HTTP REST API | Block-level direct I/O |
| Bootable | ❌ No | ✅ Yes |
| File System | None | ext4, NTFS, XFS |
| Random R/W | No (whole object) | Yes (individual blocks) |
| Best For | Images, videos, backups | OS disks, data disks |
| AWS Equivalent | S3 | EBS |

**Why can't you boot an OS from Blob Storage?** Operating systems require millisecond-level random block I/O during boot and runtime. Blob Storage works via HTTP — it retrieves entire objects, not individual disk blocks. Fundamentally incompatible for booting.

---

**Q7. What is a VM Scale Set (VMSS) and when would you use it over a single VM?**

**A:** A VM Scale Set is a group of identical VMs that automatically scales out (adds VMs) when demand increases and scales in (removes VMs) when demand decreases.

**Use VMSS when:**
- Traffic is unpredictable (e-commerce, event-driven apps)
- Application needs to stay available even if VMs fail
- You need to manage many VMs as one unit
- Cost optimization: only pay for what you use

**Use single VM when:**
- Dev/test/learning environments
- Simple personal projects
- Traffic is constant and small
- Cost is the top priority

```
Scale Rules:
  CPU > 70% for 5 min  → Add 2 VMs
  CPU < 20% for 10 min → Remove 1 VM
  Min: 2 VMs    Max: 10 VMs
```

---

**Q8. What are the four Azure Blob Storage tiers and what is "rehydration"?**

**A:**

| Tier | Cost | Retrieval | Min Duration | Best For |
|------|:----:|:---------:|:------------:|---------|
| **Hot** | Highest | Instant | None | Daily-access data |
| **Cool** | ~50% less | Instant | 30 days | Monthly access |
| **Cold** | Lower | Instant | 90 days | Quarterly access |
| **Archive** | Lowest | **Hours** | 180 days | Annual/compliance |

**Rehydration** = The process of moving an archived blob to Hot or Cool tier before it can be read. Archive blobs are "offline" and cannot be read directly. Rehydration takes:
- Standard priority: up to 15 hours
- High priority: under 1 hour (higher cost)

Early deletion fees apply if you delete before the minimum duration.

---

**Q9. What is an NSG (Network Security Group) and how does it work?**

**A:** An NSG is Azure's firewall that controls inbound and outbound network traffic for VMs and subnets using rules.

```
NSG Rule Structure:
  Priority + Direction + Port + Protocol + Source + Action

Example: Allow SSH from my IP only
  Priority: 100
  Direction: Inbound
  Port: 22
  Protocol: TCP
  Source: 203.0.113.5 (my IP)
  Action: Allow

Lower priority number = evaluated first (100 before 200)
Default: All inbound blocked, all outbound allowed
```

NSG can be applied at two levels: VM NIC level and Subnet level. Both apply if both exist.

---

**Q10. What is Azure Bastion and what problem does it solve?**

**A:** Azure Bastion is a fully managed service that provides browser-based SSH and RDP access to VMs directly from the Azure Portal — without the VM needing a public IP address.

**Problem it solves:** Traditionally, to SSH/RDP into a VM you need a public IP and open port 22/3389. This exposes your VM to the internet and potential brute-force attacks. Azure Bastion acts as a secure jump host managed by Microsoft, where your VM stays private and only Bastion has internet access.

**Benefits:** No public IP needed · All traffic TLS encrypted · No software installation · Works from any browser · Protected by Azure AD · Audit log of all sessions

**Cost:** ~$0.19/hour — not free.

---

**Q11. What is a VM Extension? Give an example.**

**A:** A VM Extension is a lightweight software agent that runs inside a VM to automate post-deployment tasks without requiring manual login. Extensions are managed by the Azure VM Agent pre-installed on all Azure Marketplace VMs.

**Example — Custom Script Extension:**
```bash
#!/bin/bash
# This runs automatically on VM after deployment — no SSH needed
apt-get update -y && apt-get install -y nginx
systemctl start nginx && systemctl enable nginx
echo "<h1>Auto-deployed via Extension</h1>" > /var/www/html/index.html
```

Store this script in Blob Storage, attach the extension, and Azure runs it silently.

**Other examples:** Azure Monitor Agent (logs/metrics), Antimalware Extension (Windows Defender), Key Vault Extension (auto-rotate certs), NVIDIA GPU Driver (auto-install drivers).

---

**Q12. What is Resource Health in Azure and how is it different from Azure Monitor?**

**A:**

| Tool | Scope | Question Answered |
|------|-------|------------------|
| **Resource Health** | Your specific resource | "Is MY VM healthy right now?" |
| **Service Health** | Azure platform globally | "Is there an Azure outage in my region?" |
| **Azure Monitor** | Custom metrics/logs | "What was my VM CPU at 3AM yesterday?" |
| **Activity Log** | Operations on a resource | "Who deleted my storage account?" |

**Resource Health path:** VM → Help (left sidebar) → Resource Health

Status values: Available / Unavailable / Degraded / Unknown

Resource Health distinguishes: **Platform-initiated** issues (Microsoft's fault) vs **User-initiated** issues (your fault — misconfiguration, application error).

---

**Q13. What is the difference between "Stopped" and "Stopped (Deallocated)" for a VM?**

**A:** This is one of the most important cost-related concepts for Azure beginners.

| State | Compute Charged? | Disk Charged? | IP Released? |
|-------|:----------------:|:------------:|:------------:|
| Running | ✅ Yes | ✅ Yes | No |
| **Stopped** (via OS shutdown) | ✅ **Still charging!** | ✅ Yes | No |
| **Stopped (Deallocated)** | ❌ No | ✅ Yes (small) | Dynamic IP released |
| Deleted | ❌ No | ❌ No | Yes |

**Key rule:** To stop compute billing, you must **Deallocate** the VM — not just stop it. Use Azure Portal → VM → Stop button (which deallocates), or: `az vm deallocate --resource-group RG --name VMName`

---

**Q14. What is Azure Active Directory (Azure AD / Entra ID)?**

**A:** Azure AD (now called Microsoft Entra ID) is Microsoft's cloud-based identity and access management service. It is the equivalent of AWS IAM.

**Core functions:**
- User and group management
- Authentication (who you are)
- Authorization via RBAC (what you can do)
- Multi-Factor Authentication (MFA)
- Single Sign-On (SSO) to thousands of apps
- App registrations for OAuth2/OIDC

**RBAC roles:** Owner (full), Contributor (manage resources, no permissions), Reader (view only)

**Key difference from AWS IAM:** Azure AD is tenant-based (your organization gets a tenant). Storage blob operations need a separate "Storage Blob Data Contributor" role even if you're subscription Owner.

---

**Q15. Can you shrink (decrease the size of) an Azure Managed Disk?**

**A:** No. You cannot decrease the size of an Azure Managed Disk once it has been increased. This is by design.

**Why:** Shrinking a disk risks data corruption because the filesystem's partition table and data layout are written based on the disk size. Reducing the physical disk size without shrinking the filesystem first would destroy data.

```
Allowed:     50 GB → 100 GB  ✅ (expand — straightforward)
Not allowed: 100 GB → 50 GB  ❌ (shrink — not supported)

Workaround if you must downsize:
  1. Create new smaller disk
  2. Copy data from old disk to new disk
  3. Swap disks on the VM
  4. Delete old large disk
```

Same rule applies to AWS EBS volumes.

---

## 2. Advantage-Based Questions

*Tests whether you understand WHY to use Azure features and their business value.*

---

**Q16. What are the key advantages of using Azure over managing on-premise servers?**

**A:**

| Advantage | On-Premise | Azure |
|-----------|-----------|-------|
| **Upfront cost** | Buy servers ($thousands) | Pay-as-you-go ($0 to start) |
| **Scaling** | Buy more hardware (weeks) | Scale in minutes |
| **Maintenance** | Your team's responsibility | Microsoft manages hardware |
| **Global reach** | Build data centers (millions) | 60+ regions, deploy in minutes |
| **Reliability** | Single data center | Multi-AZ, 99.99% SLA |
| **Disaster recovery** | Complex, expensive | Built-in with Azure Site Recovery |
| **Security patches** | Manual, often delayed | Azure handles hypervisor/infra |
| **Innovation** | New tech requires new hardware | AI, ML, Kubernetes available instantly |

**Best for Azure:**
- Variable/unpredictable workloads (only pay for what you use)
- Global user base (deploy in any region)
- Startups (no upfront CapEx — just OpEx)

---

**Q17. What are the advantages of using Azure Availability Zones over a single VM?**

**A:**

**Single VM problem:** If the physical hardware, power supply, or the entire data center fails → application is completely down.

**Availability Zones solve this:**
- VMs spread across physically separate DCs within same region
- Zone 1, 2, 3 each have independent power and networking
- If Zone 1's entire data center goes offline → VM in Zone 2 and 3 continue serving traffic
- SLA improves from 99.9% → 99.99% (52 min downtime/year vs 8.7 hours)

**Business value:** For an e-commerce site making ₹50,000/hour, 99.99% SLA vs 99.9% SLA means the difference between ~52 minutes of potential revenue loss vs ~8.7 hours per year.

---

**Q18. What are the advantages of using Azure Blob Storage tiers (Hot/Cool/Cold/Archive)?**

**A:** Storage tiering is Azure's automatic cost-optimization mechanism for data that is accessed with different frequencies.

```
Without tiering: ALL data stored at Hot price ($0.018/GB)
With tiering:
  Active data (daily) → Hot:    $0.018/GB
  Monthly data        → Cool:   $0.010/GB  (~44% savings)
  Quarterly data      → Cold:   $0.005/GB  (~72% savings)
  Annual/compliance   → Archive: $0.001/GB  (~94% savings)
```

**Real-world business advantage:**
- Compliance data retained 7 years → Archive = 94% cheaper than Hot
- Log files not accessed after 30 days → Auto-move to Cool via Lifecycle Policy
- Zero manual effort with Lifecycle Management policies
- Regulatory requirements met (data retained) without paying full price

---

**Q19. What are the advantages of Azure VM Scale Sets (VMSS) over manually managing multiple VMs?**

**A:**

| Without VMSS (Manual) | With VMSS |
|----------------------|-----------|
| Add VMs manually when traffic spikes | Auto-scales based on CPU/custom metrics |
| Scale-up takes 5–30 min manually | Scale-up triggers automatically in 2–5 min |
| Must manually configure each new VM | All VMs identical — configured once |
| Must manually remove VMs when idle | Auto-removes idle VMs (save cost) |
| If a VM fails, manual replacement | Self-healing — failed VMs auto-replaced |
| Load balancing set up manually | Built-in load distribution |
| 1,000 VMs = 1,000 manual changes | 1,000 VMs = change once, apply to all |

**Business value:** E-commerce platform during Diwali sale — VMSS automatically handles 10x traffic spike at 2 AM and scales back down by morning, paying only for the extra VMs during the peak.

---

**Q20. What are the advantages of using Azure Bastion over opening port 22/3389 directly?**

**A:**

| Opening port 22 directly | Using Azure Bastion |
|--------------------------|---------------------|
| VM has public IP → exposed to internet | VM has NO public IP |
| Port 22 scanned by bots constantly | No exposed SSH port to scan |
| Brute-force attacks on password | Azure AD authentication protects Bastion |
| Must manage SSH keys on each VM | Centralized access via Azure Portal |
| Difficult to audit who accessed what | Full session logs and audit trail |
| Need SSH client installed locally | Just a browser — works anywhere |
| NSG must allow SSH from internet | Bastion subnet is the only entry point |

**Security advantage:** Most cloud VM breaches happen through exposed SSH/RDP ports. Bastion eliminates this entire attack surface.

---

**Q21. What is the advantage of using Azure Key Vault over storing secrets in application config files?**

**A:**

**Problem with secrets in config files:**
```
appsettings.json:
  "ConnectionString": "Server=db;Password=SuperSecret123"
                                              ↑ 
  If this file is committed to GitHub → secret is leaked permanently
  If app is compromised → attacker reads all secrets instantly
```

**Advantages of Key Vault:**
- Secrets never appear in code or config files
- Hardware Security Module (HSM) backed — physically secure
- Automatic secret rotation (no human involvement)
- Access controlled per-identity via RBAC
- Full audit log — every secret access is logged
- Integration with VMs, App Service, AKS via Managed Identity (no credentials needed)
- Centralized — one place to update secrets for all apps

---

**Q22. What are the advantages of Azure Resource Groups?**

**A:**

**Without Resource Groups:** All resources scattered — hard to track, manage, and delete.

**Advantages:**
1. **Lifecycle management** → Delete one resource group = delete ALL resources of a project in one click
2. **Access control** → Apply RBAC once at resource group level → applies to all resources inside
3. **Cost visibility** → Filter costs by resource group → see exactly what each project costs
4. **Tagging** → Tag the group once, all resources inherit the cost center
5. **Deployment** → ARM/Bicep templates deploy entire resource groups atomically
6. **Geographic constraint** → Resource group has a "location" (metadata store location, not resource location)

**Best practice:** One resource group per project/environment. `webapp-dev-rg`, `webapp-prod-rg` — not one giant resource group for everything.

---

**Q23. What is the advantage of "Pipeline as Code" in Azure DevOps vs configuring pipelines through the UI?**

**A:**

**UI-configured pipelines:**
- Configuration lives only in Azure DevOps — not tracked
- No peer review of pipeline changes
- If Azure DevOps is deleted → pipeline config lost
- Different pipelines for different teams — inconsistency

**Pipeline as Code (azure-pipelines.yml):**
- Pipeline definition stored in Git with the application code
- Every change is tracked, reviewable, and reversible via PR
- New team members get the correct pipeline instantly by cloning the repo
- Same pipeline runs identically in all branches/environments
- Disaster recovery: re-import from Git if Azure DevOps is reset
- Audit trail: see who changed what in the pipeline and when

---

**Q24. What is the advantage of using Azure Managed Identity over service accounts with passwords?**

**A:**

**Traditional service account:**
```
App needs to access Blob Storage:
  1. Create service account with password
  2. Store password in app config
  3. Rotate password manually every 90 days
  4. Risk: password in config file, accidentally committed to Git
```

**Managed Identity:**
```
App needs to access Blob Storage:
  1. Enable Managed Identity on VM/App Service
  2. Grant that identity the "Storage Blob Data Reader" role
  3. App calls Azure AD automatically → gets token → accesses blob
  4. Zero credentials — nothing to store, nothing to rotate, nothing to leak
```

**Benefits:** No credentials to manage, no rotation needed, no secret storage risk, Azure manages the identity lifecycle automatically.

---

**Q25. What are the advantages of Azure over AWS from a hybrid cloud perspective?**

**A:**

Azure has a significant advantage in hybrid cloud (mixing on-premise with cloud):

1. **Azure Arc** → Manage on-premise servers from the Azure Portal using the same tools as cloud VMs
2. **Azure AD (Entra ID)** → Direct extension of on-premise Active Directory → same users, groups, policies
3. **Azure ExpressRoute** → Dedicated private circuit from office to Azure (not internet) → consistent latency
4. **Azure File Sync** → Sync on-premise Windows file servers with Azure Files automatically
5. **Azure Site Recovery** → Replicate on-premise VMs to Azure for disaster recovery
6. **Azure Migrate** → Free assessment and migration tool for on-premise workloads

**Why this matters:** 90%+ of enterprise companies have existing on-premise infrastructure with Active Directory and Windows. Azure integrates natively; AWS requires additional tooling.

---

**Q26. What is the advantage of using Azure Static IP vs Dynamic IP for production VMs?**

**A:**

**Dynamic Public IP:**
- Changes every time the VM is Stopped (Deallocated)
- Free when attached to running VM
- Breaks DNS, application configs, firewall whitelist entries after restart

**Static Public IP:**
- Never changes — yours until explicitly released
- Small charge when not attached to a running resource
- Required for: production VMs, any service that needs consistent IP, DNS A records, IP whitelisting by partners

**Rule:** Always use Static IP for production. Use Dynamic only for dev/test.

---

**Q27. What is the advantage of Azure Lifecycle Management Policies for Blob Storage?**

**A:** Lifecycle policies automate the movement of blobs between storage tiers based on age or last access time — with zero manual intervention.

**Business scenario:**
```
Without lifecycle policy:
  Operations team manually reviews storage monthly
  Moves old files to Archive tier
  Hours of work, prone to human error

With lifecycle policy (configured once):
  Day 1:    File uploaded → Hot tier (active)
  Day 30:   Automatically moved to Cool tier → 44% cheaper
  Day 90:   Automatically moved to Cold tier → 72% cheaper
  Day 365:  Automatically moved to Archive → 94% cheaper
  Day 2555 (7 years): Automatically deleted (compliance period end)
```

**Advantages:** Zero operational overhead · Consistent cost optimization · Enforces data retention policies automatically · AWS equivalent: S3 Lifecycle Rules

---

**Q28. What are the advantages of using Azure Monitor and Alerts?**

**A:** Azure Monitor provides centralized observability for all Azure resources.

**What it enables:**
- **Proactive alerts** → Get notified BEFORE users do: "CPU > 80% for 5 min" → SMS/email/Teams
- **Historical analysis** → Query 90 days of metrics to understand trends
- **Root cause analysis** → Correlate VM metrics + application logs + NSG flow logs
- **Auto-healing** → Alert → Azure Automation runbook → auto-restart service
- **Cost alerts** → Budget alert at $5 → notified before charges accumulate
- **SLA tracking** → Measure actual uptime vs committed SLA

**Business value:** 80% of production incidents can be predicted and prevented before user impact with proper monitoring. Azure Monitor is the difference between reactive firefighting and proactive operations.

---

**Q29. What is the advantage of using Azure's Reserved Instances vs Pay-As-You-Go?**

**A:**

| Model | Commitment | Savings | Best For |
|-------|:----------:|:-------:|---------|
| Pay-As-You-Go | None | 0% | Dev/test, unpredictable |
| 1-Year Reserved | 1 year | ~40% | Stable production workloads |
| 3-Year Reserved | 3 years | ~72% | Long-term stable workloads |
| Spot VM | None (evictable) | ~90% | Batch jobs, fault-tolerant workloads |

**Example:** A `Standard_D4s_v3` VM at PAYG = ~$200/month. 3-year reserved = ~$56/month. On 10 VMs that's $1,440/month vs $4,440/month savings → $36,000/year.

**When NOT to use Reserved Instances:** Workloads that might be scaled down or changed — you commit to paying regardless of usage.

---

**Q30. What are the advantages of Azure DevOps over traditional software development workflows?**

**A:**

**Traditional workflow problems:**
- Code integration happens weeks apart → "integration hell"
- Manual testing → bugs found late (expensive to fix)
- Manual deployments → inconsistent, error-prone
- No visibility into what's deployed where

**Azure DevOps advantages:**
- **Azure Repos** → Git hosting with PR reviews, branch policies
- **Azure Pipelines** → Automated build + test + deploy on every commit
- **Azure Boards** → Work item tracking (bugs, features, sprints)
- **Azure Artifacts** → Private package registry (npm, Maven, NuGet)
- **Azure Test Plans** → Manual + automated test management
- **Free for 5 users** → Small teams get everything for free

**Business value:** Teams using CI/CD deploy 200x more frequently, have 24x faster recovery from failures, and 3x lower change failure rate (DORA metrics).

---

## 3. Scenario-Based Questions

*Tests ability to apply Azure knowledge to real-world problems.*

---

**Q31. Scenario: Your company's Azure VM hosting a critical web application suddenly becomes unreachable. Users are getting "connection timed out." What steps do you take to diagnose and fix it?**

**A:**

**Step 1 — Check Azure platform (2 min):**
```
status.azure.com → Any incidents in your region (Central India)?
Portal → VM → Resource Health → Is it "Available"?
```

**Step 2 — Check VM state (1 min):**
```
Portal → VM → Overview → Status: Running? (not Stopped)
If stopped → Start VM immediately
```

**Step 3 — Check NSG rules (3 min):**
```
Portal → VM → Networking → Inbound port rules
Does port 80/443 have an Allow rule? Did someone accidentally delete it?
Fix: Add Allow TCP 80 from Any
```

**Step 4 — Check if web server is running (SSH in):**
```bash
ssh azureuser@<vm-ip>
sudo systemctl status nginx      # or apache2
# If not running:
sudo systemctl start nginx
sudo journalctl -u nginx -n 50   # Check error logs
```

**Step 5 — Check disk space:**
```bash
df -h
# If full → clean logs, expand disk, or move data
```

**Step 6 — Check application logs:**
```bash
tail -100 /var/log/nginx/error.log
tail -100 /var/log/app/application.log
```

**Prevention:** Set up Azure Monitor alerts — CPU > 80%, disk > 85%, HTTP 5xx errors → notify team immediately.

---

**Q32. Scenario: Your manager asks you to design a highly available architecture for a web application that must handle 10,000 concurrent users in India with 99.99% uptime. What do you design?**

**A:**

```
Architecture Design:
────────────────────────────────────────────────────────────────

Internet Users (India)
        │
        ▼
Azure Application Gateway (Layer 7 LB + WAF)
        │
        ▼
VM Scale Set (Region: Central India)
  ├── Zone 1: VM-1, VM-2
  ├── Zone 2: VM-3, VM-4
  └── Zone 3: VM-5, VM-6
  (Min 2 VMs, Max 20 VMs — auto-scale on CPU 70%)
        │
        ▼
Azure Cache for Redis (session caching)
        │
        ▼
Azure SQL Database (General Purpose, Zone-Redundant)
        │
        ▼
Azure Blob Storage (Hot tier for assets, CDN in front)
```

**Key decisions:**
- Central India → has 3 AZs → 99.99% SLA achievable
- VMSS → handles traffic spikes automatically (scale out during peak)
- Application Gateway → SSL termination, WAF protection, URL-based routing
- Zone-redundant SQL → database continues if one DC fails
- Azure CDN → static assets served from edge nodes → faster for users
- Azure Monitor → alerts on all components with auto-remediation

---

**Q33. Scenario: You have 100 TB of log files in Azure Blob Storage (Hot tier). The files older than 30 days are rarely accessed. Files older than 1 year are accessed only for compliance audits. How do you optimize storage costs?**

**A:**

**Current situation (all Hot tier):**
```
100 TB × $0.018/GB = ~$1,800/month
```

**Solution — Implement Lifecycle Management Policy:**

```json
{
  "rules": [
    {
      "name": "log-tiering-rule",
      "filters": { "blobTypes": ["blockBlob"], "prefixMatch": ["logs/"] },
      "actions": {
        "baseBlob": {
          "tierToCool": { "daysAfterModificationGreaterThan": 30 },
          "tierToCold": { "daysAfterModificationGreaterThan": 90 },
          "tierToArchive": { "daysAfterModificationGreaterThan": 365 },
          "delete": { "daysAfterModificationGreaterThan": 2555 }
        }
      }
    }
  ]
}
```

**Estimated cost after optimization:**
```
Recent 30 days (Hot):  ~20 TB × $0.018 = $368/month
30-90 days (Cool):     ~20 TB × $0.010 = $200/month  
90-365 days (Cold):    ~30 TB × $0.005 = $150/month
1+ year (Archive):     ~30 TB × $0.001 = $30/month
Total:                                    $748/month ← 58% cheaper
```

Zero manual work — policy runs automatically daily.

---

**Q34. Scenario: A developer accidentally deleted an important file from an Azure Blob Storage container. How do you recover it?**

**A:**

**Option 1: Blob Soft Delete (if enabled):**
```
Portal → Storage Account → Container → Show deleted blobs
→ Find the file → Undelete

This works for up to 30 days after deletion (if soft delete was enabled)
```

**Option 2: Blob Versioning (if enabled):**
```
Portal → Storage Account → Container → Select blob
→ Versions tab → Select previous version → Promote to current
```

**Option 3: Restore from Azure Backup:**
```
If backup was configured → Azure Backup Vault → Restore
```

**If none of the above were enabled:**
```
File is permanently lost — cannot recover

Prevention for the future:
  1. Enable Soft Delete (30-day retention): Storage → Data protection → Enable soft delete
  2. Enable Versioning: Storage → Data protection → Enable versioning
  3. Enable Azure Backup for critical containers
  4. Apply RBAC: Restrict delete permissions to only specific roles
```

---

**Q35. Scenario: Your Azure VM in West India (Mumbai) is running a production app. A typhoon warning is issued for Mumbai. How do you ensure business continuity?**

**A:**

**Immediate action (if you prepared ahead of time):**
```
Azure Site Recovery (already configured):
  VM automatically replicating to Central India (Pune) every 30 sec
  Initiate failover: Portal → Recovery Services Vault → Failover
  → Select VM → Select recovery point → Confirm
  Application continues running in Central India within minutes
```

**If you haven't prepared:**
```
Step 1: Create snapshot of VM disk immediately
        VM → Disks → OS Disk → Create snapshot

Step 2: Copy snapshot to Central India
        Snapshot → Export → Copy to new region

Step 3: Create new VM in Central India from snapshot

Step 4: Update DNS to point to new IP
        Azure DNS or your DNS → Update A record TTL to 60 sec first
```

**Best practice architecture:**
- Always deploy production across 2 regions (active-active or active-passive)
- Azure Traffic Manager → DNS-level failover between regions automatically
- Central India (3 AZs) → primary; West India → secondary
- RTO (Recovery Time Objective): minutes with ASR, hours without

---

**Q36. Scenario: Your team complains that it takes 45 minutes to provision a new developer VM every time. How would you automate this?**

**A:**

**Solution — Infrastructure as Code + Azure DevOps:**

**Option 1: Azure Bicep template + Pipeline:**
```bicep
// devvm.bicep — defines the entire dev VM
resource devVM 'Microsoft.Compute/virtualMachines@2023-03-01' = {
  name: 'devvm-${uniqueString(resourceGroup().id)}'
  location: 'centralindia'
  properties: {
    hardwareProfile: { vmSize: 'Standard_D2s_v3' }
    storageProfile: {
      imageReference: { publisher: 'Canonical', offer: 'UbuntuServer', sku: '22.04-LTS', version: 'latest' }
    }
    // + networking, SSH keys, extensions for dev tools
  }
}
```

**Pipeline (azure-pipelines.yml):**
```yaml
trigger: none  # Manual trigger
parameters:
  - name: developerName
    type: string

steps:
  - task: AzureCLI@2
    inputs:
      scriptType: bash
      scriptLocation: inlineScript
      inlineScript: |
        az deployment group create \
          --resource-group dev-vms-rg \
          --template-file devvm.bicep \
          --parameters developerName=${{ parameters.developerName }}
```

**Result:** Developer clicks "Run Pipeline" → enters their name → VM provisioned in 3–5 minutes with all tools pre-installed via VM Extension. 45 minutes → 5 minutes.

---

**Q37. Scenario: You notice your Azure bill is $300 this month but you expected $50. How do you find what's causing the overcharge?**

**A:**

**Step 1 — Cost breakdown:**
```
Portal → Cost Management + Billing → Cost analysis
  Group by: Service name → See which service is most expensive
  Group by: Resource → See which specific resource costs most
  Filter by: Last 30 days
```

**Step 2 — Common culprits to check:**
```bash
# Check for running VMs you forgot
az vm list --output table --query "[].{Name:name,Size:hardwareProfile.vmSize,Status:provisioningState}"

# Check for non-deallocated stopped VMs
az vm list --output table

# Check storage account usage
az storage account list --output table

# Check for orphaned disks (no VM attached)
az disk list --query "[?diskState=='Unattached']" --output table

# Check Public IPs not attached to anything
az network public-ip list --query "[?ipConfiguration==null]" --output table
```

**Step 3 — Check for unexpected outbound data:**
```
Storage Account → Monitoring → Metrics → Egress
VM → Monitoring → Metrics → Network Out Total
```

**Common findings:** VM in wrong size (D4 instead of B1) · Stopped but not deallocated VMs · Static public IPs accumulating charges · Data egress beyond free tier · Premium SSD instead of Standard · Orphaned disks no one deleted

**Prevention:** Set $10 budget alert → Get emailed at 80% ($8) → Investigate before reaching $10.

---

**Q38. Scenario: Your company wants to migrate 50 on-premise physical servers to Azure over 6 months. How do you approach this?**

**A:**

**Phase 1 — Assessment (Month 1):**
```
1. Install Azure Migrate appliance on-premise
2. Migrate → Assess → Create assessment of all 50 servers
3. Get recommended Azure VM sizes (right-sizing)
4. Get estimated monthly cost in Azure
5. Identify which servers have dependencies (communicate before migrating)
```

**Phase 2 — Pilot Migration (Month 2):**
```
1. Pick 3-5 non-critical servers (dev/test)
2. Migrate using Azure Migrate:
   Portal → Migrate → Replicate → Configure → Test migration → Validate
3. Validate application functionality
4. Update runbook/procedures based on learnings
```

**Phase 3 — Wave Migrations (Month 3–5):**
```
Wave 1: Dev/Test servers (lowest risk)
Wave 2: Internal business apps
Wave 3: Customer-facing non-critical apps
Wave 4: Business-critical production apps (most prepared)
```

**Phase 4 — Production Cutover (Month 6):**
```
1. Replicate production servers (ASR running continuously)
2. Schedule maintenance window (low traffic time)
3. Final sync → Failover → Update DNS → Validate → Decommission on-prem
```

**Tools used:** Azure Migrate · Azure Site Recovery · Azure ExpressRoute (private connection) · Azure Arc (manage during transition) · Azure Monitor (verify performance post-migration)

---

**Q39. Scenario: A client asks for a storage solution to archive 5 years of financial records (rarely accessed, but may be audited at any time), while keeping the current year's records instantly accessible. What do you design?**

**A:**

```
Current Year Records (daily access):
  → Azure Blob Storage: HOT TIER
  → Instant access, ~$0.018/GB
  → Container: "financial-records-current"

Previous Year Records (occasional access):
  → Azure Blob Storage: COOL TIER
  → Instant access, ~$0.010/GB, 30-day min
  → Container: "financial-records-archive"

2-4 Year Old Records (rare access — audit only):
  → Azure Blob Storage: COLD TIER
  → Instant access, ~$0.005/GB, 90-day min
  → Container: "financial-records-cold"

5 Year Records (compliance hold, almost never accessed):
  → Azure Blob Storage: ARCHIVE TIER
  → Requires rehydration (up to 15 hours), ~$0.001/GB
  → Container: "financial-records-compliance"

Automation:
  → Lifecycle Policy: auto-move on Jan 1 each year
  → Auto-delete after 7 years (compliance period)
```

**Additional requirements for financial data:**
- Enable Blob Versioning (accidental delete protection)
- Enable Soft Delete (30 day recovery window)
- Enable immutability policy (WORM — Write Once Read Many) for compliance
- Apply RBAC: Finance team = Reader, Auditors = Reader (specific container), IT Admin = Contributor
- Enable Azure Defender for Storage (threat detection)
- Store encryption keys in Azure Key Vault (customer-managed keys)

---

**Q40. Scenario: You deploy an application to a VM. It works when you SSH in and run it manually, but when you add a Custom Script Extension to run on startup, it fails. How do you debug?**

**A:**

**Step 1 — Check extension logs on the VM:**
```bash
# Linux:
cat /var/log/azure/custom-script/handler.log
cat /var/lib/waagent/custom-script/download/0/stdout
cat /var/lib/waagent/custom-script/download/0/stderr

# Windows PowerShell:
Get-Content "C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.CustomScriptExtension\*\CustomScriptHandler.log"
```

**Step 2 — Common causes:**
```
Cause 1: Script works as your user (azureuser) but extension runs as ROOT
  Root has different PATH, different environment variables
  Fix: Use absolute paths in script: /usr/bin/npm instead of just npm

Cause 2: No internet access for downloading packages
  Extension runs at boot — network might not be ready
  Fix: Add sleep 30 at start of script; verify internet from VM

Cause 3: Script file not accessible from storage URL
  SAS token expired or storage is private
  Fix: Use public blob or regenerate SAS token with longer expiry

Cause 4: Extension timeout (90 seconds default)
  Long-running scripts exceed limit
  Fix: Run script in background: nohup ./install.sh &> /var/log/install.log &

Cause 5: Script uses interactive prompts
  apt install asks "Y/N" but no terminal = hangs
  Fix: Use apt-get install -y (force yes)
```

**Step 3 — Test in isolation:**
```bash
# SSH into VM and run the exact script as root to reproduce:
sudo bash -c "$(curl -fsSL 'https://your-storage.blob.core.windows.net/scripts/setup.sh')"
```

---

**Q41. Scenario: Your web application deployed on Azure VMs is experiencing very slow response times during business hours (9 AM–6 PM). How do you diagnose and fix it?**

**A:**

**Step 1 — Diagnose with Azure Monitor:**
```
Portal → VM → Monitoring → Metrics
  CPU Percentage     → Is CPU > 80% during 9-6?
  Available Memory   → Is memory < 10%?
  Disk Read/Write    → High disk queue?
  Network In/Out     → Bandwidth saturation?
```

**Step 2 — Application-level check:**
```bash
# SSH into VM during slow period:
top                    # What's consuming CPU?
vmstat 1 10           # CPU, memory, I/O summary
iotop                  # Which process is using disk?
netstat -ant | wc -l  # Too many open connections?
```

**Step 3 — Identify root cause and fix:**

| Root Cause | Symptom | Fix |
|-----------|---------|-----|
| CPU maxed out | CPU > 95% in Monitor | Scale up VM size (more vCPUs) OR optimize app code |
| Memory pressure | High page faults | Scale up RAM or add Redis cache |
| Disk bottleneck | High disk queue | Upgrade to Premium SSD |
| Too many connections | Database timeouts | Add connection pooling, read replicas |
| Single VM overloaded | All above | Migrate to VM Scale Set with load balancer |

**Step 4 — Long-term fix:**
```
Convert single VM to VMSS:
  → 2-4 VMs during 9 AM–6 PM (scale out)
  → 1-2 VMs during 6 PM–9 AM (scale in, save cost)
  → Auto-scaling based on CPU threshold
  → Application Gateway in front for distribution
```

---

**Q42. Scenario: Security audit finds that several Azure VMs have port 22 open to the entire internet (0.0.0.0/0). How do you remediate this at scale?**

**A:**

**Immediate remediation:**

```bash
# Azure CLI — Fix all NSG rules allowing SSH from 0.0.0.0/0
# Step 1: Find all such NSGs and rules
az network nsg list --query \
  "[].{NSG:name, RG:resourceGroup}" --output table

# Step 2: For each NSG, update SSH rule source
az network nsg rule update \
  --resource-group <RG> \
  --nsg-name <NSG-NAME> \
  --name Allow-SSH \
  --source-address-prefixes <COMPANY-IP-RANGE>
  # Example: 203.0.113.0/24 (your corporate network)
```

**Proper long-term fixes:**

```
Option 1: Restrict SSH to VPN/corporate IP only
  Source: 203.0.113.0/24 (corporate NAT IP)
  → Only accessible from office network

Option 2: Deploy Azure Bastion — eliminate public SSH entirely
  Remove all public IPs from VMs
  Remove all SSH NSG rules
  Access ONLY through Bastion (browser, no port 22 exposed)

Option 3: Azure VPN Gateway + NSG
  Employees connect to VPN first
  NSG allows SSH from VPN subnet CIDR only

Option 4: Just-in-Time VM Access (Microsoft Defender for Cloud)
  Portal → Defender for Cloud → Workload protections → Just-in-time VM access
  SSH port 22 is closed by default
  Request access → port opens for 3 hours → auto-closes
  Every access is logged
```

**Prevention — Azure Policy:**
```
Create Azure Policy: "Deny NSG rules allowing SSH from internet"
Applies to all future resource creations
Non-compliant resources flagged in Defender for Cloud
```

---

**Q43. Scenario: You need to store 10,000 high-resolution images (average 5 MB each) uploaded by users, serve them quickly on a website, and back them up for 10 years. Design the complete storage solution.**

**A:**

```
Complete Storage Design:
════════════════════════════════════════════════════════════════

1. USER UPLOAD:
   User uploads → Azure App Service API → Azure Blob Storage
   
   Container: user-uploads (Hot tier, private access)
   Blob naming: uploads/{userId}/{year}/{month}/{imageId}.jpg
   Enable: Blob versioning + Soft delete (30 days)
   
2. IMAGE PROCESSING:
   Upload triggers Azure Function
   Function resizes: original + thumbnail (200x200) + medium (800x600)
   
   Containers:
   originals/     → Hot tier (recent 30 days) → Cool tier → Cold
   thumbnails/    → Hot tier (always, small size)
   medium/        → Hot tier (active use)

3. SERVING IMAGES FAST:
   Azure CDN in front of Blob Storage
   CDN edge nodes serve cached images globally → fast for users
   Cache-Control: max-age=31536000 (1 year for static images)
   
   URL: https://cdn.myapp.com/medium/{imageId}.jpg
        → CDN → Blob Storage (on cache miss)
   
4. 10-YEAR BACKUP:
   Lifecycle Policy:
   originals/ 0-30d    → Hot   (~50TB × $0.018 = $900/mo)
   originals/ 30-90d   → Cool  (50TB × $0.010 = $500/mo)
   originals/ 90-365d  → Cold  (50TB × $0.005 = $250/mo)
   originals/ 1yr+     → Archive (50TB × $0.001 = $50/mo)
   
   Enable WORM (immutability) on backup container
   
5. ACCESS CONTROL:
   App Service: Managed Identity → Storage Blob Data Contributor
   Users: SAS tokens (read-only, 1-hour expiry) for original downloads
   Admin: RBAC Contributor role
   CDN: Public read on thumbnails/medium containers

Result: Fast serving + automatic cost optimization + 10-year compliance
```

---

**Q44. Scenario: Your team in India is working with a team in the US. They both need to access the same Azure File Share. How do you set this up, and what latency issues should you expect?**

**A:**

**Setup:**

```bash
# Create Storage Account + File Share
az storage account create \
  --name globalfileshare123 \
  --resource-group shared-rg \
  --location eastus \
  --sku Standard_GRS \    # Geo-redundant for US+India access
  --kind StorageV2

az storage share create \
  --name teamshare \
  --account-name globalfileshare123 \
  --quota 100            # 100 GB quota
```

**Mount on Linux VM (India team):**
```bash
sudo mkdir /mnt/teamshare
sudo apt-get install cifs-utils
sudo mount -t cifs //globalfileshare123.file.core.windows.net/teamshare /mnt/teamshare \
  -o vers=3.0,username=globalfileshare123,password=<storage-key>,dir_mode=0777,file_mode=0777
```

**Mount on Windows (US team):**
```
Map Network Drive → \\globalfileshare123.file.core.windows.net\teamshare
Username: AZURE\globalfileshare123
Password: <storage-key>
```

**Latency consideration:**
```
US East team → East US storage = ~5ms (same region, fast)
India team   → East US storage = ~150ms (cross-continent, slow for large files)

Better solution: Azure File Sync
  Deploy sync agent in India
  India VMs → Local cache → Syncs to Azure
  India reads from local cache = fast (<5ms)
  Changes sync to Azure → US team sees updated files
```

---

**Q45. Scenario: A startup with 5 developers wants to use Azure but has a $200/month budget. They need: a staging server, a production server, CI/CD pipeline, container registry, and version control. Design the most cost-effective architecture.**

**A:**

```
$200/Month Budget Architecture:
════════════════════════════════════════════════════════════════

VERSION CONTROL: Azure DevOps Repos
  Cost: FREE (5 users, unlimited private repos)
  
CI/CD PIPELINE: Azure Pipelines
  Cost: FREE (1 Microsoft-hosted parallel job, 1,800 min/month)
  
CONTAINER REGISTRY: Azure Container Registry Basic
  Cost: $5/month (10 GB storage, Standard features)
  
STAGING SERVER: Standard_B2s VM (2 vCPU, 4 GB RAM)
  Cost: ~$35/month
  OS: Ubuntu 22.04
  Disk: Standard SSD 64 GB
  Auto-shutdown at 11 PM (save ~30%)
  
PRODUCTION SERVER: Standard_B2s VM (2 vCPU, 4 GB RAM)
  Cost: ~$35/month (always running)
  Static Public IP: $4/month
  
STORAGE: Azure Blob (Standard LRS)
  Cost: ~$5/month for typical startup usage
  
DATABASE: Azure SQL Database (Free tier — 32 GB)
  Cost: FREE (Always Free tier)
  
NETWORKING:
  1 VNet (free), 1 NSG (free), 1 Public IP Static ($4/month)
  
MONITORING: Azure Monitor Basic
  Cost: FREE (basic metrics + 5 GB logs/month)

TOTAL ESTIMATE: ~$84/month
Remaining budget: $116/month headroom for growth

Savings tips:
  - Stop staging VM at night: az vm auto-shutdown
  - Use B-series (burstable) VMs — cheaper for variable workloads
  - Use Azure Student credits if eligible ($100 free)
  - Use 1-year Reserved Instances once stable: saves ~40%
```

---

## 4. Quick-Fire Q&A

```
Q: What port does SSH use?
A: 22

Q: What port does RDP use?
A: 3389

Q: What port does HTTPS use?
A: 443

Q: What is the default Azure VM username?
A: azureuser (for Azure Marketplace Linux images)

Q: How many Availability Zones does Central India (Pune) have?
A: 3

Q: Does West India (Mumbai) have Availability Zones?
A: No — use Availability Sets instead

Q: What is the free tier VM size in Azure?
A: Standard_B1s (1 vCPU, 1 GB RAM, 750 hours/month for 12 months)

Q: What is the difference between Hot and Archive blob tiers in one sentence?
A: Hot = expensive + instant access; Archive = cheapest + offline (hours to retrieve)

Q: What does "Stopped (Deallocated)" mean vs just "Stopped"?
A: Deallocated = compute billing stopped, IP released; Stopped = compute STILL charged

Q: What is the minimum storage duration for Archive tier?
A: 180 days

Q: What is rehydration?
A: Moving a blob from Archive tier to Hot/Cool before it can be read

Q: Can you shrink an Azure Managed Disk?
A: No — only expansion is supported

Q: What service replaces exposing port 22 to the internet?
A: Azure Bastion

Q: What is the Azure equivalent of AWS S3?
A: Azure Blob Storage

Q: What is the Azure equivalent of AWS EBS?
A: Azure Managed Disk

Q: What is the Azure equivalent of AWS VPC?
A: Azure Virtual Network (VNet)

Q: What is the Azure equivalent of AWS Security Groups?
A: Network Security Groups (NSG)

Q: What is the Azure equivalent of AWS IAM?
A: Azure Active Directory (Azure AD / Entra ID)

Q: What is the Azure equivalent of AWS Auto Scaling Groups?
A: VM Scale Sets (VMSS)

Q: What is a Fault Domain?
A: Group of VMs sharing the same physical rack (power + network switch) in an Availability Set

Q: What is an Update Domain?
A: Group of VMs that reboot together during Azure planned maintenance

Q: What does NSG stand for?
A: Network Security Group

Q: What is a SAS token?
A: Shared Access Signature — a time-limited URL granting access to a specific blob

Q: Where is the initial Jenkins-equivalent admin password for Azure VMs?
A: (Not applicable) — Azure VMs use SSH keys or passwords set during creation

Q: What Azure service lets you manage on-premise servers from Azure Portal?
A: Azure Arc

Q: What does Azure Bastion cost approximately?
A: ~$0.19/hour for the Standard tier

Q: Can Blob Storage be used as an OS boot disk?
A: No — it's non-bootable; use Managed Disk (block storage) instead

Q: What does RBAC stand for?
A: Role-Based Access Control
```

---

## 5. Interview Tips

```
WHAT INTERVIEWERS LOOK FOR:
══════════════════════════════════════════════════════════════════

FUNDAMENTAL CLARITY
  ✓ Can you explain Region vs AZ vs Availability Set clearly?
  ✓ Do you know the difference between object storage and block storage?
  ✓ Can you draw the VM architecture from memory?

COST AWARENESS (critical for cloud roles)
  ✓ Do you know which services are free vs paid?
  ✓ Can you optimize storage using tiers?
  ✓ Do you understand "Stopped" vs "Deallocated"?
  ✓ When would you use Reserved Instances?

SECURITY MINDSET
  ✓ Would you open port 22 to 0.0.0.0/0? (Answer: Never)
  ✓ How do you handle secrets? (Answer: Key Vault, not config files)
  ✓ Do you use Bastion or expose SSH publicly?
  ✓ What is the least-privilege principle in RBAC?

PRACTICAL TROUBLESHOOTING
  ✓ VM not reachable → what do you check first? (Resource Health → NSG → VM state)
  ✓ Unexpected bill → how do you find the cause? (Cost Management)
  ✓ Cannot SSH → what are the 5 common causes?

ARCHITECTURE THINKING
  ✓ Can you design for high availability using AZs?
  ✓ Do you know when to use VMSS vs single VM?
  ✓ Can you design a cost-effective storage solution for different access patterns?

AZURE vs AWS COMPARISONS
  ✓ Interviewers often ask: "What is Azure's equivalent of S3/EBS/VPC/IAM?"
  ✓ Know at least 10 Azure ↔ AWS service mappings

COMMON MISTAKES TO AVOID IN INTERVIEW
  ✗ Saying "just use the free tier" for every scenario
  ✗ Not mentioning security (NSG, RBAC, Key Vault) in architecture questions
  ✗ Forgetting to mention monitoring/alerting in design questions
  ✗ Saying Blob Storage is bootable
  ✗ Confusing "Stopped" with "Stopped (Deallocated)"
```

---

