# Microsoft Azure — Troubleshooting Guide

> **Purpose:** Diagnose and fix real-world Azure issues fast  
> **Structure:** Problem → Symptom → Root Cause → Fix → Prevention  
> **Covers:** Account · VMs · Storage · Networking · Security · Billing

---

## Table of Contents

1. [Quick Diagnostic Checklist](#1-quick-diagnostic-checklist)
2. [Account & Signup Issues](#2-account--signup-issues)
3. [Azure Portal Issues](#3-azure-portal-issues)
4. [Virtual Machine Issues](#4-virtual-machine-issues)
5. [VM Connectivity Issues](#5-vm-connectivity-issues)
6. [VM Performance Issues](#6-vm-performance-issues)
7. [Storage Account Issues](#7-storage-account-issues)
8. [Blob Storage Issues](#8-blob-storage-issues)
9. [Networking Issues](#9-networking-issues)
10. [Security & Access Issues](#10-security--access-issues)
11. [Billing & Cost Issues](#11-billing--cost-issues)
12. [Azure CLI & API Issues](#12-azure-cli--api-issues)
13. [VM Extension Issues](#13-vm-extension-issues)
14. [Availability & HA Issues](#14-availability--ha-issues)
15. [Quick Error Reference Table](#15-quick-error-reference-table)
16. [Useful Commands Cheat Sheet](#16-useful-commands-cheat-sheet)

---

## 1. Quick Diagnostic Checklist

Run this checklist first — it resolves the majority of common issues.

```
FIRST-RESPONSE CHECKLIST
══════════════════════════════════════════════════════════════════

[ ] Is the Azure platform itself healthy?
      Check: status.azure.com
      Look for incidents in your region

[ ] Is your subscription active?
      Portal → Subscriptions → Check status (not Disabled)

[ ] Is the resource in the right region?
      Some services have region-specific limitations

[ ] Is there a quota limit hit?
      Portal → Subscriptions → Usage + quotas

[ ] Does the NSG allow the required traffic?
      VM → Networking → Check Inbound / Outbound rules

[ ] Is the VM actually running?
      VM → Overview → Status: Running (not Stopped/Deallocated)

[ ] Do you have the correct RBAC permissions?
      Portal → Resource → Access control (IAM) → Check your role

[ ] Is there sufficient disk space on the VM?
      SSH → df -h

[ ] Are credentials valid and not expired?
      Check passwords, SSH keys, SAS tokens, API keys

[ ] Check Azure Activity Log for recent changes:
      Resource → Activity Log → Last 24 hours
```

---

## 2. Account & Signup Issues

---

### 2.1 Credit Card Declined During Signup

**Symptoms:**
- "Your card was declined" during free account registration
- Payment verification fails

**Causes & Fixes:**

| Cause | Fix |
|-------|-----|
| Virtual / prepaid card | Use a standard Visa/Mastercard debit card |
| International transaction blocked | Enable international transactions with your bank |
| Bank SMS OTP required | Approve the ₹2/$1 hold via bank OTP/2FA |
| Card not enabled for online use | Enable online/international use in bank app |
| Card linked to another Azure account | Use a different card |

**Fix — Contact bank first:**
```
Tell your bank:
  "Please allow a temporary ₹2 hold from Microsoft Azure
   for identity verification."
Then retry the Azure signup.
```

---

### 2.2 Phone Number Already Used

**Symptoms:**
- "This phone number has already been used to verify an Azure account"

**Cause:** Each phone number can only verify one Azure free account.

**Fixes:**
```
Option 1: Use a different phone number (family member, second number)
Option 2: Use your existing Azure account (check if you already have one)
Option 3: Azure Student plan doesn't require phone — check eligibility
```

---

### 2.3 OTP Not Received

**Symptoms:**
- "Text me" clicked but no SMS received
- Verification code never arrives

**Fixes:**
```
1. Wait 60 seconds — SMS can be delayed
2. Click "Resend OTP" button
3. Try "Call me" option instead of SMS
4. Check if DND (Do Not Disturb) is enabled on your phone
5. Check if your carrier blocks international SMS (Microsoft sends from US numbers)
6. Try a different phone number
```

---

### 2.4 Email Already Has Azure Account

**Symptoms:**
- "An account with this email already exists"
- Cannot create a new free account

**Fixes:**
```
Option 1: Log into existing account at portal.azure.com
          (You may already have a valid account)
Option 2: Use a different email address for the new free account
Option 3: Check if you used a Microsoft work/school account before
          (These are different from personal Microsoft accounts)
```

---

### 2.5 Free Trial Expired / Subscription Disabled

**Symptoms:**
- "Your subscription has been disabled"
- Resources show as stopped
- Portal shows "Free Trial has ended"

**Causes & Fixes:**

```
Cause 1: 30 days elapsed
  Fix: Upgrade to Pay-As-You-Go in:
       Subscriptions → Free Trial → Upgrade

Cause 2: $200 credit exhausted before 30 days
  Fix: Same — upgrade to PAYG or create a new account with new email + card

Cause 3: Policy violation
  Fix: Review Microsoft Azure Terms of Service, contact Azure Support

After upgrade:
  Always-Free services continue working
  12-month free services continue for remainder of 12 months
  Pay-As-You-Go charges start for anything beyond free limits
```

---

## 3. Azure Portal Issues

---

### 3.1 Portal Not Loading / Blank Screen

**Symptoms:**
- portal.azure.com loads blank
- Dashboard shows but resources don't load
- Endless spinner

**Fixes:**

```bash
# Fix 1: Clear browser cache and cookies
# Chrome: Ctrl+Shift+Delete → Clear All → Reload

# Fix 2: Try Incognito/Private window
# Chrome: Ctrl+Shift+N

# Fix 3: Disable browser extensions (especially ad blockers)
# Go to: chrome://extensions → Disable all → Retry

# Fix 4: Try a different browser
# Edge is most compatible with Azure Portal

# Fix 5: Check Azure portal status
# Visit: status.azure.com
```

---

### 3.2 Cannot Find a Service in Portal

**Symptoms:**
- Looking for Azure Bastion, but can't find it
- Service not in navigation

**Fix — Use the search bar:**

```
Top search bar (Ctrl+/) → Type service name
Examples:
  "Bastion"        → Azure Bastion
  "Cost"           → Cost Management + Billing
  "Key vault"      → Key Vault
  "Log"            → Log Analytics
  "Blob"           → Storage Accounts → Blob

Also: All Services (left panel) → Browse by category
```

---

### 3.3 Azure Cloud Shell Not Starting

**Symptoms:**
- Cloud Shell hangs on "Requesting a Cloud Shell..." for minutes
- "Storage not configured" error

**Fix — Configure Cloud Shell storage:**

```
1. Click Cloud Shell icon (>_) in portal top bar
2. When prompted "You have no storage mounted":
   Click "Show advanced settings"
   Select your Subscription
   Select a Region close to you
   Create new Resource Group: cloudshell-rg
   Create new Storage Account: cloudshell<random>
   Create new File Share: cloudshell
   Click "Create storage"
3. Cloud Shell opens with Bash or PowerShell
```

**Fix — Cloud Shell timeout:**

```bash
# Cloud Shell times out after 20 minutes of inactivity
# Run this to keep it alive during long tasks:
while true; do echo "keepalive $(date)"; sleep 600; done &
```

---

## 4. Virtual Machine Issues

---

### 4.1 VM Fails to Start

**Symptoms:**
- VM stuck in "Starting" state for 10+ minutes
- VM status shows "Failed" after starting
- Error: "Allocation failed"

**Diagnosis:**

```
Azure Portal → VM → Activity Log → Check recent errors
Azure Portal → VM → Boot Diagnostics → Screenshot (see what OS shows)
```

**Common Causes & Fixes:**

| Cause | Error Message | Fix |
|-------|--------------|-----|
| No capacity in region/size | "Allocation failed" | Try different VM size, different region, or wait |
| OS disk corrupted | "OS not found" | Restore from backup or repair disk |
| Quota exceeded | "QuotaExceeded" | Request quota increase in Subscriptions → Usage + quotas |
| Azure platform issue | "InternalServerError" | Check status.azure.com; retry after 15 min |
| Subscription disabled | "SubscriptionNotFound" | Reactivate/upgrade subscription |

**Fix — Allocation failed:**

```
Steps:
1. Try same VM size in a different Availability Zone
   (VM → Redeploy → pick Zone 2 if you were in Zone 1)

2. Try a different but similar VM size:
   Standard_D2s_v3  →  Standard_D2as_v4  (AMD alternative)

3. Try a different region (East US has most capacity)

4. Contact Azure Support for capacity reservation
```

---

### 4.2 VM Shows "Stopped (Deallocated)" Unexpectedly

**Symptoms:**
- VM was running, now shows Stopped (Deallocated)
- App is down unexpectedly

**Common Causes:**

```
Cause 1: Auto-shutdown schedule configured
  Fix: VM → Operations → Auto-shutdown → Disable or adjust time

Cause 2: Azure Spot VM evicted
  Fix: Spot VMs are by design evicted — use Regular VMs for critical workloads

Cause 3: Subscription credit ran out (Free Trial)
  Fix: Upgrade subscription or add payment method

Cause 4: Quota/policy enforcement
  Fix: Check Activity Log → Filter "Deallocate"

Cause 5: Someone in team deallocated it
  Fix: Check Activity Log → See WHO performed the operation
```

---

### 4.3 VM Deployment Takes Too Long / Fails

**Symptoms:**
- VM creation stuck at "Creating" for 20+ minutes
- Deployment fails with ARM template error

**Diagnosis:**

```
Portal → Resource Group → Deployments → Click the failed deployment
→ See detailed error message

OR:
Portal → VM → Activity Log → Look for "Failed" operations
```

**Common Fixes:**

```bash
# Fix 1: VM size not available in selected region/zone
# Try different size or zone

# Fix 2: Name already taken
# VM names must be unique within resource group

# Fix 3: Invalid image/OS combination
# Some VM sizes don't support certain OS images

# Fix 4: NSG rules too restrictive during deployment
# Temporarily allow all inbound, deploy, then restrict

# Fix 5: Retry via Azure CLI
az vm create \
  --resource-group MyRG \
  --name MyVM \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-sku Standard
```

---

### 4.4 VM Disk Full

**Symptoms:**
- Application crashes on VM
- SSH/RDP connects but nothing works
- Error: "No space left on device" in logs

**Fix — On Linux VM:**

```bash
# Check disk usage
df -h
du -sh /* 2>/dev/null | sort -rh | head -20

# Find large files
find / -type f -size +100M 2>/dev/null | head -20

# Quick cleanup
sudo apt-get autoremove -y          # Remove unused packages
sudo apt-get autoclean              # Clean package cache
sudo journalctl --vacuum-size=100M  # Truncate system logs
sudo rm -rf /tmp/*                  # Clear temp files
docker system prune -af 2>/dev/null # Clear Docker (if installed)

# Check and clear large log files
ls -lh /var/log/
sudo truncate -s 0 /var/log/syslog
```

**Fix — Expand the OS disk (Azure Portal):**

```
VM → Disks → OS Disk → Disk size → Change size → Save
→ Restart VM
→ SSH into VM:
     sudo growpart /dev/sda 1
     sudo resize2fs /dev/sda1       # ext4
     # OR for xfs:
     sudo xfs_growfs /              # xfs
```

> ⚠️ Disk expansion is one-way: you CAN increase size, you CANNOT decrease it.

---

### 4.5 VM High CPU / Memory Usage

**Symptoms:**
- VM very slow or unresponsive
- Applications timing out
- Azure Monitor shows 95%+ CPU for extended period

**Diagnosis:**

```bash
# SSH into VM and check:
top                          # Real-time CPU/memory usage
htop                         # Better visual (install: apt install htop)
ps aux --sort=-%cpu | head   # Top CPU processes
ps aux --sort=-%mem | head   # Top memory processes
free -h                      # Memory usage summary
iostat -x 1 5                # Disk I/O stats (install: apt install sysstat)
```

**Fixes:**

```bash
# Fix 1: Kill runaway process
kill -9 <PID>

# Fix 2: Resize VM to larger size (via Portal)
# VM → Size → Select larger size → Resize
# ⚠️ VM restarts during resize

# Fix 3: Enable autoscaling (use VMSS instead of single VM)

# Fix 4: Set up Azure Monitor alert
# VM → Monitoring → Alerts → + New alert rule
# Metric: CPU Percentage > 80% for 5 minutes → Email me
```

---

## 5. VM Connectivity Issues

---

### 5.1 Cannot SSH into Linux VM

**Symptoms:**
- `ssh azureuser@<ip>` hangs or "Connection refused"
- "Permission denied (publickey)"
- "Connection timed out"

**Step-by-step diagnosis:**

```bash
# Step 1: Can you reach the VM at all?
ping <vm-public-ip>        # If no response → NSG blocking ICMP (normal)
nc -zv <vm-public-ip> 22   # If refused → NSG blocking port 22

# Step 2: Check NSG rules
# Portal → VM → Networking → Inbound port rules
# Must have: Port 22 TCP Allow from your IP (or Any)

# Step 3: Try with verbose mode to see where it fails
ssh -vvv azureuser@<vm-public-ip>
```

**Common Causes & Fixes:**

| Cause | Error | Fix |
|-------|-------|-----|
| NSG blocking port 22 | Connection timed out | Add NSG rule: Allow TCP 22 Inbound |
| Wrong username | Permission denied | Azure Linux VMs default: `azureuser` |
| Wrong key file | Permission denied (publickey) | Use `-i /path/to/key.pem` |
| Key permissions too open | "Unprotected private key" | `chmod 400 mykey.pem` |
| VM not running | Connection refused | Start the VM first |
| Wrong IP | Various | Check Public IP in VM Overview page |
| SSH service crashed | Connection refused | Use Serial Console to restart SSH |

**Fix — Most common SSH fix:**

```bash
# Wrong permissions on key file
chmod 400 ~/.ssh/mykey.pem

# Correct SSH command
ssh -i ~/.ssh/mykey.pem azureuser@<public-ip>

# If still failing, try Azure Bastion (browser-based):
# Portal → VM → Connect → Bastion
```

**Fix — NSG not allowing SSH:**

```
Portal → VM → Networking → Add inbound port rule:
  Source:            My IP address (or IP ranges)
  Source port:       *
  Destination:       Any
  Destination port:  22
  Protocol:          TCP
  Action:            Allow
  Priority:          100
  Name:              Allow-SSH
```

---

### 5.2 Cannot RDP into Windows VM

**Symptoms:**
- Remote Desktop connection fails
- "Remote Desktop can't connect to the remote computer"
- "An authentication error has occurred"

**Diagnosis & Fixes:**

```
Checklist:
[ ] VM is Running (not Stopped/Deallocated)
[ ] NSG has Allow TCP 3389 Inbound rule
[ ] You're using the correct Public IP (check VM Overview)
[ ] RDP credentials: username and password set during VM creation
[ ] Port 3389 not blocked by your local firewall/ISP

Fix — NSG rule for RDP:
  Source: My IP address
  Port:   3389
  Protocol: TCP
  Action: Allow
  Priority: 110

Fix — Reset password if forgotten:
  Portal → VM → Help → Reset password
  Enter new username + password → Update
  Then retry RDP
```

---

### 5.3 Azure Bastion Not Connecting

**Symptoms:**
- Bastion connects but terminal is blank
- "Bastion is not deployed" error

**Fixes:**

```
Fix 1: Bastion must be in the SAME VNet as the VM
  Check: Bastion → Configuration → Virtual Network

Fix 2: Bastion subnet must be named exactly "AzureBastionSubnet"
  Cannot be renamed or reused for other resources
  Minimum size: /26 (64 addresses)

Fix 3: Bastion requires Standard tier for some features
  Basic tier doesn't support file copy or shareable links

Fix 4: VM NSG must allow:
  Inbound from AzureBastionSubnet (or tag: GatewayManager) on port 22/3389
  Outbound 443 from VM to Internet (for Bastion agent)

Fix 5: Browser issue — try Edge for best Bastion compatibility
```

---

### 5.4 VM Unreachable — Application Not Responding on Port 80/443

**Symptoms:**
- VM is running, SSH works, but website not accessible
- `curl http://<vm-ip>` gives connection refused or timeout

**Diagnosis checklist:**

```bash
# On the VM:
# Is the web server running?
sudo systemctl status nginx
sudo systemctl status apache2

# Is the port actually listening?
sudo ss -tlnp | grep 80
sudo ss -tlnp | grep 443

# Is the firewall inside VM blocking it?
sudo ufw status        # Ubuntu firewall
sudo iptables -L -n    # Raw iptables rules

# From outside:
# Check NSG (most common cause)
# Portal → VM → Networking → Inbound port rules
# Must have: Allow TCP 80 from Any
```

**Fix — Add NSG rule for HTTP/HTTPS:**

```
Portal → VM → Networking → Add inbound port rule:
  Port 80  → Allow HTTP traffic
  Port 443 → Allow HTTPS traffic
  Source: Any (or specific IP range)
```

**Fix — Start web server:**

```bash
sudo systemctl start nginx
sudo systemctl enable nginx   # Start automatically on reboot
```

---

## 6. VM Performance Issues

---

### 6.1 VM Disk I/O Very Slow

**Symptoms:**
- File operations take much longer than expected
- Database queries slow (if DB is on VM)
- `iostat` shows very high `%iops` utilization

**Diagnosis:**

```bash
# Check disk I/O stats
iostat -x 1 5

# Check disk type (Standard HDD is slowest)
# Portal → VM → Disks → Check "Storage type"
```

**Fixes:**

```
Fix 1: Upgrade disk type
  Portal → VM → Disks → OS Disk → Change disk type
  Standard HDD  → Standard SSD  (better, free tier eligible)
  Standard SSD  → Premium SSD   (much faster, costs more)
  Premium SSD   → Ultra Disk    (highest IOPS, costly)

Fix 2: Separate OS disk from data disk
  Don't store application data on OS disk
  Add a Premium SSD data disk for databases

Fix 3: Enable disk caching
  Portal → VM → Disks → Host caching: ReadWrite (for data disks)

Fix 4: Use VM sizes with local NVMe SSD (L-series for storage-heavy workloads)
```

---

### 6.2 Slow Network Between VMs

**Symptoms:**
- Transfers between two Azure VMs are slow
- High latency (>10ms) between VMs in same region

**Diagnosis:**

```bash
# Check network throughput between VMs (install iperf3 on both VMs)
# On VM-1 (server):
iperf3 -s

# On VM-2 (client):
iperf3 -c <VM-1-private-ip>

# Check latency
ping <VM-2-private-ip>
```

**Fixes:**

```
Fix 1: VMs should be in the SAME VNet for fastest communication
  Peered VNets add latency

Fix 2: Enable Accelerated Networking
  Portal → VM → Networking → Enable Accelerated Networking
  Supported on: D2s_v3 and larger (not on B1s)
  Reduces latency by bypassing virtual switch

Fix 3: Use VMs in same Availability Zone
  Intra-zone traffic has lower latency than inter-zone

Fix 4: Use Premium SSD if disk I/O is the bottleneck
```

---

## 7. Storage Account Issues

---

### 7.1 Storage Account Name Unavailable

**Symptoms:**
- "The storage account name is already taken"
- Every name you try is rejected

**Cause:** Storage account names must be **globally unique across ALL of Azure** (3–24 chars, lowercase alphanumeric only).

**Fix:**

```
Naming strategy:
  <company><project><environment><random4>
  Example: acmewebprod3847

Add random numbers to common names:
  mystorage        → Too generic, taken
  mystorage2026    → Try adding year
  mystorage20260531 → Add full date
  projectname + uuid[:8]
```

---

### 7.2 Blob Upload Fails

**Symptoms:**
- File upload to Blob Storage fails
- "AuthorizationFailure" error
- "BlobAccessTierNotSupported" error

**Fixes:**

```
Fix 1: AuthorizationFailure
  → You don't have permissions on this storage account
  → Check: Storage Account → Access control (IAM)
  → Your role must be: Storage Blob Data Contributor (or Owner)
  → Note: Subscription "Owner" doesn't automatically give blob access
    You need BOTH subscription owner AND storage data role

Fix 2: BlobAccessTierNotSupported
  → Archive tier blobs are offline
  → Rehydrate first: Blob → Change tier → Cool or Hot → Wait hours

Fix 3: File too large (single upload limit = 5 GB)
  → Use AzCopy or Azure Storage SDK for large files (they use multi-part upload)

Fix 4: Container doesn't exist
  → Create container first: Storage Account → Containers → + Container
  → Set access: Private / Blob / Container
```

---

### 7.3 SAS Token Not Working

**Symptoms:**
- Shared Access Signature URL returns 403 Forbidden
- "Signature did not match" error

**Common Causes & Fixes:**

```
Cause 1: SAS token expired
  Fix: Generate a new SAS token with longer expiry
  Path: Storage Account → Shared access signature → Set dates → Generate

Cause 2: Wrong permissions in SAS
  Fix: Ensure the SAS includes Read permission (and Write if uploading)
  Check: r=read, w=write, d=delete, l=list in the SAS URI

Cause 3: Clock skew (your machine clock is wrong)
  Fix: Sync your system clock
  Windows: Settings → Time → Sync now
  Linux: sudo ntpdate pool.ntp.org

Cause 4: IP restriction on SAS
  Fix: Check "Allowed IP addresses" in SAS config — ensure your IP is included

Cause 5: HTTPS only but you used HTTP
  Fix: Use https:// in the URL
```

---

### 7.4 Archived Blob Cannot Be Accessed

**Symptoms:**
- 409 BlobArchived error when reading a blob
- "This operation is not permitted as the blob is archived"

**Fix — Rehydrate the blob:**

```
Option 1: Change tier via Portal
  Storage Account → Container → Select blob
  → Change tier → Hot or Cool → OK
  Wait: Standard = up to 15 hours | High priority = under 1 hour

Option 2: Via Azure CLI
  az storage blob set-tier \
    --account-name mystorageacct \
    --container-name mycontainer \
    --name myblob.zip \
    --tier Hot \
    --rehydrate-priority High

Option 3: Copy to new blob (rehydrate via copy)
  az storage blob copy start \
    --destination-container hotcontainer \
    --destination-blob myblob.zip \
    --source-container archivecontainer \
    --source-blob myblob.zip \
    --tier Hot
```

---

### 7.5 Storage Costs Unexpectedly High

**Symptoms:**
- Monthly storage bill higher than expected
- Archive tier costing more than anticipated

**Diagnosis:**

```
Portal → Storage Account → Monitoring → Metrics
→ View: Transactions, Egress (data transfer), Storage capacity

Portal → Cost Management + Billing → Cost analysis
→ Filter by Resource → Storage Account
→ See breakdown by service (storage, operations, bandwidth)
```

**Fixes:**

```
Fix 1: Enable Lifecycle Management policies
  Storage Account → Data management → Lifecycle Management → + Add rule
  Example: Move to Cool after 30 days, Archive after 90, Delete after 365

Fix 2: Delete orphaned snapshots
  Storage Account → Blob containers → Manage snapshots

Fix 3: Early deletion fees on Archive
  Archive minimum = 180 days
  If you delete at 90 days → charged for remaining 90 days
  Solution: Don't archive data you might need within 180 days

Fix 4: Data egress charges
  Inbound to Azure: FREE
  Outbound from Azure: Charged (first 100 GB/month free)
  Between regions: Charged
  Fix: Keep frequently accessed data in same region as compute
```

---

## 8. Blob Storage Issues

---

### 8.1 Public Access Denied / Blob Not Accessible via URL

**Symptoms:**
- Direct blob URL returns 403 or 404
- `https://account.blob.core.windows.net/container/file.jpg` not loading

**Diagnosis:**

```
Check 1: Container access level
  Storage Account → Containers → Select container → Change access level
  
  Private        → No public access (need auth or SAS)
  Blob           → Public access to blobs, not container list
  Container      → Full public access (blobs + listing)

Check 2: Storage Account public access setting
  Storage Account → Configuration → Allow Blob public access: Enabled
  If disabled → No public access even if container is set to public
```

**Fix:**

```
For public file hosting (website, images):
  1. Storage Account → Configuration → Allow Blob public access: Enabled
  2. Container → Change access level: Blob (public read for blobs)
  3. URL: https://<account>.blob.core.windows.net/<container>/<blobname>

For private access:
  Use SAS Token or Azure AD authentication
  Never make containers public unless intentional
```

---

### 8.2 Lifecycle Policy Not Running

**Symptoms:**
- Set up lifecycle policy but blobs haven't moved to Cool/Archive
- Policy shows as enabled but no effect

**Causes & Fixes:**

```
Cause 1: Policy takes up to 24-48 hours to run after creation
  Fix: Wait 1-2 days before checking

Cause 2: Policy filter doesn't match any blobs
  Fix: Check prefix filter — if you set prefix "logs/", blob must start with "logs/"

Cause 3: Blob has a legal hold or immutability policy
  Fix: Remove immutability lock before lifecycle policy can move/delete

Cause 4: Blob is in a version or snapshot state
  Fix: Enable "Delete previous versions" or "Delete snapshots" in the policy

Cause 5: Blob was accessed recently (cool tier based on last access tracking)
  Fix: Enable "Last access time tracking" in lifecycle management
```

---

## 9. Networking Issues

---

### 9.1 VM Cannot Access Internet

**Symptoms:**
- SSH works but `apt update` fails (cannot download packages)
- `ping google.com` from VM gives "Network unreachable"
- No outbound internet from VM

**Diagnosis:**

```bash
# From inside VM:
ping 8.8.8.8            # Test IP connectivity (no DNS)
ping google.com         # Test DNS + IP
curl -v https://google.com
nslookup google.com     # Test DNS resolution
route -n                # Check routing table
```

**Fixes:**

```
Fix 1: NSG outbound rules blocking internet
  Portal → VM → Networking → Outbound port rules
  Default NSG has "AllowInternetOutBound" rule (priority 65001)
  If missing or denied → Add: Allow Any outbound to Internet

Fix 2: No public IP and no NAT Gateway
  VMs without public IP need a NAT Gateway or Azure Firewall for outbound
  Fix: Add Public IP to VM OR deploy NAT Gateway on subnet

Fix 3: DNS not configured
  Default Azure DNS: 168.63.129.16 (Azure's internal DNS)
  Check: cat /etc/resolv.conf
  Fix: sudo echo "nameserver 8.8.8.8" >> /etc/resolv.conf

Fix 4: UDR (User Defined Route) forcing traffic through wrong next hop
  Portal → VM → Networking → Effective routes → Check routes
```

---

### 9.2 Two VMs Cannot Communicate

**Symptoms:**
- VM-1 cannot ping or connect to VM-2
- Port timeout between VMs in same VNet

**Diagnosis:**

```bash
# From VM-1:
ping <VM-2-private-IP>       # Test basic connectivity
nc -zv <VM-2-IP> <port>      # Test specific port
traceroute <VM-2-IP>         # Check routing path
```

**Fixes:**

```
Fix 1: VMs in different VNets (not peered)
  Fix: Set up VNet Peering between the two VNets
  Portal → VNet1 → Peerings → + Add → Select VNet2

Fix 2: NSG on VM-2 blocking inbound
  VM-2 → Networking → Inbound port rules
  Add rule: Allow from VM-1's IP (or subnet CIDR)

Fix 3: VMs in different subnets with restrictive NSG
  Check subnet-level NSG (not just VM-level NSG)

Fix 4: Application/OS firewall on VM-2
  sudo ufw status         # Ubuntu firewall
  sudo ufw allow from <VM-1-IP>
  sudo iptables -L -n     # Check iptables
```

---

### 9.3 Public IP Changes After VM Restart

**Symptoms:**
- VM's public IP address is different after restart
- Application/DNS configuration breaks after VM restart

**Cause:** Azure dynamic public IPs change when VM is Stopped (Deallocated).

**Fix — Use Static Public IP:**

```
Option 1: Change existing public IP to Static
  Portal → Public IP resource → Configuration → Assignment: Static → Save
  This IP is now permanently yours until you explicitly release it

Option 2: Assign a new Static IP during VM creation
  Create VM → Networking → Public IP → Create new → Static → OK

Option 3: Use Azure DNS with dynamic TTL
  Point your domain to the IP with low TTL
  Update DNS automatically with Azure Automation when IP changes

Note: Static Public IPs have a small charge (~$0.005/hour when not attached to running VM)
```

---

## 10. Security & Access Issues

---

### 10.1 "You don't have permission" / Authorization Failed

**Symptoms:**
- "AuthorizationFailed" in Azure Portal
- "The client does not have authorization to perform action"
- Cannot create/delete/modify a resource

**Diagnosis:**

```
Portal → Resource → Access control (IAM) → Check your access
→ "View my access" tab → See your assigned roles

Portal → Subscriptions → Your subscription → My permissions
→ Shows your effective permissions
```

**Fixes:**

```
Fix 1: Assign correct RBAC role
  Resource Group / Resource → Access control (IAM) → + Add → Add role assignment
  
  For full access:      Owner or Contributor
  For read-only:        Reader
  For blob access:      Storage Blob Data Contributor
  For VM management:    Virtual Machine Contributor
  
  ⚠️ Owner at subscription level ≠ automatic blob data access
     Storage blob operations need explicit Storage Blob Data role

Fix 2: Wait for role propagation
  RBAC role changes take 1-5 minutes to propagate globally
  Log out and log back in if it still doesn't work

Fix 3: Check if subscription is read-only (spending limit hit)
  Subscriptions → Check for "Spending limit reached" notice
```

---

### 10.2 SSH Key Lost / Cannot Log Into VM

**Symptoms:**
- Original .pem file deleted or lost
- VM locked out

**Fix — Reset SSH public key via Portal:**

```
Portal → VM → Help → Reset password

Mode: Reset SSH public key
  Username: azureuser
  SSH public key: Paste your new public key

OR

Mode: Reset password
  Username: azureuser
  Password: <new-strong-password>

Click Update → Wait 2 minutes → Try connecting
```

**Fix — Generate new SSH key:**

```bash
# Generate new key pair
ssh-keygen -t rsa -b 4096 -f ~/.ssh/azure_new_key

# Get the public key to paste in Azure Portal
cat ~/.ssh/azure_new_key.pub

# Connect with new key
ssh -i ~/.ssh/azure_new_key azureuser@<vm-ip>
```

---

### 10.3 Azure AD Authentication Failing

**Symptoms:**
- Cannot log in to Azure Portal
- Multi-Factor Authentication (MFA) not working

**Fixes:**

```
Fix 1: MFA app not working
  Use backup code (generated during MFA setup)
  Go to: aka.ms/mfasetup to re-register MFA device

Fix 2: Account locked out
  Contact your Azure AD admin to unlock the account
  Admin: Azure AD → Users → Select user → Account status → Unblock

Fix 3: Conditional Access blocking login
  Common with: Login from new country, new device, non-compliant device
  Contact IT admin to allow your login

Fix 4: Forgot Azure AD password
  Go to: aka.ms/sspr (Self-Service Password Reset)
  Requires: alternate email or phone registered for recovery
```

---

### 10.4 Key Vault Secret Access Denied

**Symptoms:**
- "Access denied" when trying to get a secret from Key Vault
- Application cannot retrieve Key Vault secrets

**Fixes:**

```
Fix 1: Managed Identity not assigned to VM/App
  VM → Identity → System assigned → Status: On → Save
  Then: Key Vault → Access control (IAM) → Add role assignment
  Role: Key Vault Secrets User
  Assign to: Your VM's managed identity

Fix 2: Access policy not configured (old model)
  Key Vault → Access policies → + Add access policy
  Secret permissions: Get, List
  Select principal: Your app/user/managed identity

Fix 3: Network rules blocking access
  Key Vault → Networking → Firewalls and virtual networks
  Allow your VM's VNet subnet or IP

Fix 4: Key Vault in different subscription
  Managed identity in one subscription cannot directly access Key Vault in another
  Solution: Use service principal instead
```

---

## 11. Billing & Cost Issues

---

### 11.1 Unexpected Charges on Free Account

**Symptoms:**
- Email about charges despite being on free tier
- Cost Management shows unexpected billing

**Most Common Causes:**

```
Cause 1: VM NOT properly stopped
  "Stopped" state ≠ "Stopped (Deallocated)"
  Stopped but not deallocated: STILL CHARGES COMPUTE
  
  Fix: Portal → VM → Stop → Confirm "Do you want to stop?"
       Wait until status shows: "Stopped (deallocated)"
       OR: az vm deallocate --resource-group RG --name VMName

Cause 2: Using wrong VM size (not B1s)
  B1s = free; any larger size = charges immediately
  
  Fix: Resize to B1s or delete and recreate as B1s

Cause 3: Public IP address (Static IP)
  Static IPs charge ~$0.005/hour even when not attached
  
  Fix: Delete static IP when not needed

Cause 4: Premium SSD disk (not covered by free tier)
  Fix: Change to Standard SSD or Standard HDD
  VM → Disks → Change disk type

Cause 5: Data egress (outbound bandwidth)
  Free: 100 GB/month outbound
  Beyond that: charged per GB
  Fix: Minimize data transfers, use CDN for web assets

Cause 6: 12-month free services expired
  After 12 months from signup, those services bill normally
```

---

### 11.2 Unable to Set Budget Alert

**Symptoms:**
- Budget alert created but no email received
- Spending exceeded budget but no notification

**Fixes:**

```
Fix 1: Check email in notification settings
  Cost Management → Budgets → Select budget → Edit
  Verify email address is correct and accessible

Fix 2: Alerts go to spam folder
  Check spam/junk folder
  Whitelist: azure-noreply@microsoft.com

Fix 3: Alert threshold not met
  Actual spend must exceed X% of budget to trigger
  Set multiple thresholds: 25%, 50%, 80%, 100%

Fix 4: Check action group
  Cost Management → Budgets → Action Group → Verify email action configured

Fix 5: Budget scope wrong
  Ensure budget is scoped to correct subscription/resource group
```

---

## 12. Azure CLI & API Issues

---

### 12.1 Azure CLI Login Fails

**Symptoms:**
- `az login` opens browser but auth doesn't complete
- "AADSTS" error codes during login

**Fixes:**

```bash
# Fix 1: Clear existing credentials and retry
az logout
az account clear
az login

# Fix 2: Use device code login (better for headless/remote environments)
az login --use-device-code
# Opens: https://microsoft.com/devicelogin → Enter code shown in terminal

# Fix 3: Login with specific tenant
az login --tenant <your-tenant-id>

# Fix 4: Service Principal login (for automation)
az login --service-principal \
  --username <app-id> \
  --password <client-secret> \
  --tenant <tenant-id>

# Fix 5: Update Azure CLI to latest version
# Ubuntu/Debian:
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
# Windows: Download from https://aka.ms/installazurecliwindows
```

---

### 12.2 Wrong Subscription Context

**Symptoms:**
- CLI commands working on wrong subscription
- Resources not visible after `az vm list`

**Fixes:**

```bash
# List all subscriptions
az account list --output table

# Set the correct subscription
az account set --subscription "Free Trial"
# OR by subscription ID:
az account set --subscription "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

# Verify current context
az account show
```

---

### 12.3 ARM Template / Bicep Deployment Fails

**Symptoms:**
- `az deployment group create` fails
- Template deployment error in Portal

**Diagnosis:**

```bash
# Get detailed error from failed deployment
az deployment group show \
  --resource-group MyRG \
  --name MyDeployment \
  --query properties.error

# View all deployment operations
az deployment operation group list \
  --resource-group MyRG \
  --name MyDeployment \
  --query "[?properties.provisioningState=='Failed']"
```

**Common Fixes:**

```
Error: "Resource already exists"
  Fix: Change resource name or add to existing resource

Error: "Location not available for resource type"
  Fix: Some resource types have limited region availability
  Try: East US or West Europe for broadest coverage

Error: "QuotaExceeded"
  Fix: Subscriptions → Usage + quotas → Request increase

Error: "InvalidTemplate"
  Fix: Validate template before deploying:
       az bicep build --file main.bicep
       az deployment group validate --resource-group RG --template-file main.bicep
```

---

## 13. VM Extension Issues

---

### 13.1 Custom Script Extension Fails

**Symptoms:**
- Extension shows "Failed" in Portal
- Script ran but application not installed correctly

**Diagnosis:**

```bash
# View extension logs on the VM (after SSH)
# Linux:
cat /var/log/azure/custom-script/handler.log
cat /var/lib/waagent/custom-script/download/0/stdout
cat /var/lib/waagent/custom-script/download/0/stderr

# Windows (PowerShell):
Get-Content "C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.CustomScriptExtension\*\CustomScriptHandler.log"
```

**Common Fixes:**

```
Fix 1: Script download failed (storage URL wrong/expired)
  Verify the blob URL is accessible
  Use SAS token if storage is private
  
Fix 2: Script permission denied
  chmod +x yourscript.sh
  Or add to script: #!/bin/bash at top

Fix 3: Script syntax error
  Test locally before using as extension

Fix 4: Extension already installed, needs update
  Portal → VM → Extensions → Delete the old extension
  Re-add with new script

Fix 5: Internet access needed but blocked
  Your script downloads packages → VM needs internet access
  Check NSG outbound rules allow HTTPS outbound
```

---

## 14. Availability & HA Issues

---

### 14.1 Availability Zone Not Available for Selected Region

**Symptoms:**
- Zone option not appearing during VM creation
- "Availability Zones are not available in this region"

**Cause:** Not all Azure regions have Availability Zones.

```
India Regions:
  Central India (Pune) → ✅ 3 AZs available  ← USE THIS for HA
  West India (Mumbai)  → ❌ No AZs
  South India (Chennai)→ ❌ No AZs

Fix for West/South India:
  Use Availability Set instead (protects from rack failure within one DC)
  Portal → VM → Availability options → Availability Set → Create new
```

---

### 14.2 Load Balancer Not Distributing Traffic

**Symptoms:**
- All traffic going to one VM instead of being distributed
- Unhealthy VMs still receiving traffic

**Diagnosis:**

```
Portal → Load Balancer → Backend pools → Check VMs listed
Portal → Load Balancer → Health probes → Check probe settings
Portal → Load Balancer → Load balancing rules → Check configuration
Portal → Load Balancer → Insights → See per-VM traffic distribution
```

**Fixes:**

```
Fix 1: Health probe failing for all VMs
  Check probe endpoint (HTTP GET /health must return 200)
  Ensure web server is running on all VMs
  Ensure NSG allows health probe traffic (source: AzureLoadBalancer)

Fix 2: Session persistence (sticky sessions) enabled
  Load Balancer → Load balancing rules → Session persistence: None
  Sticky sessions = all requests from one client go to same VM

Fix 3: VMs not in backend pool
  Load Balancer → Backend pools → Add your VMs

Fix 4: NSG blocking health probe
  VM NSG must allow: Source = AzureLoadBalancer, Port = probe port
  Add rule: Allow TCP <probe-port> from AzureLoadBalancer (service tag)
```

---

## 15. Quick Error Reference Table

```
ERROR MESSAGE / CODE                    CAUSE                           FIX
───────────────────────────────────     ─────────────────────────────   ──────────────────────────────────────
AuthorizationFailed                     Missing RBAC permission          Add role in Access control (IAM)
SubscriptionNotFound                    Wrong sub or sub disabled        Check subscription status, re-login
QuotaExceeded                           vCPU/resource limit hit          Request quota increase in Portal
AllocationFailed                        No capacity for VM size/region   Try different size, zone, or region
BlobArchived                            Blob in Archive tier             Rehydrate blob to Hot/Cool first
BlobAccessTierNotSupported              Unsupported tier for operation   Check blob type supports tiers
AuthenticationFailed                    Wrong credentials                Reset password/SSH key
ResourceNotFound                        Resource deleted or wrong region Check resource group and region
StorageAccountAlreadyTaken              Name globally taken              Use unique name with numbers
InvalidStorageAccountName               Invalid characters in name       Use only lowercase alphanumeric
NetworkSecurityGroup blocked port       NSG rule missing or denying      Add Allow rule for required port
VNetPeeringNotFound                     VNets not peered                 Set up VNet peering
PublicIPAllocationFailed                No Standard Public IP available  Try different region/zone
DiskShrinkNotSupported                  Tried to reduce disk size        Cannot shrink — create new smaller disk
VMExtensionProvisioningError            Extension script failed          Check extension logs on VM
OperationNotAllowed (Archive)           Cannot operate on Archive blob   Change tier first
ResourceGroupNotFound                   Wrong resource group name        Check spelling and region
Unauthorized (403)                      SAS expired or invalid           Generate new SAS token
RequestTimeout (504)                    Operation took too long          Retry — often transient
InternalServerError (500)               Azure platform error             Check status.azure.com, retry
```

---

## 16. Useful Commands Cheat Sheet

```bash
# ══════════════════════════════════════════════════════════════
# AZURE CLI — AUTH & ACCOUNT
# ══════════════════════════════════════════════════════════════
az login                                    # Interactive login
az login --use-device-code                  # Device code login
az logout                                   # Logout
az account list --output table              # List subscriptions
az account set --subscription "Free Trial"  # Switch subscription
az account show                             # Current subscription

# ══════════════════════════════════════════════════════════════
# RESOURCE GROUPS
# ══════════════════════════════════════════════════════════════
az group list --output table
az group create --name MyRG --location centralindia
az group delete --name MyRG --yes --no-wait  # Delete all resources!
az group show --name MyRG

# ══════════════════════════════════════════════════════════════
# VIRTUAL MACHINES
# ══════════════════════════════════════════════════════════════
az vm list --output table
az vm show --resource-group MyRG --name MyVM

# Create Linux VM (Free Tier)
az vm create \
  --resource-group MyRG \
  --name MyVM \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-sku Standard

# VM power operations
az vm start --resource-group MyRG --name MyVM
az vm stop --resource-group MyRG --name MyVM
az vm deallocate --resource-group MyRG --name MyVM     # Stop + deallocate
az vm restart --resource-group MyRG --name MyVM
az vm delete --resource-group MyRG --name MyVM --yes

# Get VM public IP
az vm show --resource-group MyRG --name MyVM \
  --show-details --query publicIps --output tsv

# Open a port in NSG
az vm open-port --resource-group MyRG --name MyVM --port 80
az vm open-port --resource-group MyRG --name MyVM --port 22 --priority 900

# Reset SSH key
az vm user update \
  --resource-group MyRG \
  --name MyVM \
  --username azureuser \
  --ssh-key-value ~/.ssh/id_rsa.pub

# Resize VM
az vm resize --resource-group MyRG --name MyVM --size Standard_B2s

# ══════════════════════════════════════════════════════════════
# STORAGE
# ══════════════════════════════════════════════════════════════
az storage account list --output table
az storage account create \
  --name mystorageacct123 \
  --resource-group MyRG \
  --location centralindia \
  --sku Standard_LRS

# Container operations
az storage container create \
  --name mycontainer \
  --account-name mystorageacct123 \
  --public-access blob

az storage container list \
  --account-name mystorageacct123 --output table

# Blob operations
az storage blob upload \
  --account-name mystorageacct123 \
  --container-name mycontainer \
  --name photo.jpg \
  --file ~/photo.jpg

az storage blob list \
  --account-name mystorageacct123 \
  --container-name mycontainer --output table

az storage blob download \
  --account-name mystorageacct123 \
  --container-name mycontainer \
  --name photo.jpg \
  --file ~/downloaded.jpg

# Change blob tier
az storage blob set-tier \
  --account-name mystorageacct123 \
  --container-name mycontainer \
  --name oldfile.zip \
  --tier Archive

# ══════════════════════════════════════════════════════════════
# NETWORKING
# ══════════════════════════════════════════════════════════════
# List VNets
az network vnet list --output table

# List NSG rules
az network nsg rule list \
  --resource-group MyRG \
  --nsg-name MyNSG --output table

# Add NSG rule
az network nsg rule create \
  --resource-group MyRG \
  --nsg-name MyNSG \
  --name Allow-HTTP \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-range 80

# ══════════════════════════════════════════════════════════════
# COST & MONITORING
# ══════════════════════════════════════════════════════════════
# Check VM status
az vm get-instance-view \
  --resource-group MyRG \
  --name MyVM \
  --query instanceView.statuses[1].displayStatus

# Activity log
az monitor activity-log list \
  --resource-group MyRG \
  --offset 24h \
  --output table

# ══════════════════════════════════════════════════════════════
# LINUX VM DIAGNOSTIC COMMANDS (run after SSH)
# ══════════════════════════════════════════════════════════════
# System info
uname -a                     # Kernel version
cat /etc/os-release          # OS version
uptime                       # System uptime
hostname -I                  # All IP addresses

# Resource usage
top                          # CPU + memory (interactive)
free -h                      # Memory usage
df -h                        # Disk usage all filesystems
du -sh /var/log/*            # Log sizes
iostat -x 1 3               # Disk I/O stats

# Network
ip addr                      # All network interfaces + IPs
ip route                     # Routing table
ss -tlnp                     # Listening ports
ping 8.8.8.8                # Internet connectivity
curl -v https://google.com  # HTTP test with details
nslookup google.com         # DNS resolution test

# Services
systemctl status nginx      # Check nginx status
systemctl list-units --failed # All failed services
journalctl -xe              # System error logs
journalctl -fu nginx        # Follow nginx logs

# Processes
ps aux --sort=-%cpu | head  # Top CPU consumers
ps aux --sort=-%mem | head  # Top memory consumers
kill -9 <PID>               # Force kill process

# Files
ls -lah /var/log/           # Log files
tail -100 /var/log/syslog   # Recent system log
tail -f /var/log/nginx/error.log  # Follow nginx errors
find / -size +500M 2>/dev/null   # Find large files
```

---

