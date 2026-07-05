# GCP Troubleshooting Guide
---

## Table of Contents

1. [General Troubleshooting Methodology](#module-1-general-troubleshooting-methodology)
2. [Compute Engine (GCE) Troubleshooting](#module-2-compute-engine-gce-troubleshooting)
3. [GKE Troubleshooting](#module-3-gke-troubleshooting)
4. [VPC & Networking Troubleshooting](#module-4-vpc--networking-troubleshooting)
5. [IAM & Permissions Troubleshooting](#module-5-iam--permissions-troubleshooting)
6. [Cloud Storage Troubleshooting](#module-6-cloud-storage-troubleshooting)
7. [Cloud SQL / Database Troubleshooting](#module-7-cloud-sql--database-troubleshooting)
8. [Cloud Run & Cloud Functions Troubleshooting](#module-8-cloud-run--cloud-functions-troubleshooting)
9. [Load Balancer & CDN Troubleshooting](#module-9-load-balancer--cdn-troubleshooting)
10. [Cloud Build / CI-CD Troubleshooting](#module-10-cloud-build--cicd-troubleshooting)
11. [Billing & Quota Troubleshooting](#module-11-billing--quota-troubleshooting)
12. [Monitoring & Logging Troubleshooting](#module-12-monitoring--logging-troubleshooting)

---

## MODULE 1: General Troubleshooting Methodology

### 1.1 The Layered Troubleshooting Approach

```
User Report / Alert
        │
1. Scope the blast radius (all users? one region? one service?)
        │
2. Check Cloud Monitoring dashboards (metrics anomaly?)
        │
3. Check Cloud Logging (errors, stack traces, audit logs)
        │
4. Check the specific resource (gcloud describe, kubectl describe)
        │
5. Check IAM / network path (permissions, firewall, routing)
        │
6. Reproduce in isolation if possible
        │
7. Fix, validate, document (postmortem if customer-impacting)
```

Always start broad (is this a GCP-wide status page issue?) before diving into your own configuration — check https://status.cloud.google.com first for any active incidents that might explain the symptom before spending time debugging your own resources.

---

### 1.2 Essential Diagnostic Commands to Always Have Ready

```bash
gcloud config list                                  # confirm active project/account context
gcloud projects get-iam-policy $(gcloud config get-value project)  # who has access
gcloud logging read "severity>=ERROR" --limit=50 --freshness=1h    # recent errors
gcloud compute operations list --filter="status=DONE AND operationType!=compute.instances.get" --limit=20  # recent changes
```

---

## MODULE 2: Compute Engine (GCE) Troubleshooting

### 2.1 VM Won't Start / Stuck in "Provisioning"

```bash
gcloud compute instances describe my-vm --zone=asia-south1-a --format="value(status,statusMessage)"
gcloud compute operations list --filter="targetLink~my-vm"
```

**Common causes:** Regional resource quota exhausted (check `gcloud compute regions describe asia-south1`), requested machine type unavailable in that zone, or a Persistent Disk in a different zone than the VM being attached.

---

### 2.2 Cannot SSH Into a VM

```bash
gcloud compute instances get-serial-port-output my-vm --zone=asia-south1-a
gcloud compute firewall-rules list --filter="name~allow-ssh"
gcloud compute ssh my-vm --zone=asia-south1-a --troubleshoot
```

**Checklist:**
- Firewall rule allows TCP:22 from your source IP/range.
- VM has an external IP (or you're using IAP tunneling / within the VPC).
- OS Login vs metadata-based SSH keys — check `enable-oslogin` metadata setting isn't conflicting with manually-added keys.
- For IAP-based SSH: confirm `roles/iap.tunnelResourceAccessor` is granted and firewall allows ingress from `35.235.240.0/20`.

```bash
# Use IAP tunneling instead of external IP (recommended, more secure)
gcloud compute ssh my-vm --zone=asia-south1-a --tunnel-through-iap
```

---

### 2.3 VM Serial Console Shows Boot Loop / Kernel Panic

```bash
gcloud compute instances get-serial-port-output my-vm --zone=asia-south1-a > serial.log
```

Look for filesystem corruption messages, a bad kernel update, or `fsck` failures. Recovery: attach the boot disk to a temporary rescue VM as a secondary disk, `fsck`/repair it, then reattach it as the boot disk to the original VM.

```bash
gcloud compute instances detach-disk my-vm --disk=my-vm-boot --zone=asia-south1-a
gcloud compute instances attach-disk rescue-vm --disk=my-vm-boot --zone=asia-south1-a
```

---

### 2.4 Live Migration Causing Unexpected Application Behavior

If an app has issues specifically correlated with host maintenance events, check whether it assumes a stable underlying clock/hardware or is sensitive to the brief pause during live migration.

```bash
gcloud compute instances describe my-vm --zone=asia-south1-a --format="value(scheduling.onHostMaintenance)"
```
For workloads that genuinely can't tolerate live migration (e.g., certain latency-sensitive HPC workloads), set `--maintenance-policy=TERMINATE` so the VM is stopped/restarted instead of live-migrated, and rely on your own HA/redundancy instead.

---

### 2.5 Instance Metadata / Startup Script Not Running

```bash
gcloud compute instances describe my-vm --zone=asia-south1-a --format="value(metadata.items)"
sudo journalctl -u google-startup-scripts.service    # inside the VM
```
Common cause: startup script has a syntax error (it fails silently unless you check the systemd unit logs), or metadata was updated but the VM needs a restart to re-trigger it (startup scripts run on every boot, not on metadata update alone).

---

## MODULE 3: GKE Troubleshooting

### 3.1 Pod Stuck in Pending

```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

**Common causes:** insufficient node capacity (check `gcloud container clusters describe my-cluster` autoscaling limits and Node Auto-Provisioning config), unsatisfied `nodeSelector`/affinity/taints, or no PersistentVolume matching a pending PVC.

---

### 3.2 Pod CrashLoopBackOff

```bash
kubectl logs <pod-name> -n <namespace> --previous
kubectl describe pod <pod-name> -n <namespace>
```
Check for `OOMKilled` in the describe output (increase memory limits), missing ConfigMap/Secret keys the app expects, or a failing readiness/liveness probe misconfigured against the wrong path/port.

---

### 3.3 Cannot Pull Image from Artifact Registry

```bash
kubectl describe pod <pod-name> -n <namespace>   # look for ImagePullBackOff / ErrImagePull
gcloud artifacts docker images list asia-south1-docker.pkg.dev/my-project/my-repo
```
**Common causes:** GKE node service account lacks `roles/artifactregistry.reader` on the target repo, incorrect image path/tag, or the Artifact Registry repo is in a different project without cross-project IAM binding granted.

```bash
gcloud projects add-iam-policy-binding my-project \
  --member="serviceAccount:GKE_NODE_SA@my-project.iam.gserviceaccount.com" \
  --role="roles/artifactregistry.reader"
```

---

### 3.4 Workload Identity Not Working — Pod Can't Access GCP APIs

```bash
kubectl describe sa my-ksa -n my-namespace     # confirm the annotation is present
gcloud iam service-accounts get-iam-policy my-app-sa@my-project.iam.gserviceaccount.com
```
Checklist: the Kubernetes Service Account has the correct `iam.gke.io/gcp-service-account` annotation, the IAM binding grants `roles/iam.workloadIdentityUser` scoped to the exact `PROJECT_ID.svc.id.goog[namespace/ksa-name]` member string, and the Pod spec actually references the annotated KSA (not `default`).

---

### 3.5 GKE Cluster Autoscaler Not Scaling Up

```bash
kubectl get events -A | grep -i "TriggeredScaleUp\|NotTriggerScaleUp"
gcloud container clusters describe my-cluster --format="value(autoscaling)"
```
Common causes: max node count already reached for the node pool, requested machine type/quota not available in remaining zones, or Node Auto-Provisioning constraints (min/max CPU/memory bounds) too narrow for the pending pod's requirements.

---

### 3.6 GKE Cluster Autoscaler Not Scaling Down

Check for pods blocking eviction: pods without a controller, pods using local storage (`emptyDir`), restrictive PodDisruptionBudgets, or the `cluster-autoscaler.kubernetes.io/safe-to-evict: "false"` annotation.

```bash
kubectl get pods -A -o json | jq '.items[] | select(.metadata.annotations."cluster-autoscaler.kubernetes.io/safe-to-evict"=="false")'
```

---

### 3.7 DNS Resolution Failing Intermittently Inside the Cluster

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl top pods -n kube-system -l k8s-app=kube-dns
kubectl exec -it <pod> -- nslookup my-service.my-namespace.svc.cluster.local
```
Check kube-dns/CoreDNS resource limits/CPU throttling under load; consider enabling **NodeLocal DNSCache** to reduce load on the central DNS pods and cut resolution latency.

---

### 3.8 GKE Upgrade Failing or Nodes Stuck Draining

```bash
gcloud container operations list --filter="targetLink~my-cluster"
kubectl get nodes -o wide
kubectl describe node <node-name>
```
Check for PodDisruptionBudgets preventing the required number of pod evictions, or a `preStop` hook/graceful shutdown taking longer than the node drain timeout — increase `terminationGracePeriodSeconds` if legitimate, or investigate why the app is slow to shut down.

---

## MODULE 4: VPC & Networking Troubleshooting

### 4.1 Instance Can't Reach the Internet

```bash
gcloud compute routes list --filter="network=my-vpc"
gcloud compute routers nats describe my-nat --router=my-router --region=asia-south1
```
**Checklist:** instance has no external IP → confirm Cloud NAT is configured for its subnet range; check firewall egress rules aren't unexpectedly restrictive (default is allow-all-egress, but custom rules may have overridden this); confirm the default route (`0.0.0.0/0`) to the default internet gateway exists and wasn't deleted.

---

### 4.2 Two VMs in the Same VPC Can't Communicate

```bash
gcloud compute firewall-rules list --format="table(name,direction,sourceRanges.list(),targetTags.list(),allowed[].map().firewall_rule().list())"
```
Remember GCP firewall rules apply based on **network + tags/service accounts**, not subnet attachment — check that both the ingress rule on the destination and any explicit egress-deny rules on the source allow the specific traffic, and confirm both instances are actually in the intended VPC/subnet (`gcloud compute instances describe`).

---

### 4.3 VPC Peering Connection Shows "INACTIVE"

```bash
gcloud compute networks peerings list --network=my-vpc
```
Common cause: overlapping IP CIDR ranges between the two VPCs (peering will not activate if ranges overlap), or the peering wasn't established from **both** sides (VPC Peering requires a matching peering configuration created independently in each VPC).

---

### 4.4 Cloud VPN / Interconnect Tunnel Down

```bash
gcloud compute vpn-tunnels describe my-tunnel --region=asia-south1
gcloud compute routers get-status my-router --region=asia-south1
```
Check BGP session status specifically (`get-status` shows BGP peer state) — a tunnel can show "established" at the IPSec layer while BGP routing has failed to converge, meaning no actual traffic flows despite the tunnel appearing "up."

---

### 4.5 Shared VPC: Service Project Can't Create Resources in the Host VPC

```bash
gcloud compute shared-vpc get-host-project SERVICE_PROJECT_ID
gcloud projects get-iam-policy HOST_PROJECT_ID
```
Checklist: the service project is actually attached (`shared-vpc associated-projects list` on the host), and the specific user/service account has `roles/compute.networkUser` granted **on the specific subnet** (or the host project) — not just general project-level Editor access, which does not automatically grant Shared VPC subnet usage rights.

---

### 4.6 Latency Spikes Between Regions on the Same VPC

```bash
gcloud compute networks describe my-vpc --format="value(routingConfig.routingMode)"
```
Check whether the VPC's routing mode is `GLOBAL` (routes visible to all regions) vs `REGIONAL` (routes only visible within the region they were created in) — a `REGIONAL` routing mode can cause traffic to take an unintended, longer path between regions if routes aren't propagated as expected.

---

## MODULE 5: IAM & Permissions Troubleshooting

### 5.1 "Permission Denied" Despite Assigning the Correct Role

```bash
gcloud projects get-iam-policy my-project --flatten="bindings[].members" --filter="bindings.members:user:john@example.com"
gcloud organizations get-iam-policy ORG_ID
```
Check for a **Deny policy** or a more restrictive Organization Policy overriding the allow — GCP resolves effective permissions as a combination of Allow policies (which are additive/union across the hierarchy) AND any explicit Deny policies (which take precedence regardless of Allow bindings) plus Organization Policy constraints, so an Allow at the project level can still be blocked by a Deny or Org Policy set higher up.

```bash
gcloud iam policies list --attachment-point=organizations/ORG_ID  # check for Deny policies
```

---

### 5.2 Service Account Impersonation Failing

```bash
gcloud iam service-accounts get-iam-policy TARGET_SA@my-project.iam.gserviceaccount.com
```
Confirm the calling identity has `roles/iam.serviceAccountTokenCreator` granted specifically **on the target service account** (not just on the project) — this is a very common gap, since project-level IAM roles don't automatically grant impersonation rights on individual service accounts.

---

### 5.3 Debugging "Why Was This Action Denied" Using Policy Troubleshooter

```bash
gcloud policy-intelligence troubleshoot-iam \
  --principal-email=john@example.com \
  --resource=//cloudresourcemanager.googleapis.com/projects/my-project \
  --permission=compute.instances.get
```
GCP's Policy Troubleshooter (Policy Analyzer) explains exactly which binding (or the absence of one, or a Deny policy) is responsible for a given permission check's outcome — always more reliable than manually eyeballing a long list of IAM bindings.

---

### 5.4 Org Policy Blocking Resource Creation Unexpectedly

```bash
gcloud resource-manager org-policies list --project=my-project
gcloud resource-manager org-policies describe compute.vmExternalIpAccess --project=my-project
```
Common cause: a constraint (e.g., "restrict VM external IPs" or "restrict resource locations") was applied at the Organization/Folder level and is inherited down, blocking an action that appears otherwise permitted by IAM roles — Org Policies and IAM are separate, independent layers, and both must allow an action for it to succeed.

---

## MODULE 6: Cloud Storage Troubleshooting

### 6.1 "403 Access Denied" When Reading/Writing a Bucket

```bash
gsutil iam get gs://my-bucket
gcloud projects get-iam-policy my-project --filter="bindings.members:serviceAccount:my-sa@my-project.iam.gserviceaccount.com"
```
Checklist: confirm Uniform Bucket-Level Access is enabled (if it's disabled, legacy ACLs may be interfering unpredictably with IAM), and check for **Public Access Prevention** or an Org Policy blocking public/cross-project access if the requester is external to the project.

---

### 6.2 Object Uploaded but Not Visible / 404 on GET

```bash
gsutil ls -L gs://my-bucket/my-object
```
Common causes: object was uploaded to a different bucket/path than expected (case-sensitive path typo), a Retention Policy/Object Hold is present but shouldn't affect visibility (only deletion), or eventual consistency in list operations immediately after a very rapid write (rare, but `gsutil ls -L` on the specific known object path is always authoritative over list/browse operations).

---

### 6.3 Cross-Region Latency Reading from a Bucket

```bash
gsutil ls -L -b gs://my-bucket | grep "Location"
```
Confirm the bucket's location/storage class matches where your compute is actually running — a `US` multi-region bucket accessed heavily from `asia-south1` compute will have materially higher latency than a bucket located in (or dual-region including) `asia-south1`.

---

### 6.4 Lifecycle Rule Not Deleting/Transitioning Objects as Expected

```bash
gsutil lifecycle get gs://my-bucket
```
Common cause: the lifecycle condition (e.g., `age`, `createdBefore`) doesn't match objects' actual metadata timestamps as expected, or Object Versioning is enabled and the rule was written for "live" objects but needs a separate condition (`isLive: false`) to target noncurrent versions specifically.

---

## MODULE 7: Cloud SQL / Database Troubleshooting

### 7.1 Cloud SQL Instance Unreachable from an Application

```bash
gcloud sql instances describe my-instance --format="value(ipAddresses,settings.ipConfiguration)"
gcloud sql connect my-instance --user=root
```
Checklist: confirm whether the app is using Private IP (requires VPC peering/Private Service Access configured between the app's VPC and Cloud SQL) or Public IP (requires the app's IP in Authorized Networks, or use the **Cloud SQL Auth Proxy** which handles this via IAM instead of IP allowlisting).

---

### 7.2 High CPU / Slow Queries on Cloud SQL

```bash
gcloud sql operations list --instance=my-instance
```
Use Cloud SQL Insights (built-in query performance dashboard) to identify the specific slow query, check for missing indexes, and verify the instance tier hasn't become undersized for current load — `gcloud sql instances patch my-instance --tier=db-custom-4-16384` to scale up if genuinely resource-constrained (note: this requires a brief restart unless using regional HA with minimal-downtime failover).

---

### 7.3 Replication Lag on a Read Replica

```bash
gcloud sql instances describe my-read-replica --format="value(replicaConfiguration)"
```
Check `SHOW SLAVE STATUS` (MySQL) or replication lag metrics in Cloud Monitoring — common causes include the replica being undersized relative to the primary's write volume, or long-running transactions on the replica blocking replication apply threads.

---

### 7.4 Cloud SQL Failover Not Happening Automatically

```bash
gcloud sql instances describe my-instance --format="value(settings.availabilityType,state)"
```
Confirm `availabilityType` is actually `REGIONAL` (not `ZONAL`) — a zonal instance has no standby and cannot automatically fail over; this is a common misconfiguration mistaken for "HA is broken" when HA was simply never enabled.

---

### 7.5 Cloud Spanner Hotspotting (Uneven Load Across Nodes)

```bash
gcloud spanner instances describe my-spanner-instance
```
Use the Spanner Query Statistics and Key Visualizer tooling in the console to identify hot rows/splits — typically caused by monotonically increasing primary keys (e.g., auto-incrementing IDs or timestamps as the leading key column) concentrating writes on a single split; fix by using a hash-prefixed or UUID-based key design to distribute writes evenly.

---

## MODULE 8: Cloud Run & Cloud Functions Troubleshooting

### 8.1 Cloud Run Service Returns 503 / Container Failed to Start

```bash
gcloud run services describe my-service --region=asia-south1
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=my-service" --limit=50
```
Common causes: the container doesn't listen on the `PORT` environment variable Cloud Run injects (must bind to `0.0.0.0:$PORT`, not a hardcoded port), the container takes longer to start than the configured startup probe timeout, or insufficient memory allocated causing an OOM crash on startup.

---

### 8.2 Cloud Run Cold Starts Causing Latency Spikes

```bash
gcloud run services describe my-service --region=asia-south1 --format="value(spec.template.metadata.annotations)"
```
Set `--min-instances=1` (or higher) to keep warm instances always available, reducing cold starts at the cost of paying for idle capacity — a direct tradeoff decision between cost and latency consistency, same conceptual tradeoff as Lambda Provisioned Concurrency.

---

### 8.3 Cloud Function Timing Out

```bash
gcloud functions describe my-function --gen2 --region=asia-south1 --format="value(serviceConfig.timeoutSeconds)"
```
Increase the timeout (`--timeout`) up to the service's maximum, or (better) investigate why the function is slow — often a downstream dependency (Cloud SQL, an external API) with no timeout/retry logic causing the function to hang until the platform's own timeout kills it.

---

### 8.4 Cloud Run / Functions Can't Reach a Private VPC Resource (e.g., Cloud SQL Private IP, internal service)

```bash
gcloud compute networks vpc-access connectors describe my-connector --region=asia-south1
```
Cloud Run/Functions are serverless and not natively inside your VPC — confirm a **Serverless VPC Access connector** is configured and attached to the service (`--vpc-connector=my-connector`), which is required to reach any private-IP resource inside your VPC.

---

## MODULE 9: Load Balancer & CDN Troubleshooting

### 9.1 Backend Showing "UNHEALTHY" in Load Balancer

```bash
gcloud compute backend-services get-health my-backend-service --global
gcloud compute health-checks describe my-health-check
```
Checklist: confirm the health check's expected path/port actually returns a 200 from the backend, firewall rules allow the health-check probe IP ranges (`35.191.0.0/16`, `130.211.0.0/22`) specifically — a very common miss, since these ranges are easy to forget when writing custom firewall rules.

---

### 9.2 Global Load Balancer Routing Users to the Wrong (Farther) Region

```bash
gcloud compute backend-services describe my-backend-service --global --format="value(backends)"
```
Check backend capacity settings (`--max-utilization`, `--capacity-scaler`) — if the nearest region's backend is at its configured capacity limit, GCP's load balancer will route overflow traffic to the next-closest healthy region with available capacity, which can look like "wrong region routing" but is actually working as designed under load.

---

### 9.3 Cloud CDN Serving Stale Content After a Deploy

```bash
gcloud compute url-maps invalidate-cdn-cache my-lb --path="/*"
```
Confirm cache-control headers from the origin are set appropriately for your intended caching behavior, and explicitly invalidate the CDN cache after deployments that change cached content if your cache-control policy doesn't already handle this via versioned URLs/cache-busting.

---

### 9.4 SSL Certificate Provisioning Stuck in "PROVISIONING" (Google-Managed Certs)

```bash
gcloud compute ssl-certificates describe my-cert --global --format="value(managed.status,managed.domainStatus)"
```
Google-managed certificates require the domain's DNS to actually point to the load balancer's IP **before** provisioning can complete — if DNS propagation is incomplete or the domain points elsewhere, provisioning will remain stuck indefinitely rather than failing outright; verify DNS resolution first.

---

## MODULE 10: Cloud Build / CI-CD Troubleshooting

### 10.1 Cloud Build Fails with Permission Denied Pushing to Artifact Registry

```bash
gcloud projects get-iam-policy my-project --filter="bindings.members:serviceAccount:PROJECT_NUMBER@cloudbuild.gserviceaccount.com"
```
Confirm the Cloud Build service account has `roles/artifactregistry.writer` on the target repository — this role isn't granted by default and is a very common first-time setup gap.

---

### 10.2 Cloud Build Trigger Not Firing on Push

```bash
gcloud builds triggers describe my-trigger --region=asia-south1
```
Check the trigger's branch/tag filter regex actually matches your push, confirm the connected repository's webhook/connection status (for GitHub App-based connections, check the GitHub App installation hasn't been revoked), and check IAM permissions on the Cloud Build service account for reading the source repo.

---

### 10.3 Cloud Deploy Release Stuck in "Pending Approval"

```bash
gcloud deploy rollouts list --delivery-pipeline=my-pipeline --release=my-release --region=asia-south1
```
Confirm whether the target environment's `requireApproval: true` setting is intentional, and that the approving principal has `roles/clouddeploy.approver` — a release will sit indefinitely if no one with the correct role is aware an approval is pending (add Cloud Monitoring alerting on pending-approval state to avoid silent stalls).

---

## MODULE 11: Billing & Quota Troubleshooting

### 11.1 Unexpected Cost Spike Overnight

```bash
gcloud billing accounts list
# Then use Cloud Billing reports / BigQuery billing export for detailed breakdown
bq query --use_legacy_sql=false 'SELECT service.description, SUM(cost) as total_cost FROM `billing_export.gcp_billing_export_v1_XXXX` WHERE DATE(usage_start_time) = CURRENT_DATE() GROUP BY 1 ORDER BY 2 DESC'
```
Enable BigQuery billing export in advance (you can't retroactively query cost breakdowns without it) — cross-reference the spike with Cloud Audit Logs for any resource creation events around the same time (a runaway autoscaler, an accidentally-public resource being abused, forgotten large instances).

---

### 11.2 "Quota Exceeded" Errors Blocking Resource Creation

```bash
gcloud compute regions describe asia-south1 --format="value(quotas)"
gcloud compute project-info describe --project=my-project
```
Identify the specific exhausted quota (CPUs, IP addresses, specific API request quotas) from the error message, and request a quota increase via the Console (IAM & Admin → Quotas) or `gcloud alpha quotas` — quota increases are not instant and may require justification for large requests.

---

### 11.3 Budget Alert Fired But No One Was Notified

```bash
gcloud billing budgets list --billing-account=BILLING_ACCOUNT_ID
gcloud billing budgets describe BUDGET_ID --billing-account=BILLING_ACCOUNT_ID
```
Confirm the budget's notification channels (Pub/Sub topic or email recipients) are correctly configured and that any downstream Pub/Sub-triggered automation (e.g., a Cloud Function meant to page the team) is actually deployed and functioning — a budget alert firing internally in GCP's billing system doesn't guarantee your custom notification pipeline actually delivered it.

---

## MODULE 12: Monitoring & Logging Troubleshooting

### 12.1 Logs Not Appearing in Cloud Logging

```bash
gcloud logging sinks list
gcloud projects get-iam-policy my-project --filter="bindings.members:serviceAccount:cloud-logs@system.gserviceaccount.com"
```
Check whether a log **exclusion filter** was inadvertently configured (silently dropping logs matching a pattern before they're stored/billed), and confirm the resource's logging agent (for GCE, the Ops Agent) is actually installed and running if logs are expected from a VM rather than a fully-managed service.

---

### 12.2 Alerting Policy Not Firing Despite Metric Clearly Breaching Threshold

```bash
gcloud alpha monitoring policies list
gcloud alpha monitoring policies describe POLICY_ID
```
Check the alignment period and "for" duration configuration — a policy requiring the condition to be true for a sustained period (e.g., 5 minutes) won't fire on a brief spike that recovers within that window; also confirm the notification channel itself (Slack/PagerDuty/email integration) hasn't had its own auth/webhook expire silently.

---

### 12.3 Cloud Trace Missing Spans / Incomplete Traces

```bash
gcloud services list --enabled --filter="name:cloudtrace.googleapis.com"
```
Confirm the Cloud Trace API is enabled and the application's OpenTelemetry/Trace SDK is correctly configured with sampling rate not set too low (a low sampling rate can make it look like "traces are missing" when they're actually just not being captured for most requests by design) — and check trace context propagation headers are correctly forwarded across service boundaries (a common gap when a load balancer or proxy strips custom headers).

---

## Quick Reference: Most Useful GCP Troubleshooting Commands

```bash
# Status & Health
gcloud compute instances describe <vm> --zone=<zone> --format="value(status,statusMessage)"
gcloud compute backend-services get-health <backend-service> --global

# Logs
gcloud logging read "severity>=ERROR" --limit=50 --freshness=1h
kubectl logs <pod> -n <namespace> --previous

# IAM Debugging
gcloud projects get-iam-policy <project> --flatten="bindings[].members" --filter="bindings.members:<principal>"
gcloud policy-intelligence troubleshoot-iam --principal-email=<user> --resource=<resource> --permission=<permission>

# Networking
gcloud compute firewall-rules list
gcloud compute routes list
gcloud compute networks peerings list --network=<vpc>

# GKE
kubectl describe pod <pod> -n <namespace>
kubectl get events -A --sort-by='.lastTimestamp'

# Serial Console (GCE boot issues)
gcloud compute instances get-serial-port-output <vm> --zone=<zone>

# Quotas & Billing
gcloud compute regions describe <region> --format="value(quotas)"
gcloud billing budgets list --billing-account=<account-id>
```

---
