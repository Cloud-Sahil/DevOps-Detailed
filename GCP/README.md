# GCP Complete Study Guide

> A complete, modular Google Cloud Platform study guide — architecture, IAM, networking (VPC), compute (GCE/GKE/Cloud Run/Cloud Functions), storage, databases, load balancing, security, monitoring, cost optimization, and Infrastructure as Code — with diagrams and `gcloud`/Terraform commands.

---

## Table of Contents

1. [GCP Fundamentals & Global Infrastructure](#module-1-gcp-fundamentals--global-infrastructure)
2. [Resource Hierarchy & IAM](#module-2-resource-hierarchy--iam)
3. [VPC & Networking](#module-3-vpc--networking)
4. [Compute Engine (GCE)](#module-4-compute-engine-gce)
5. [Google Kubernetes Engine (GKE)](#module-5-google-kubernetes-engine-gke)
6. [Serverless: Cloud Run, Cloud Functions, App Engine](#module-6-serverless-cloud-run-cloud-functions-app-engine)
7. [Storage: Cloud Storage, Persistent Disk, Filestore](#module-7-storage-cloud-storage-persistent-disk-filestore)
8. [Databases: Cloud SQL, Spanner, Firestore, BigQuery, Bigtable](#module-8-databases-cloud-sql-spanner-firestore-bigquery-bigtable)
9. [Load Balancing & Auto Scaling](#module-9-load-balancing--auto-scaling)
10. [Security & Identity](#module-10-security--identity)
11. [Monitoring & Operations Suite](#module-11-monitoring--operations-suite)
12. [Cost Optimization](#module-12-cost-optimization)
13. [Infrastructure as Code & CI/CD on GCP](#module-13-infrastructure-as-code--cicd-on-gcp)
14. [High Availability & Disaster Recovery](#module-14-high-availability--disaster-recovery)

---

## MODULE 1: GCP Fundamentals & Global Infrastructure

### 1.1 GCP Global Infrastructure

```
                    Google Global Network
                             │
        ┌────────────────────┼────────────────────┐
     Region                Region                Region
  (asia-south1)          (us-central1)        (europe-west1)
        │
   ┌────┼────┐
  Zone-a  Zone-b  Zone-c   ← Zones (isolated within a region)
        │
  Edge PoPs (Cloud CDN) — closest point to end users globally
```

- **Region** — independent geographic area (e.g., Mumbai = `asia-south1`).
- **Zone** — isolated deployment area within a region (`asia-south1-a`, `-b`, `-c`).
- **Multi-region** — resources (e.g., GCS multi-region buckets) replicated across an entire continent for maximum durability.

**Key differentiator vs AWS:** Google's network uses its own private global fiber backbone between regions (rather than public internet) for a large portion of inter-region traffic, generally giving lower latency for cross-region communication.

---

### 1.2 GCP Service Categories Overview

| Category | Key Services |
|---|---|
| Compute | Compute Engine, GKE, Cloud Run, Cloud Functions, App Engine |
| Storage | Cloud Storage, Persistent Disk, Filestore |
| Database | Cloud SQL, Cloud Spanner, Firestore, Bigtable, BigQuery |
| Networking | VPC, Cloud Load Balancing, Cloud CDN, Cloud DNS, Cloud Interconnect |
| Security | IAM, Cloud KMS, Secret Manager, Security Command Center |
| Operations | Cloud Monitoring, Cloud Logging, Cloud Trace, Cloud Profiler |
| DevOps | Cloud Build, Artifact Registry, Cloud Deploy |
| Data/Analytics | BigQuery, Dataflow, Pub/Sub, Dataproc |

---

### 1.3 GCP Shared Responsibility Model

```
┌──────────────────────────────┐
│   Customer Responsibility     │  ← IAM config, data classification,
│   ("Security IN the cloud")   │     OS patching (GCE), app security
├──────────────────────────────┤
│   Google Responsibility       │  ← Physical security, hardware,
│   ("Security OF the cloud")   │     hypervisor, global network
└──────────────────────────────┘
```

For fully managed services (BigQuery, Cloud Run, Cloud SQL), Google takes on more of the operational burden (patching, scaling) than for unmanaged Compute Engine VMs, shifting the responsibility line further toward Google as you move up the abstraction stack.

---

## MODULE 2: Resource Hierarchy & IAM

### 2.1 GCP Resource Hierarchy

```
Organization
    │
Folders (e.g., "Production", "Development")
    │
Projects (billing + resource container boundary)
    │
Resources (VMs, buckets, GKE clusters, etc.)
```

- **Organization** — root node, tied to your Google Workspace/Cloud Identity domain.
- **Folders** — group projects (e.g., by department or environment).
- **Projects** — the primary billing and resource-isolation boundary — almost everything in GCP lives inside a project.
- IAM policies set at a higher level (Org/Folder) are **inherited** downward; you cannot override an inherited "deny" with an "allow" at a lower level (unless using IAM Deny policies deliberately).

```bash
gcloud projects create my-project-id --organization=123456789012
gcloud resource-manager folders create --display-name="Production" --organization=123456789012
```

---

### 2.2 IAM Roles: Basic vs Predefined vs Custom

| Type | Example | Granularity |
|---|---|---|
| Basic (legacy) | Owner, Editor, Viewer | Very broad — avoid in production |
| Predefined | `roles/compute.instanceAdmin.v1` | Scoped to a specific service/task |
| Custom | `roles/myCustomComputeViewer` | Exact permissions you define |

```bash
gcloud projects add-iam-policy-binding my-project \
  --member="user:john@example.com" \
  --role="roles/compute.instanceAdmin.v1"

gcloud iam roles create customViewer --project=my-project \
  --permissions=compute.instances.get,compute.instances.list
```

**Best practice:** Avoid Basic roles (Owner/Editor/Viewer) in production — they're overly broad. Use Predefined roles, and Custom roles only when predefined roles don't fit precisely.

---

### 2.3 Service Accounts and Workload Identity

A **Service Account** is an identity used by applications/VMs/GKE workloads (not a human) to call GCP APIs.

```bash
gcloud iam service-accounts create my-app-sa --display-name="My App SA"
gcloud projects add-iam-policy-binding my-project \
  --member="serviceAccount:my-app-sa@my-project.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"
```

**Workload Identity (GKE)** — the equivalent of AWS IRSA — lets a Kubernetes Service Account impersonate a GCP IAM Service Account without needing to distribute service account key files:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-ksa
  annotations:
    iam.gke.io/gcp-service-account: my-app-sa@my-project.iam.gserviceaccount.com
```

```bash
gcloud iam service-accounts add-iam-policy-binding my-app-sa@my-project.iam.gserviceaccount.com \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:my-project.svc.id.goog[my-namespace/my-ksa]"
```

**Security best practice:** Avoid downloading and distributing service account JSON key files wherever possible — prefer Workload Identity (GKE), attached service accounts (GCE), or short-lived impersonated credentials.

---

### 2.4 Organization Policies

Organization Policies enforce guardrails across the resource hierarchy (e.g., "no external IPs on VMs," "restrict resource locations to `asia-south1` only") — similar to AWS SCPs.

```bash
gcloud resource-manager org-policies enable-enforce \
  compute.disableSerialPortAccess --project=my-project

gcloud resource-manager org-policies set-policy policy.yaml --project=my-project
```

---

## MODULE 3: VPC & Networking

### 3.1 GCP VPC Architecture — A Key Difference From AWS

```
GCP VPC (GLOBAL resource — spans ALL regions automatically)
        │
   ┌────┼────────────────────┐
Subnet (asia-south1)   Subnet (us-central1)   Subnet (europe-west1)
   REGIONAL resource       REGIONAL resource      REGIONAL resource
```

**Critical distinction from AWS:** In GCP, a VPC is **global** — it's not tied to a single region. Subnets are **regional** (spanning all zones in that region automatically, unlike AWS where subnets are zone-specific). This means a single VPC can have resources across multiple regions communicating over Google's private backbone, without VPC Peering between "regional VPCs" the way AWS requires.

---

### 3.2 Subnet Modes

- **Auto mode** — GCP automatically creates one subnet per region with predefined IP ranges (convenient for quick starts, less control).
- **Custom mode** — you manually define subnets and IP ranges per region (recommended for production).

```bash
gcloud compute networks create my-vpc --subnet-mode=custom
gcloud compute networks subnets create my-subnet \
  --network=my-vpc --region=asia-south1 --range=10.0.0.0/24
```

---

### 3.3 Firewall Rules

Unlike AWS Security Groups (attached to instances) or NACLs (attached to subnets), GCP firewall rules are defined **at the VPC level** and apply based on tags, service accounts, or IP ranges — not instance/subnet attachment.

```bash
gcloud compute firewall-rules create allow-ssh \
  --network=my-vpc --direction=INGRESS --action=ALLOW \
  --rules=tcp:22 --source-ranges=203.0.113.0/24 --target-tags=ssh-allowed
```

**Key point:** GCP firewall rules are **stateful** (like AWS Security Groups) and there is an implied deny-all-ingress / allow-all-egress default, which you then punch holes into.

---

### 3.4 Cloud NAT

Cloud NAT provides outbound-only internet access for instances without external IPs — fully managed, no NAT Gateway/instance to provision or scale manually (unlike AWS NAT Gateway, which you still explicitly provision per AZ).

```bash
gcloud compute routers create my-router --network=my-vpc --region=asia-south1
gcloud compute routers nats create my-nat \
  --router=my-router --region=asia-south1 --auto-allocate-nat-external-ips \
  --nat-all-subnet-ip-ranges
```

---

### 3.5 VPC Peering vs Shared VPC vs Network Connectivity Center

```
VPC Peering:         VPC-A ←──peered──→ VPC-B  (no transitive routing, like AWS)

Shared VPC:          Host Project's VPC
                            │
                    ┌───────┼───────┐
              Service     Service    Service
              Project A   Project B  Project C
              (attach their resources to the host VPC)

Network Connectivity Center: Hub-and-spoke, like AWS Transit Gateway
```

- **VPC Peering** — direct, non-transitive connection between two VPCs.
- **Shared VPC** — a centralized networking model where a "host" project owns the VPC, and multiple "service" projects attach their resources to it — giving centralized network administration while still allowing per-project resource/billing isolation. This is a very GCP-specific, commonly-tested concept with no direct 1:1 AWS equivalent.
- **Network Connectivity Center** — GCP's hub-and-spoke model, equivalent to AWS Transit Gateway, for connecting many VPCs/on-prem networks at scale.

```bash
gcloud compute shared-vpc enable HOST_PROJECT_ID
gcloud compute shared-vpc associated-projects add SERVICE_PROJECT_ID --host-project=HOST_PROJECT_ID
```

---

### 3.6 Hybrid Connectivity: Cloud VPN vs Cloud Interconnect

| Cloud VPN | Dedicated Interconnect | Partner Interconnect |
|---|---|---|
| IPSec over internet | Direct physical connection at a Google colocation facility | Via a supported service provider, no colocation needed |
| Quick setup, lower cost | 10Gbps/100Gbps dedicated circuits | Flexible bandwidth (50Mbps–50Gbps) |
| Higher latency/variable | Lowest latency, highest reliability | Good middle ground |

```bash
gcloud compute vpn-gateways create my-vpn-gw --network=my-vpc --region=asia-south1
```

---

### 3.7 Cloud DNS and Load Balancing Overview

```bash
gcloud dns managed-zones create my-zone --dns-name="example.com." --description="My zone"
gcloud dns record-sets create www.example.com. --zone=my-zone --type=A --ttl=300 --rrdatas=1.2.3.4
```

GCP's load balancing is fundamentally **global by default** for HTTP(S) Load Balancing (a single anycast IP in front of backends across multiple regions) — a major architectural difference from AWS ALB, which is always regional.

---

## MODULE 4: Compute Engine (GCE)

### 4.1 Machine Types

| Family | Use Case |
|---|---|
| E2 | Cost-optimized, general-purpose |
| N2/N2D | Balanced, general-purpose (Intel/AMD) |
| C2/C3 | Compute-optimized |
| M1/M2/M3 | Memory-optimized (large in-memory DBs) |
| A2/A3 | GPU-accelerated (ML workloads) |
| T2A/T2D | ARM-based (Tau T2A on Ampere Altra) |

```bash
gcloud compute instances create my-vm \
  --zone=asia-south1-a --machine-type=e2-medium \
  --image-family=debian-12 --image-project=debian-cloud
```

---

### 4.2 Managed Instance Groups (MIGs) and Autoscaling

```
Instance Template (defines machine type, image, disks)
        │
Managed Instance Group (MIG) — maintains desired count across zones
        │
Autoscaler (based on CPU, load balancing capacity, or custom metrics)
```

```bash
gcloud compute instance-templates create my-template \
  --machine-type=e2-medium --image-family=debian-12 --image-project=debian-cloud

gcloud compute instance-groups managed create my-mig \
  --base-instance-name=my-app --template=my-template --size=2 --zone=asia-south1-a

gcloud compute instance-groups managed set-autoscaling my-mig \
  --zone=asia-south1-a --max-num-replicas=10 --target-cpu-utilization=0.7
```

**Regional MIGs** (spanning multiple zones within a region) are recommended over zonal MIGs for high availability — analogous to spreading an AWS Auto Scaling Group across multiple AZs.

---

### 4.3 Preemptible VMs vs Spot VMs

- **Spot VMs** (current recommendation) — like AWS Spot Instances, up to ~60-91% discount, can be reclaimed with a 30-second termination notice, no maximum runtime.
- **Preemptible VMs** (older/legacy) — similar discount, but hard-capped at 24 hours max runtime; being phased out in favor of Spot VMs.

```bash
gcloud compute instances create my-spot-vm --provisioning-model=SPOT \
  --instance-termination-action=STOP --zone=asia-south1-a
```

---

### 4.4 Live Migration — A GCP Differentiator

Unlike AWS (where host maintenance typically requires stopping/restarting an instance), GCP performs **live migration** by default for standard VMs during host maintenance events — your VM keeps running, transparently moved to different physical hardware, with no reboot required. This is a frequently-asked "what's different about GCP" interview point.

```bash
gcloud compute instances describe my-vm --zone=asia-south1-a --format="value(scheduling.onHostMaintenance)"
```

---

## MODULE 5: Google Kubernetes Engine (GKE)

### 5.1 GKE Architecture: Autopilot vs Standard

```
GKE Standard                          GKE Autopilot
      │                                     │
You manage node pools,                Google manages nodes,
node sizing, scaling                  capacity, and node-level
      │                                security entirely
Pay for provisioned VMs                     │
                                     Pay per pod resource request
                                     (vCPU/memory/pod-seconds)
```

- **GKE Standard** — you control node pools, machine types, and scaling logic — more flexibility, more operational responsibility (analogous to EKS with managed/self-managed node groups).
- **GKE Autopilot** — Google manages the entire node infrastructure; you only define Pod specs. No node-level access, but significantly reduced operational overhead and a hardened-by-default security posture. This is increasingly the recommended default for most workloads.

```bash
gcloud container clusters create-auto my-autopilot-cluster --region=asia-south1
gcloud container clusters create my-standard-cluster --zone=asia-south1-a --num-nodes=3
```

---

### 5.2 GKE Control Plane

Unlike EKS (where you always pay a per-cluster control plane fee), GKE offers **one zonal cluster free per billing account**, with regional clusters and additional zonal clusters incurring a management fee — an important cost-comparison talking point vs AWS/Azure.

The control plane (API server, etcd, scheduler) is fully managed by Google and, for regional clusters, is made highly available across multiple zones automatically.

---

### 5.3 Workload Identity (Recap in GKE Context)

```
Pod
 │
Kubernetes Service Account (KSA)
 │
Workload Identity binding
 │
GCP IAM Service Account (GSA)
 │
GCP API (Cloud Storage, Secret Manager, etc.)
```

This is the GKE equivalent of AWS IRSA, eliminating the need to mount long-lived service account key files into pods.

---

### 5.4 GKE Networking: VPC-Native Clusters

Modern GKE clusters are **VPC-native** by default — Pods and Services get IP addresses from secondary IP ranges within the VPC itself (via alias IPs), rather than an overlay network — giving Pods routable IPs directly usable by other GCP resources (e.g., firewall rules can target Pod IP ranges natively).

```bash
gcloud container clusters create my-cluster \
  --enable-ip-alias --cluster-secondary-range-name=pods \
  --services-secondary-range-name=services
```

---

### 5.5 GKE Autoscaling Layers

| Layer | Purpose |
|---|---|
| Horizontal Pod Autoscaler (HPA) | Scale pod replica count |
| Vertical Pod Autoscaler (VPA) | Adjust pod CPU/memory requests |
| Cluster Autoscaler | Add/remove nodes (Standard mode) |
| Node Auto-Provisioning (NAP) | Automatically create new node pools with appropriate machine types as needed |

```bash
kubectl autoscale deployment myapp --cpu-percent=70 --min=2 --max=10
gcloud container clusters update my-cluster --enable-autoscaling --min-nodes=1 --max-nodes=10
```

---

### 5.6 GKE Upgrades and Release Channels

```
Rapid Channel   → newest versions, least stable, early access
Regular Channel → balanced (default recommendation for most)
Stable Channel  → most tested, slower to receive new versions
```

```bash
gcloud container clusters update my-cluster --release-channel=regular
```

GKE can auto-upgrade both the control plane and node pools according to the selected release channel, with configurable maintenance windows to control *when* upgrades happen.

---

## MODULE 6: Serverless: Cloud Run, Cloud Functions, App Engine

### 6.1 Cloud Run

Cloud Run runs stateless containers in a fully serverless model — scales to zero, and scales out automatically based on incoming requests.

```bash
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-app:v1 \
  --region=asia-south1 --allow-unauthenticated \
  --min-instances=0 --max-instances=20
```

**Key advantage over Lambda-style FaaS:** Cloud Run runs **any containerized workload** (not restricted to a specific runtime/handler signature), making it easy to lift-and-shift existing containerized apps into a serverless model without significant rewrites.

---

### 6.2 Cloud Functions (1st gen vs 2nd gen)

- **1st gen** — simple, lightweight, event-driven single-purpose functions.
- **2nd gen** — built on Cloud Run under the hood, supporting longer timeouts, larger instances, concurrency, and more event sources (via Eventarc).

```bash
gcloud functions deploy my-function --gen2 \
  --runtime=python312 --trigger-http --region=asia-south1 --entry-point=main
```

---

### 6.3 App Engine (Standard vs Flexible)

- **Standard** — sandboxed runtime, fast scale-to-zero, limited language/runtime versions.
- **Flexible** — runs in Docker containers on Compute Engine VMs managed by App Engine, more flexibility but slower scale-up and no true scale-to-zero.

App Engine was GCP's original serverless PaaS offering; most new serverless workloads today are steered toward Cloud Run instead, but App Engine remains common in legacy GCP-native applications (a common interview/migration topic).

---

## MODULE 7: Storage: Cloud Storage, Persistent Disk, Filestore

### 7.1 Cloud Storage Classes

| Class | Use Case |
|---|---|
| Standard | Frequently accessed data |
| Nearline | Accessed roughly once a month |
| Coldline | Accessed roughly once a quarter |
| Archive | Accessed less than once a year, long-term archival |

```bash
gsutil mb -l asia-south1 gs://my-bucket
gsutil lifecycle set lifecycle.json gs://my-bucket
```

**Key GCS feature:** Object storage classes can be set **per-object** and transitioned automatically via lifecycle rules, similar to S3 lifecycle policies but with a simpler four-tier model.

---

### 7.2 Persistent Disk vs Local SSD vs Filestore

| Persistent Disk | Local SSD | Filestore |
|---|---|---|
| Network-attached block storage | Physically attached to the host, ephemeral | Managed NFS file storage |
| Survives instance deletion (if not deleted with instance) | Data lost on stop/terminate | Mountable by multiple instances/GKE pods concurrently |
| Analogous to AWS EBS | Analogous to AWS Instance Store | Analogous to AWS EFS |

```bash
gcloud compute disks create my-disk --size=100GB --zone=asia-south1-a --type=pd-ssd
gcloud filestore instances create my-filestore --zone=asia-south1-a --tier=BASIC_HDD --file-share=name="share1",capacity=1TB --network=name="my-vpc"
```

---

### 7.3 Securing Cloud Storage Buckets

- Enable **Uniform bucket-level access** (disables legacy per-object ACLs, enforcing consistent IAM-based access control).
- Use **Public Access Prevention** to guarantee a bucket can never become publicly accessible, even via misconfiguration.
- Enable **Object Versioning** and **Retention Policies** (with Bucket Lock for compliance/immutability, similar to S3 Object Lock).

```bash
gsutil uniformbucketlevelaccess set on gs://my-bucket
gsutil pap set enforced gs://my-bucket
gsutil versioning set on gs://my-bucket
```

---

## MODULE 8: Databases: Cloud SQL, Spanner, Firestore, BigQuery, Bigtable

### 8.1 Choosing the Right GCP Database

```
Relational, single-region        → Cloud SQL (MySQL/PostgreSQL/SQL Server)
Relational, globally distributed,
strongly consistent, horizontally
scalable                          → Cloud Spanner
Document/NoSQL, mobile/web apps   → Firestore
Wide-column NoSQL, huge scale,
low-latency (IoT, time-series)    → Bigtable
Analytics/data warehouse (OLAP)   → BigQuery
```

---

### 8.2 Cloud SQL High Availability

```bash
gcloud sql instances create my-instance \
  --database-version=POSTGRES_15 --tier=db-custom-2-8192 \
  --region=asia-south1 --availability-type=REGIONAL
```

`--availability-type=REGIONAL` provisions a synchronous standby in a different zone within the region — directly analogous to AWS RDS Multi-AZ.

```bash
gcloud sql instances create my-read-replica \
  --master-instance-name=my-instance --tier=db-custom-2-8192 --region=asia-south1
```

---

### 8.3 Cloud Spanner — GCP's Standout Differentiator

Cloud Spanner provides **horizontally scalable, globally distributed, strongly consistent relational database** capability — a combination that's genuinely hard to find equivalently elsewhere (AWS Aurora Global Database, by contrast, only supports a single write region at a time). This is one of the most commonly asked "what's unique about GCP" interview topics.

```bash
gcloud spanner instances create my-spanner-instance \
  --config=regional-asia-south1 --nodes=1 --description="My Spanner instance"
```

Spanner achieves global consistency using **TrueTime**, Google's globally synchronized clock infrastructure (using GPS and atomic clocks) to provide externally-consistent transaction ordering across regions.

---

### 8.4 BigQuery Fundamentals

BigQuery is a fully serverless, petabyte-scale data warehouse — you don't manage any compute infrastructure; you simply run SQL queries and pay per query (bytes scanned) or via flat-rate/capacity-based pricing.

```bash
bq query --use_legacy_sql=false 'SELECT COUNT(*) FROM `my-project.my_dataset.my_table`'
bq load --source_format=CSV my_dataset.my_table gs://my-bucket/data.csv schema.json
```

**Cost optimization tip:** Always query only the specific columns needed (BigQuery is columnar-storage, so `SELECT *` scans and bills for far more data than `SELECT col1, col2`), and use partitioned/clustered tables to prune scanned data.

---

### 8.5 Firestore vs Bigtable

| Firestore | Bigtable |
|---|---|
| Document model, strong consistency | Wide-column, eventual consistency across regions |
| Great for mobile/web app backends | Great for massive-scale time-series/IoT/analytics workloads |
| Automatic multi-region replication option | Manual replication configuration for multi-region |
| Simpler query model | Requires careful row-key design for performance |

```bash
gcloud firestore databases create --location=asia-south1
gcloud bigtable instances create my-bigtable --cluster=my-cluster --cluster-zone=asia-south1-a --display-name="My Bigtable"
```

---

## MODULE 9: Load Balancing & Auto Scaling

### 9.1 GCP Load Balancer Types

```
                    Global External LB
                  (HTTP(S), single anycast IP)
                            │
              ┌─────────────┼─────────────┐
        Backend (Region A)          Backend (Region B)
         (MIG / GKE / Cloud Run)     (MIG / GKE / Cloud Run)
```

| Load Balancer Type | Scope | Use Case |
|---|---|---|
| Global External HTTP(S) LB | Global | Web apps needing global anycast entry point |
| Regional External HTTP(S) LB | Regional | Region-scoped web apps |
| Internal HTTP(S) LB | Regional/VPC | Internal microservices routing |
| Network LB (TCP/UDP, pass-through) | Regional | Non-HTTP, high-performance TCP/UDP workloads |

```bash
gcloud compute backend-services create my-backend-service \
  --global --protocol=HTTP --health-checks=my-health-check

gcloud compute url-maps create my-lb --default-service=my-backend-service
gcloud compute target-http-proxies create my-proxy --url-map=my-lb
gcloud compute forwarding-rules create my-forwarding-rule \
  --global --target-http-proxy=my-proxy --ports=80
```

**Key differentiator:** GCP's Global External HTTP(S) Load Balancer is a single anycast IP address that automatically routes users to the *closest healthy region* with capacity — genuinely global load balancing out of the box, unlike AWS ALB (regional) which requires Route 53 latency-based routing across multiple regional ALBs to achieve similar behavior.

---

### 9.2 Health Checks

```bash
gcloud compute health-checks create http my-health-check \
  --port=8080 --request-path=/healthz --check-interval=10s --unhealthy-threshold=3
```

---

## MODULE 10: Security & Identity

### 10.1 Cloud KMS (Key Management Service)

```bash
gcloud kms keyrings create my-keyring --location=asia-south1
gcloud kms keys create my-key --keyring=my-keyring --location=asia-south1 --purpose=encryption
```

GCP encrypts data at rest by default for all customer data (even without explicit CMEK configuration) — Customer-Managed Encryption Keys (CMEK) via Cloud KMS let you control key rotation/access explicitly for compliance requirements, analogous to AWS KMS CMKs.

---

### 10.2 Secret Manager

```bash
gcloud secrets create db-password --replication-policy="automatic"
echo -n "supersecret" | gcloud secrets versions add db-password --data-file=-
gcloud secrets versions access latest --secret="db-password"
```

Directly analogous to AWS Secrets Manager, with native integration into Cloud Run, GKE (via Secret Manager CSI driver), and Cloud Functions.

---

### 10.3 VPC Service Controls

VPC Service Controls create a security perimeter around GCP resources (e.g., BigQuery, Cloud Storage) to prevent data exfiltration — even if IAM credentials are compromised, data cannot leave the defined perimeter to an untrusted network/project. This addresses a specific risk class (credential-based exfiltration) that IAM alone doesn't fully solve — a notable, frequently-tested GCP security concept without a precise 1:1 AWS analog (closest comparison is a combination of VPC endpoints + SCPs, but VPC-SC is more purpose-built for this exact scenario).

```bash
gcloud access-context-manager perimeters create my_perimeter \
  --resources=projects/123456789012 \
  --restricted-services=storage.googleapis.com,bigquery.googleapis.com
```

---

### 10.4 Security Command Center

Security Command Center is GCP's centralized security/risk dashboard — providing vulnerability scanning, misconfiguration detection, and threat detection (analogous to a combination of AWS Security Hub + GuardDuty + Inspector in one integrated product).

---

### 10.5 Binary Authorization

Binary Authorization enforces that only signed, verified container images (from a trusted build pipeline) can be deployed to GKE/Cloud Run — blocking unverified or tampered images at deploy time, directly analogous to AWS's image-signing + admission-control patterns discussed in supply-chain security.

```bash
gcloud container binauthz policy import policy.yaml
```

---

## MODULE 11: Monitoring & Operations Suite

### 11.1 Cloud Operations Suite Overview

```
Cloud Monitoring  → Metrics, Dashboards, Alerting (like CloudWatch Metrics/Alarms)
Cloud Logging     → Centralized logs (like CloudWatch Logs)
Cloud Trace       → Distributed tracing (like AWS X-Ray)
Cloud Profiler    → Continuous CPU/memory profiling in production
Error Reporting   → Automatic error aggregation/grouping from logs
```

```bash
gcloud logging read "resource.type=gce_instance AND severity>=ERROR" --limit=50
gcloud monitoring dashboards create --config-from-file=dashboard.json
```

---

### 11.2 Log-based Metrics and Alerting

```bash
gcloud logging metrics create error_count \
  --description="Count of ERROR logs" \
  --log-filter='severity=ERROR'

gcloud alpha monitoring policies create --policy-from-file=alert-policy.yaml
```

---

## MODULE 12: Cost Optimization

### 12.1 GCP-Specific Cost Optimization Levers

- **Sustained Use Discounts** — automatic discounts (no upfront commitment needed) applied simply for running a VM for a significant portion of the billing month — a GCP-specific advantage over AWS/Azure, which require explicit Reserved Instance/Savings Plan purchases to get similar discounts.
- **Committed Use Discounts (CUDs)** — 1 or 3-year commitments for predictable workloads (similar to AWS Reserved Instances/Savings Plans).
- **Spot VMs** — for fault-tolerant, interruptible workloads.
- **Recommender** — GCP's built-in service providing automated rightsizing/idle-resource recommendations (similar to AWS Compute Optimizer/Trusted Advisor).

```bash
gcloud recommender recommendations list \
  --project=my-project --location=asia-south1-a \
  --recommender=google.compute.instance.MachineTypeRecommender
```

---

### 12.2 Budget Alerts

```bash
gcloud billing budgets create --billing-account=BILLING_ACCOUNT_ID \
  --display-name="Monthly Budget" --budget-amount=5000USD \
  --threshold-rule=percent=0.9
```

---

## MODULE 13: Infrastructure as Code & CI/CD on GCP

### 13.1 Terraform on GCP

```hcl
provider "google" {
  project = "my-project"
  region  = "asia-south1"
}

resource "google_compute_instance" "vm" {
  name         = "my-vm"
  machine_type = "e2-medium"
  zone         = "asia-south1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-12"
    }
  }

  network_interface {
    network = "default"
  }
}
```

```bash
terraform init
terraform plan
terraform apply
```

---

### 13.2 Cloud Build (GCP's Native CI/CD Service)

```yaml
# cloudbuild.yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/my-app:$SHORT_SHA', '.']
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/my-app:$SHORT_SHA']
  - name: 'gcr.io/cloud-builders/gcloud'
    args: ['run', 'deploy', 'my-service', '--image', 'gcr.io/$PROJECT_ID/my-app:$SHORT_SHA', '--region', 'asia-south1']
```

```bash
gcloud builds submit --config=cloudbuild.yaml
```

Cloud Build is GCP's fully managed, serverless CI/CD service — directly analogous to AWS CodeBuild/CodePipeline, integrated natively with Artifact Registry, GKE, and Cloud Run.

---

### 13.3 Cloud Deploy (Managed Continuous Delivery for GKE/Cloud Run)

Cloud Deploy provides a managed, declarative continuous delivery pipeline specifically for progressing releases across environments (dev → staging → prod) on GKE/Cloud Run, with built-in approval gates and rollback — analogous to a managed version of Argo Rollouts + ArgoCD combined.

```bash
gcloud deploy apply --file=clouddeploy.yaml --region=asia-south1
gcloud deploy releases create my-release --delivery-pipeline=my-pipeline --region=asia-south1
```

---

## MODULE 14: High Availability & Disaster Recovery

### 14.1 GCP Multi-Region Architecture Example

```
                Global Users
                     │
         Global External HTTP(S) Load Balancer
              (single anycast IP)
              /                  \
       Region: asia-south1     Region: us-central1
       (GKE/Cloud Run + MIG)   (GKE/Cloud Run + MIG)
              │                       │
       Cloud Spanner (multi-region config — single global database)
              │
       Cloud Storage (multi-region bucket)
```

Because Cloud Spanner natively supports multi-region configurations with a single logical database spanning regions (unlike needing separate regional databases + replication), and the Global HTTP(S) Load Balancer is inherently multi-region, GCP can achieve certain multi-region active-active architectures with notably less custom plumbing than the equivalent AWS design.

---

### 14.2 Backup and DR Strategies on GCP

```bash
gcloud sql backups create --instance=my-instance
gcloud compute disks snapshot my-disk --zone=asia-south1-a --snapshot-names=my-snapshot
gsutil -m rsync -r gs://primary-bucket gs://dr-bucket   # cross-region sync
```

Same four DR strategy tiers apply conceptually as in AWS (Backup & Restore, Pilot Light, Warm Standby, Multi-Site Active-Active) — the specific GCP tools differ (Cloud SQL cross-region replicas, Spanner multi-region configs, GCS multi-region/dual-region buckets, Persistent Disk snapshots with cross-region snapshot storage).

---

## Summary Cheat-Sheet: Most-Used `gcloud` Commands

```bash
# Auth & Config
gcloud auth login
gcloud config set project my-project

# Compute
gcloud compute instances list
gcloud compute instances create my-vm --zone=asia-south1-a --machine-type=e2-medium

# GKE
gcloud container clusters get-credentials my-cluster --zone=asia-south1-a
kubectl get nodes

# Storage
gsutil ls gs://my-bucket
gsutil cp file.txt gs://my-bucket/

# IAM
gcloud projects get-iam-policy my-project
gcloud iam service-accounts list

# Cloud SQL
gcloud sql instances list
gcloud sql connect my-instance --user=root

# Logging & Monitoring
gcloud logging read "severity>=ERROR" --limit=20
gcloud monitoring dashboards list

# Cloud Build / Deploy
gcloud builds submit --config=cloudbuild.yaml
gcloud deploy releases list --delivery-pipeline=my-pipeline --region=asia-south1
```

---

## Quick GCP ↔ AWS ↔ Azure Service Mapping (for cross-cloud interview context)

| Category | AWS | GCP | Azure |
|---|---|---|---|
| Compute | EC2 | Compute Engine | Virtual Machines |
| Kubernetes | EKS | GKE | AKS |
| Serverless Functions | Lambda | Cloud Functions | Azure Functions |
| Serverless Containers | Fargate | Cloud Run | Container Apps |
| Object Storage | S3 | Cloud Storage | Blob Storage |
| Relational DB | RDS | Cloud SQL | Azure SQL Database |
| Globally-distributed relational DB | Aurora Global DB (single-write) | Cloud Spanner (multi-write) | Cosmos DB (multi-model) |
| Data Warehouse | Redshift | BigQuery | Synapse Analytics |
| IAM | IAM | Cloud IAM | Azure AD / Entra ID |
| Secrets | Secrets Manager | Secret Manager | Key Vault |
| IaC (native) | CloudFormation | Deployment Manager / Config Connector | ARM/Bicep |
| CI/CD (native) | CodePipeline/CodeBuild | Cloud Build/Cloud Deploy | Azure DevOps/Pipelines |

---
