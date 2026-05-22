# what is k8s?
Kubernetes (often called K8s) is an open-source container orchestration platform used to automate deployment, scaling, and management of containerized applications.

# Difference between Docker swarm Vs k8s
| Feature              | Docker Swarm 🐳              | Kubernetes ☸️                  |
|----------------------|------------------------------|--------------------------------|
| Setup                | Easy & quick                 | Complex                        |
| Learning Curve       | Low                          | High                           |
| Scalability          | Limited                      | Highly scalable                |
| Auto Scaling         | Basic                        | Advanced (HPA, VPA)            |
| Load Balancing       | Built-in                     | Advanced                       |
| Deployment           | Simple                       | Flexible (many options)        |
| Networking           | Simple overlay               | Advanced networking            |
| Community Support    | Smaller                      | Very large                     |
| Self-Healing         | Basic                        | Strong (auto-restart, reschedule) |
| Ecosystem            | Limited                      | Huge (Helm, Istio, etc.)       |
| Use Case             | Small projects               | Enterprise / production        |

# Architecture  Docker swarm & k8s
```hcl
Docker Swarm                    Kubernetes
--------------                  ------------------
Manager Node                   Master Node
   │                               │
Worker Nodes                   Worker Nodes
   │                               │
Containers                     Pods (Containers)
```
# ☸️ Kubernetes & Docker — Complete Interview Guide

> **Covers:** Troubleshooting · Scaling · Security · Networking · Storage · Monitoring · Scenarios
> **Duplicates removed** · All answers in English · Modular sections

---

## 📁 Table of Contents

| # | Section | Topics |
|---|---------|--------|
| 1 | [Pod Troubleshooting](#1-pod-troubleshooting) | Failed pods, CrashLoopBackOff, restarts |
| 2 | [Resource Management](#2-resource-management) | CPU, memory, HPA, optimization |
| 3 | [Deployments & Updates](#3-deployments--updates) | Rolling, blue-green, zero-downtime |
| 4 | [Node Management](#4-node-management) | Node failure, NotReady, upgrades |
| 5 | [Networking](#5-networking) | Pod networking, CoreDNS, overlay, ingress |
| 6 | [Storage](#6-storage) | PVC pending, persistent storage failures |
| 7 | [Security & RBAC](#7-security--rbac) | Secrets, RBAC, cluster hardening |
| 8 | [Monitoring & Observability](#8-monitoring--observability) | Metrics, logs, alerts |
| 9 | [Disaster Recovery](#9-disaster-recovery) | Backup, restore, multi-env |
| 10 | [Docker Specific](#10-docker-specific) | Image issues, networking, local vs K8s |
| 11 | [Scenario-Based Questions](#11-scenario-based-questions) | 35 real-world scenarios with answers |

---

## Duplicates Removed

The following 8 overlapping questions were merged into single comprehensive answers:

| Removed (duplicate) | Merged into |
|---------------------|-------------|
| "pods restart continuously" | Pod Troubleshooting (Q1) |
| "Docker containers exit immediately" | CrashLoopBackOff (Q4) |
| "Docker networking fails between containers" | Networking (Q9) |
| "ImagePullBackOff occurs in production" | Image pull errors (Q16) |
| "ingress suddenly stops routing traffic" | Ingress troubleshooting (Q18) |
| "PVC remains stuck in Pending state" | Storage failures (Q10) |
| "rolling deployment causes temporary downtime" | Rolling updates (Q13) |
| "manage zero-downtime deployments" | Rolling updates (Q13) |

---

## 1. Pod Troubleshooting

### Q1. How would you troubleshoot a failed Pod / pods restarting continuously?

**Answer:**

When a pod fails or keeps restarting, follow this systematic approach:

**Step 1 — Check pod status:**
```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
```

Look for: `Status`, `Restart Count`, `Last State`, `Events` section at the bottom.

**Step 2 — Check pod logs:**
```bash
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous   # logs from last crash
kubectl logs <pod-name> -c <container-name>         # multi-container pod
```

**Step 3 — Common causes and fixes:**

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `OOMKilled` | Memory limit too low | Increase `resources.limits.memory` |
| `Error 1` / `Error 137` | Application crash | Check app logs for errors |
| `CrashLoopBackOff` | App keeps crashing | Fix app code / config |
| `Pending` | No node has capacity | Scale cluster or reduce requests |
| `ImagePullBackOff` | Image not found | Check image name/tag |
| `ContainerCreating` | Volume mount issue | Check PVC, ConfigMap, Secrets |

**Step 4 — Check events:**
```bash
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

**Step 5 — Exec into pod for debugging:**
```bash
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh
```

**Step 6 — Check resource usage:**
```bash
kubectl top pod <pod-name> -n <namespace>
```

---

### Q2. How would you troubleshoot CrashLoopBackOff issues / Docker containers that exit immediately?

**Answer:**

`CrashLoopBackOff` means the container starts, crashes, and Kubernetes keeps restarting it with exponential backoff (10s → 20s → 40s → 80s → 5min).

**Diagnosis steps:**

```bash
# 1. Get crash details
kubectl describe pod <pod-name>
# Look for: Exit Code, Last State, Reason

# 2. Check current logs
kubectl logs <pod-name>

# 3. Check logs from BEFORE the crash (critical!)
kubectl logs <pod-name> --previous

# 4. Check all containers in pod
kubectl logs <pod-name> --all-containers=true
```

**Common Exit Codes:**

| Exit Code | Meaning | Solution |
|-----------|---------|---------|
| `0` | Success (unexpected exit) | Add `command: ["sleep", "infinity"]` to debug |
| `1` | Application error | Check application logs |
| `137` | OOM Killed (SIGKILL) | Increase memory limits |
| `139` | Segmentation fault | Application bug |
| `143` | SIGTERM — graceful shutdown | Check liveness probe timing |

**Most common root causes:**

1. **Wrong command/entrypoint** — container starts and exits immediately
2. **Missing environment variables** — app crashes because config is missing
3. **Incorrect ConfigMap or Secret** — wrong keys or values
4. **Database not reachable** — app crashes on startup trying to connect
5. **Resource limits too low** — OOMKilled on startup
6. **Liveness probe too aggressive** — kills container before app is ready

**Fix example:**
```yaml
# Add startup probe to give app time to initialize
startupProbe:
  httpGet:
    path: /health
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

---

### Q3. What will you verify if readiness probe fails unexpectedly?

**Answer:**

A failing readiness probe removes the pod from the Service endpoints — traffic stops reaching it but the pod stays running.

```bash
# Check probe config
kubectl describe pod <pod-name> | grep -A 10 "Readiness"

# Check events for probe failures
kubectl describe pod <pod-name> | grep -A 5 "Events"
```

**Checklist:**

| Check | Command |
|-------|---------|
| Is the app actually listening? | `kubectl exec -it <pod> -- curl localhost:8080/health` |
| Is the port correct in probe? | `kubectl get pod <pod> -o yaml | grep readiness` |
| Is the path correct? | Check app routing for `/health` or `/ready` |
| Is the initialDelay long enough? | Increase `initialDelaySeconds` |
| Is the app slow to start? | Add `startupProbe` |

**Fix:**
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 15    # Give app time to start
  periodSeconds: 5
  failureThreshold: 3
  successThreshold: 1
```

---

## 2. Resource Management

### Q4. How would you handle high CPU or memory usage in a cluster?

**Answer:**

**Step 1 — Identify the problem:**
```bash
# Node-level resource usage
kubectl top nodes

# Pod-level resource usage
kubectl top pods --all-namespaces --sort-by=cpu
kubectl top pods --all-namespaces --sort-by=memory
```

**Step 2 — Find resource-hungry pods:**
```bash
kubectl top pods --all-namespaces | sort -k4 -rn | head -20
```

**Step 3 — Check if limits are set:**
```bash
kubectl describe pod <pod> | grep -A 5 "Limits"
kubectl describe pod <pod> | grep -A 5 "Requests"
```

**Step 4 — Fixes:**

| Problem | Fix |
|---------|-----|
| No resource limits set | Add `resources.limits` to all containers |
| Memory leak in app | Restart pod, patch the app, set memory limits |
| CPU throttling | Increase CPU limits or optimize code |
| Single namespace consuming all | Apply ResourceQuota to namespace |
| Node pressure | Scale out cluster (add nodes) |

**Step 5 — Apply ResourceQuota to namespace:**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-quota
  namespace: production
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"
```

**Step 6 — Set LimitRange for defaults:**
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: production
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    type: Container
```

---

### Q5. How would you scale applications during high traffic?

**Answer:**

Kubernetes supports three scaling approaches:

**1. Manual Scaling:**
```bash
kubectl scale deployment my-app --replicas=10
```

**2. Horizontal Pod Autoscaler (HPA):**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
```

**3. Vertical Pod Autoscaler (VPA):**
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
```

**4. Cluster Autoscaler (node-level):**
- Automatically adds/removes nodes when pods are unschedulable
- Configured at cloud provider level (EKS, GKE, AKS)

**HPA not scaling? Check:**
```bash
kubectl describe hpa my-app-hpa
# Common issues:
# - metrics-server not installed
# - resource requests not set on deployment
# - minReplicas already reached maxReplicas
```

---

### Q6. How would you optimize Kubernetes resource utilization?

**Answer:**

**1. Set accurate resource requests and limits:**
```yaml
resources:
  requests:
    cpu: "100m"      # What scheduler uses for placement
    memory: "128Mi"
  limits:
    cpu: "500m"      # Hard cap
    memory: "512Mi"
```

**2. Use Namespace ResourceQuotas** — prevent one team from consuming all cluster resources.

**3. Right-size pods using VPA recommendations:**
```bash
kubectl describe vpa my-app-vpa | grep Recommendation
```

**4. Use Pod Disruption Budgets for safe autoscaling:**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-app
```

**5. Node affinity and bin-packing:**
```yaml
spec:
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - my-app
          topologyKey: kubernetes.io/hostname
```

**6. Use spot/preemptible nodes** for non-critical workloads to reduce cost.

**7. Schedule batch jobs during off-peak hours** using CronJobs.

---

### Q7. What will you check if HPA is not scaling during traffic spikes?

**Answer:**

```bash
# Check HPA status
kubectl describe hpa <hpa-name>
kubectl get hpa

# Common output when broken:
# "unable to get metrics for resource cpu: unable to fetch metrics"
```

**Checklist:**

| Check | Command | Fix |
|-------|---------|-----|
| metrics-server running? | `kubectl get pods -n kube-system | grep metrics` | Install metrics-server |
| Resource requests defined? | `kubectl describe deploy <name>` | Add `resources.requests` |
| HPA conditions | `kubectl describe hpa` → Conditions section | Fix based on condition |
| Current/desired replicas | `kubectl get hpa` | Check min/max bounds |
| CPU threshold hit? | `kubectl top pods` | Adjust target utilization |

```bash
# Install metrics-server if missing
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

## 3. Deployments & Updates

### Q8. How would you manage zero-downtime deployments / troubleshoot failed rolling updates?

**Answer:**

**Zero-downtime rolling update configuration:**
```yaml
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # Extra pods during update
      maxUnavailable: 0    # Never take pods down before new ones are ready
  template:
    spec:
      containers:
      - name: app
        image: myapp:v2
        readinessProbe:         # REQUIRED for zero-downtime
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
```

**Monitor rolling update:**
```bash
kubectl rollout status deployment/my-app
kubectl rollout history deployment/my-app
```

**Troubleshoot stuck rolling update:**
```bash
# Check deployment events
kubectl describe deployment my-app

# Check new pods coming up
kubectl get pods -w

# Check if new pods are failing
kubectl logs <new-pod>
kubectl describe pod <new-pod>
```

**Common issues:**
- New image has a bug → new pods crash → update stalls
- Readiness probe failing → new pods never become Ready
- Resource limits too low → OOMKilled

**Rollback:**
```bash
kubectl rollout undo deployment/my-app
kubectl rollout undo deployment/my-app --to-revision=3
```

---

### Q9. How would you implement blue-green deployment in Kubernetes?

**Answer:**

Blue-green runs two identical production environments — "blue" (current) and "green" (new). Switch traffic by updating the Service selector.

**Setup:**

```yaml
# Blue deployment (current live)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: blue
  template:
    metadata:
      labels:
        app: my-app
        version: blue
    spec:
      containers:
      - name: app
        image: myapp:v1
---
# Green deployment (new version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: green
  template:
    metadata:
      labels:
        app: my-app
        version: green
    spec:
      containers:
      - name: app
        image: myapp:v2
---
# Service — initially points to blue
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
    version: blue     # ← Change this to 'green' to switch traffic
  ports:
  - port: 80
    targetPort: 8080
```

**Switch traffic to green:**
```bash
kubectl patch service my-app-service \
  -p '{"spec":{"selector":{"version":"green"}}}'
```

**Rollback (switch back to blue):**
```bash
kubectl patch service my-app-service \
  -p '{"spec":{"selector":{"version":"blue"}}}'
```

**Blue-green vs Rolling:**

| | Blue-Green | Rolling |
|--|-----------|---------|
| Downtime | Zero (instant switch) | Near-zero |
| Resource cost | Double (2 full envs) | Normal (+maxSurge) |
| Rollback speed | Instant | Slow (rollout undo) |
| Risk | Lower (full test before switch) | Higher |

---

### Q10. What will you investigate if deployment rollout gets stuck midway?

**Answer:**

```bash
# Check rollout status
kubectl rollout status deployment/my-app
# Output: "Waiting for rollout to finish: 1 out of 3 new replicas have been updated..."

# Check what's happening
kubectl describe deployment my-app | tail -20
kubectl get pods | grep my-app
```

**Investigation checklist:**

1. **New pods failing to start:**
   ```bash
   kubectl logs <new-pod-name> --previous
   kubectl describe pod <new-pod-name>
   ```

2. **Readiness probe failing:**
   ```bash
   kubectl describe pod <new-pod> | grep -A 10 Conditions
   ```

3. **Image pull issue:**
   ```bash
   kubectl describe pod <new-pod> | grep -i "image\|pull"
   ```

4. **Resource pressure:**
   ```bash
   kubectl describe nodes | grep -A 5 "Conditions"
   ```

5. **Pause and investigate:**
   ```bash
   kubectl rollout pause deployment/my-app   # Pause the rollout
   # Fix the issue
   kubectl rollout resume deployment/my-app  # Resume
   # OR rollback
   kubectl rollout undo deployment/my-app
   ```

---

### Q11. What will you verify if rollback deployment still does not restore stability?

**Answer:**

If rollback doesn't fix the issue, the problem is likely NOT in the application code.

**Check these in order:**

```bash
# 1. What revision are you on now?
kubectl rollout history deployment/my-app

# 2. Check if older image is actually running
kubectl describe pods | grep "Image:"

# 3. Check ConfigMaps / Secrets (external config changes)
kubectl describe configmap <app-config>
kubectl get secret <app-secret> -o yaml

# 4. Check if database schema changed (migration may be irreversible)
kubectl logs <db-migration-pod>

# 5. Check if infrastructure changed (new node, network policy)
kubectl get nodes
kubectl get networkpolicies

# 6. Check persistent volume data corruption
kubectl describe pvc
kubectl exec -it <pod> -- ls /data
```

**Common non-code issues:**
- Database migration ran and changed schema incompatibly
- Secret or ConfigMap was updated externally
- A network policy was added blocking traffic
- A PersistentVolume has corrupt data

---

## 4. Node Management

### Q12. How would you handle node failure in production?

**Answer:**

**Immediate response:**

```bash
# 1. Identify failed node
kubectl get nodes
# STATUS: NotReady

# 2. Get details
kubectl describe node <node-name>

# 3. Check what pods were on the failed node
kubectl get pods --all-namespaces -o wide | grep <node-name>
```

**How Kubernetes auto-heals:**
- After 5 minutes (default `pod-eviction-timeout`), pods are evicted from NotReady node
- Pods with Deployments/ReplicaSets are automatically rescheduled to healthy nodes
- DaemonSet pods are NOT rescheduled (they're per-node by design)

**Manual eviction (speed up recovery):**
```bash
# Cordon — prevent new pods from scheduling
kubectl cordon <failed-node>

# Drain — move existing pods to other nodes
kubectl drain <failed-node> \
  --ignore-daemonsets \
  --delete-emptydir-data \
  --force
```

**After node repair:**
```bash
kubectl uncordon <node-name>   # Allow scheduling again
```

**Prevention:**
- Use Pod Disruption Budgets to ensure minimum replicas
- Deploy across multiple availability zones
- Set `podAntiAffinity` to spread pods across nodes

---

### Q13. What will you investigate if node becomes NotReady intermittently?

**Answer:**

```bash
# Check node conditions
kubectl describe node <node-name>
# Look for: MemoryPressure, DiskPressure, PIDPressure, NetworkUnavailable

# Check node events
kubectl get events --field-selector involvedObject.name=<node-name>
```

**Troubleshooting by condition:**

| Condition | Cause | Fix |
|-----------|-------|-----|
| `MemoryPressure` | Node running out of memory | Evict pods, reduce memory limits, add node |
| `DiskPressure` | Disk full (logs, images) | Clean images `docker system prune`, expand disk |
| `PIDPressure` | Too many processes | Find process leak, set PID limits |
| `NetworkUnavailable` | CNI issue | Restart CNI plugin, check overlay network |
| `kubelet not reporting` | kubelet crash | SSH into node, `systemctl restart kubelet` |

**SSH and check kubelet:**
```bash
ssh <node-ip>
systemctl status kubelet
journalctl -u kubelet -n 100
```

---

### Q14. How would you reduce downtime during cluster upgrades?

**Answer:**

**Pre-upgrade checklist:**
```bash
# Check current version
kubectl version
kubectl get nodes

# Review changelog for breaking changes
# Test in staging first

# Backup etcd
ETCDCTL_API=3 etcdctl snapshot save backup.db
```

**Upgrade strategy (one node at a time):**

```bash
# Step 1: Cordon the node (no new pods)
kubectl cordon <node-name>

# Step 2: Drain the node (move pods away)
kubectl drain <node-name> \
  --ignore-daemonsets \
  --delete-emptydir-data

# Step 3: Upgrade node (kubeadm example)
sudo kubeadm upgrade node
sudo systemctl restart kubelet

# Step 4: Uncordon node
kubectl uncordon <node-name>

# Step 5: Verify
kubectl get nodes
# Repeat for each node
```

**For managed K8s (EKS/GKE/AKS):**
- Use rolling node pool upgrades
- Enable PodDisruptionBudgets before upgrade
- Use maintenance windows during low-traffic periods

**Key protection during upgrades:**
```yaml
# PodDisruptionBudget ensures app stays available
apiVersion: policy/v1
kind: PodDisruptionBudget
spec:
  minAvailable: 2    # At least 2 pods always running
  selector:
    matchLabels:
      app: my-critical-app
```

---

## 5. Networking

### Q9. How would you troubleshoot networking issues between Pods / Docker containers?

**Answer:**

**Step 1 — Verify pod-to-pod connectivity:**
```bash
# Get pod IPs
kubectl get pods -o wide

# Test connectivity from one pod to another
kubectl exec -it <pod-a> -- curl http://<pod-b-ip>:8080
kubectl exec -it <pod-a> -- ping <pod-b-ip>
```

**Step 2 — Test via Service (DNS):**
```bash
kubectl exec -it <pod> -- nslookup my-service
kubectl exec -it <pod> -- curl http://my-service:8080
kubectl exec -it <pod> -- curl http://my-service.namespace.svc.cluster.local:8080
```

**Step 3 — Check NetworkPolicy:**
```bash
kubectl get networkpolicies -n <namespace>
kubectl describe networkpolicy <policy-name>
# NetworkPolicies can BLOCK traffic between pods
```

**Step 4 — Check Service and Endpoints:**
```bash
kubectl get service my-service
kubectl get endpoints my-service
# If Endpoints is empty → selector doesn't match any pods!

# Check selector matches
kubectl get pods --show-labels
kubectl describe service my-service | grep "Selector"
```

**Step 5 — CoreDNS check:**
```bash
kubectl get pods -n kube-system | grep coredns
kubectl logs -n kube-system <coredns-pod>

# Test DNS from pod
kubectl exec -it <pod> -- nslookup kubernetes.default
```

**Common fixes:**

| Problem | Fix |
|---------|-----|
| Service selector mismatch | Fix labels on pods or selector on service |
| NetworkPolicy blocking | Add appropriate ingress/egress rules |
| CoreDNS crash | `kubectl rollout restart deployment/coredns -n kube-system` |
| CNI plugin issue | Restart CNI daemonset |

---

### Q15. What will you investigate if CoreDNS fails intermittently?

**Answer:**

```bash
# Check CoreDNS pods health
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system <coredns-pod-1>
kubectl logs -n kube-system <coredns-pod-2>

# Test DNS resolution from pod
kubectl run dns-test --image=busybox --rm -it -- nslookup kubernetes.default

# Check CoreDNS ConfigMap
kubectl describe configmap coredns -n kube-system
```

**Common causes:**

| Cause | Fix |
|-------|-----|
| Memory pressure on CoreDNS pods | Increase memory limits for CoreDNS |
| High DNS query load | Scale CoreDNS replicas |
| ndots misconfiguration | Reduce `ndots:5` to `ndots:2` in pod spec |
| Upstream DNS unreachable | Check `forward` section in CoreDNS config |
| Nodes' `/etc/resolv.conf` issue | Check node DNS config |

**Fix CoreDNS scaling:**
```bash
kubectl scale deployment coredns -n kube-system --replicas=4
```

---

### Q16. What will you verify if overlay network causes communication issues?

**Answer:**

Overlay networks (Flannel, Calico, Weave, Cilium) create virtual networks across nodes.

```bash
# Check CNI pods are running
kubectl get pods -n kube-system | grep -E "flannel|calico|weave|cilium"

# Check CNI logs
kubectl logs -n kube-system <cni-pod>

# Test cross-node connectivity
# Pod on Node1 → Pod on Node2
kubectl exec -it <pod-on-node1> -- ping <pod-ip-on-node2>
```

**Diagnosis:**
```bash
# Check if MTU mismatch
kubectl exec -it <pod> -- ip link show eth0 | grep mtu

# Check routing table on node
ssh <node>
ip route show

# Check if firewall blocking overlay ports
# Flannel: UDP 8285, 8472
# Calico: TCP 179 (BGP), IP-IP protocol
# Weave: TCP/UDP 6783, 6784
```

**Common fixes:**
- Restart CNI daemonset: `kubectl rollout restart daemonset <cni-name> -n kube-system`
- Check cloud provider security groups allow overlay traffic
- Verify MTU settings match between node and pod network

---

### Q17. How would you troubleshoot Ingress controller issues / ingress stops routing traffic?

**Answer:**

```bash
# Check ingress resource
kubectl describe ingress <ingress-name>

# Check ingress controller pods
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx <ingress-controller-pod>

# Check service and endpoints the ingress points to
kubectl get service <backend-service>
kubectl get endpoints <backend-service>
```

**Troubleshooting flow:**

```
Browser → Ingress Controller → Service → Pod

Check each hop:
1. Is the ingress controller pod running? → kubectl get pods -n ingress-nginx
2. Is the LoadBalancer/NodePort service for ingress working? → kubectl get svc -n ingress-nginx
3. Does the Ingress resource have the right host/path? → kubectl describe ingress
4. Does the backend Service selector match pods? → kubectl get endpoints
5. Are the backend pods healthy? → kubectl get pods
```

**Common issues:**

| Issue | Cause | Fix |
|-------|-------|-----|
| 404 Not Found | Wrong path in Ingress | Check `path` and `pathType` |
| 502 Bad Gateway | Backend pod crashing | Fix backend pod |
| 503 Service Unavailable | No healthy endpoints | Check pod readiness |
| SSL errors | Certificate issue | Check TLS secret |
| No address | LoadBalancer pending | Check cloud LB provisioning |

**Check TLS:**
```bash
kubectl get secret <tls-secret-name> -o yaml
kubectl describe certificate <cert-name>   # if using cert-manager
```

---

### Q18. What will you verify if service works internally but fails externally?

**Answer:**

"Works internally" = pod-to-pod works. "Fails externally" = outside cluster can't reach it.

```bash
# Step 1: Verify internal connectivity
kubectl exec -it <pod> -- curl http://my-service:8080    # should work

# Step 2: Check external service type
kubectl get service my-service
# TYPE: ClusterIP → only internal, need NodePort or LoadBalancer for external
```

**Service type matrix:**

| Type | Access | Use Case |
|------|--------|---------|
| `ClusterIP` | Internal only | Pod-to-pod |
| `NodePort` | NodeIP:Port | Dev, on-prem |
| `LoadBalancer` | Cloud LB IP | Production |
| `Ingress` | Domain/path routing | HTTP/HTTPS apps |

**External access checklist:**

```bash
# Check if NodePort is accessible
curl http://<node-ip>:<node-port>

# Check firewall / security groups
# AWS: EC2 Security Group inbound rules
# GCP: VPC Firewall rules
# On-prem: iptables, cloud firewall

# Check LoadBalancer external IP assigned
kubectl get svc my-service
# EXTERNAL-IP: <pending> means cloud LB not provisioned yet

# Check Ingress external IP
kubectl get ingress
# ADDRESS: should have an IP

# DNS resolution
nslookup my-app.example.com
```

---

## 6. Storage

### Q19. How would you handle persistent storage failures / PVC stuck in Pending state?

**Answer:**

**PVC stuck in Pending — diagnosis:**
```bash
kubectl describe pvc <pvc-name>
# Look for Events section
```

**Common PVC Pending causes:**

| Cause | Event message | Fix |
|-------|--------------|-----|
| No matching StorageClass | `no volume plugin matched` | Check `storageClassName` |
| Storage class doesn't exist | `storageclass not found` | Create StorageClass |
| Node capacity | `no nodes available` | Add storage or use different zone |
| Access mode mismatch | `incompatible access mode` | Check `accessModes` |
| Dynamic provisioner down | `failed to provision volume` | Check provisioner pod |

```bash
# Check available StorageClasses
kubectl get storageclass

# Check if dynamic provisioner is running
kubectl get pods -n kube-system | grep provisioner
kubectl logs -n kube-system <provisioner-pod>

# Check PV availability
kubectl get pv
```

**Fix PVC with correct StorageClass:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard    # must match existing StorageClass
  resources:
    requests:
      storage: 10Gi
```

**Storage failure in running pod:**
```bash
# Check if volume is mounted
kubectl exec -it <pod> -- df -h
kubectl exec -it <pod> -- ls /data

# Check node-level mount issues
ssh <node>
df -h
mount | grep <pv-name>

# Check NFS/EBS/CSI driver logs
kubectl logs -n kube-system <csi-driver-pod>
```

---

## 7. Security & RBAC

### Q20. How would you secure Kubernetes clusters in production?

**Answer:**

**Security layers:**

```
1. API Server → Authentication + Authorization
2. RBAC → Who can do what
3. Network Policies → Who can talk to whom
4. Pod Security → What containers can do
5. Secrets → How sensitive data is stored
6. Image Security → What images can run
```

**1. Enable RBAC (should be on by default):**
```bash
kubectl api-versions | grep rbac
```

**2. Use Network Policies:**
```yaml
# Default deny all ingress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

**3. Use Pod Security Admission:**
```yaml
# Namespace-level pod security
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
```

**4. Scan images:**
```bash
# Use Trivy for image scanning
trivy image myapp:latest
```

**5. Disable automounting service account tokens:**
```yaml
spec:
  automountServiceAccountToken: false
```

**6. Use read-only root filesystem:**
```yaml
securityContext:
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 1000
```

---

### Q21. How would you manage RBAC permissions in Kubernetes?

**Answer:**

RBAC controls: **Who (Subject) can do What (Verb) on Which (Resource)**

**Key objects:**

| Object | Scope | Description |
|--------|-------|-------------|
| Role | Namespace | Permissions within one namespace |
| ClusterRole | Cluster-wide | Permissions across all namespaces |
| RoleBinding | Namespace | Bind Role to user/group/SA |
| ClusterRoleBinding | Cluster-wide | Bind ClusterRole to user/group/SA |

**Create a read-only Role:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: production
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
subjects:
- kind: User
  name: developer@company.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**Check what a user can do:**
```bash
kubectl auth can-i create pods --as=developer@company.com
kubectl auth can-i delete deployments --as=developer@company.com -n production
kubectl auth can-i '*' '*' --as=admin@company.com   # admin check
```

**List permissions for current user:**
```bash
kubectl auth can-i --list
kubectl auth can-i --list -n production
```

---

### Q22. How would you manage secrets securely in Kubernetes?

**Answer:**

**Create secrets properly:**
```bash
# From literal (avoid — visible in shell history)
kubectl create secret generic db-secret \
  --from-literal=password=mypassword

# From file (better)
echo -n "mypassword" > password.txt
kubectl create secret generic db-secret \
  --from-file=password=password.txt
rm password.txt
```

**Use secrets in pods:**
```yaml
# As environment variable
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password

# As mounted file (more secure)
volumes:
- name: secret-volume
  secret:
    secretName: db-secret
volumeMounts:
- name: secret-volume
  mountPath: /etc/secrets
  readOnly: true
```

**Production secret management:**

| Tool | Use case |
|------|---------|
| HashiCorp Vault | Enterprise secret management |
| AWS Secrets Manager | AWS-native secrets |
| Sealed Secrets | GitOps-safe encrypted secrets |
| External Secrets Operator | Sync from external providers |

**Encrypt secrets at rest (etcd):**
```yaml
# /etc/kubernetes/encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  providers:
  - aescbc:
      keys:
      - name: key1
        secret: <base64-encoded-key>
  - identity: {}
```

---

## 8. Monitoring & Observability

### Q23. How would you monitor Kubernetes clusters effectively?

**Answer:**

**Monitoring stack (production standard):**

```
Metrics:  Prometheus + Grafana
Logs:     EFK (Elasticsearch + Fluentd + Kibana) OR Loki + Grafana
Tracing:  Jaeger OR Zipkin
Alerts:   Alertmanager
```

**Install kube-prometheus-stack:**
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

**Key metrics to monitor:**

| Category | Metrics |
|----------|---------|
| Node | CPU%, Memory%, Disk%, Network I/O |
| Pod | CPU throttling, Memory usage, Restart count |
| Cluster | API server latency, etcd health |
| App | Request rate, Error rate, Latency (RED metrics) |

**Critical alerts to set up:**
```yaml
# Example Prometheus alert
groups:
- name: kubernetes
  rules:
  - alert: PodCrashLooping
    expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Pod {{ $labels.pod }} is crash looping"

  - alert: NodeMemoryPressure
    expr: kube_node_status_condition{condition="MemoryPressure",status="true"} == 1
    for: 2m
    labels:
      severity: warning
```

**Check metrics manually:**
```bash
kubectl top nodes
kubectl top pods --all-namespaces
kubectl get --raw /metrics | grep apiserver_request
```

---

### Q24. What will you check if ConfigMap changes are not reflected inside pods?

**Answer:**

Kubernetes does NOT automatically restart pods when a ConfigMap changes.

**How pods consume ConfigMaps:**

| Method | Auto-update? | Restart needed? |
|--------|-------------|----------------|
| Environment variable | ❌ Never | Yes — delete/recreate pod |
| Volume mount | ✅ Eventually (sync period ~1min) | No (for volume mounts) |

```bash
# Check if ConfigMap was actually updated
kubectl describe configmap <cm-name>
kubectl get configmap <cm-name> -o yaml

# For env vars — pods must be restarted
kubectl rollout restart deployment/<deployment-name>

# Verify inside pod
kubectl exec -it <pod> -- env | grep MY_CONFIG_KEY   # env var
kubectl exec -it <pod> -- cat /etc/config/my-key     # volume mount
```

**For automatic restart on ConfigMap change (use Reloader):**
```bash
helm install reloader stakater/reloader

# Annotate deployment to auto-restart
kubectl annotate deployment my-app \
  configmap.reloader.stakater.com/reload="my-configmap"
```

---

## 9. Disaster Recovery

### Q25. How would you implement disaster recovery in Kubernetes?

**Answer:**

**DR strategy layers:**

```
1. etcd backup (cluster state)
2. PersistentVolume backup (data)
3. Multi-region deployment (availability)
4. Application-level backup (DB dumps)
```

**1. etcd backup:**
```bash
ETCDCTL_API=3 etcdctl snapshot save \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/server.crt \
  --key=/etc/etcd/server.key \
  /backup/etcd-$(date +%Y%m%d).db

# Verify backup
etcdctl snapshot status /backup/etcd-20260522.db
```

**2. Use Velero for full cluster backup:**
```bash
# Install Velero
velero install \
  --provider aws \
  --bucket my-backup-bucket \
  --backup-location-config region=us-east-1

# Create backup
velero backup create daily-backup --include-namespaces production

# Schedule daily backup
velero schedule create daily \
  --schedule="0 2 * * *" \
  --include-namespaces production

# Restore
velero restore create --from-backup daily-backup
```

**3. Multi-AZ deployment:**
```yaml
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: critical-app
        topologyKey: topology.kubernetes.io/zone
```

**RTO/RPO targets:**

| Tier | RPO | RTO | Strategy |
|------|-----|-----|---------|
| Critical | < 1 min | < 5 min | Active-active multi-region |
| Important | < 1 hour | < 30 min | Active-passive + Velero |
| Normal | < 24 hours | < 2 hours | Daily backup + restore |

---

### Q26. How would you manage multi-environment deployments in Kubernetes?

**Answer:**

**Option 1: Namespaces (simple):**
```bash
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace production
```

**Option 2: Helm values per environment:**
```bash
# Deploy to dev
helm install my-app ./chart -f values-dev.yaml -n dev

# Deploy to staging
helm install my-app ./chart -f values-staging.yaml -n staging

# Deploy to production
helm install my-app ./chart -f values-prod.yaml -n production
```

**Option 3: Kustomize overlays:**
```
base/
  deployment.yaml
  service.yaml
overlays/
  dev/
    kustomization.yaml    # replicas: 1, image: myapp:dev
  staging/
    kustomization.yaml    # replicas: 2, image: myapp:staging
  production/
    kustomization.yaml    # replicas: 10, image: myapp:v1.5.0
```

**Option 4: GitOps with ArgoCD:**
```yaml
# One Application per environment
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-production
spec:
  source:
    repoURL: https://github.com/org/repo
    targetRevision: main
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## 10. Docker Specific

### Q27. What will you investigate if Docker image works locally but fails in Kubernetes?

**Answer:**

This is one of the most common real-world issues. The key differences:

| | Local Docker | Kubernetes |
|--|-------------|-----------|
| User | Root (default) | Non-root (enforced) |
| Filesystem | Writable | Read-only possible |
| Env vars | From `.env` file | From ConfigMap/Secret |
| Networking | Bridge network | Pod network |
| Storage | Local paths | PVCs |
| Image | Local registry | Remote registry |

**Checklist:**

```bash
# 1. Check if image is accessible from cluster
kubectl run test --image=myapp:v1 --restart=Never
kubectl describe pod test

# 2. Check if running as root causes issues
kubectl exec -it <pod> -- whoami
# If "root" and K8s has runAsNonRoot: true → will fail

# 3. Check environment variables
kubectl exec -it <pod> -- env
# Compare with local docker run env

# 4. Check file permissions
kubectl exec -it <pod> -- ls -la /app

# 5. Check if app writes to filesystem (not allowed if readOnly)
kubectl exec -it <pod> -- touch /app/test
# Permission denied → app needs writable path via volumeMount

# 6. Check security context
kubectl describe pod <pod> | grep -A 10 "Security Context"
```

**Fix — add security context:**
```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000
  readOnlyRootFilesystem: false   # set true only if app supports it
```

---

### Q28. How would you handle image pull errors in production / ImagePullBackOff?

**Answer:**

`ImagePullBackOff` = Kubernetes tried to pull the image, failed, and will retry with backoff.

**Diagnosis:**
```bash
kubectl describe pod <pod-name>
# Look at Events section:
# Failed to pull image "myapp:v2": rpc error: ...
```

**Common causes:**

| Error message | Cause | Fix |
|---------------|-------|-----|
| `not found` | Wrong image name or tag | Fix image name/tag |
| `unauthorized` | Registry auth failed | Add/fix imagePullSecret |
| `connection refused` | Private registry unreachable | Check registry URL |
| `toomanyrequests` | Docker Hub rate limit | Use authenticated pull or private registry |

**Fix private registry auth:**
```bash
# Create docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.company.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=me@company.com

# Reference in pod spec
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: app
    image: registry.company.com/myapp:v2
```

**Fix Docker Hub rate limit:**
```bash
# Always use full image tag (never 'latest' in prod)
image: nginx:1.25.3   # not nginx:latest

# Use authenticated pulls
kubectl create secret docker-registry dockerhub-cred \
  --docker-username=<username> \
  --docker-password=<token>
```

---

## 11. Scenario-Based Questions

### Scenario 1: Production pods are OOMKilled every 2 hours

**Problem:** Your app pods crash with `OOMKilled` every 2 hours at exactly 14:00.

**Answer:**
```bash
# 1. Check memory limits
kubectl describe pod <pod> | grep -A 5 "Limits"
kubectl top pod <pod>

# 2. Check if there's a scheduled job at 14:00
kubectl get cronjobs
# A batch job running at 14:00 consuming memory?

# 3. Check application memory trend
# Prometheus query: container_memory_usage_bytes{pod="my-pod"}

# 4. Fix: Increase memory limits
kubectl patch deployment my-app --patch '
{"spec":{"template":{"spec":{"containers":[{
  "name":"app",
  "resources":{"limits":{"memory":"1Gi"}}
}]}}}}'

# 5. Fix memory leak in application
# Profile with: kubectl port-forward <pod> 8080 and hit /debug/pprof
```

---

### Scenario 2: 100% of requests failing — all pods running

**Problem:** `kubectl get pods` shows all Running, but users report 100% error rate.

**Answer:**
```bash
# 1. Check if pods are actually ready
kubectl get pods    # check READY column: 1/1 or 0/1?

# 2. Check service endpoints
kubectl get endpoints my-service
# Empty endpoints = no pods match selector

# 3. Check if readiness probe is failing
kubectl describe pod <pod> | grep -A 5 "Conditions"

# 4. Check actual app response
kubectl exec -it <pod> -- curl localhost:8080/health

# 5. Check application logs
kubectl logs <pod> --tail=50

# 6. Check recent deployments
kubectl rollout history deployment/my-app
kubectl rollout undo deployment/my-app  # if new deployment caused it
```

---

### Scenario 3: Kubernetes cluster ran out of CPU — how to recover fast

**Problem:** All new pods stuck in `Pending`. `kubectl describe pod` shows `Insufficient cpu`.

**Answer:**
```bash
# 1. Find what's consuming CPU
kubectl top nodes
kubectl top pods --all-namespaces | sort -k3 -rn | head -20

# 2. Find pods without proper resource requests (over-scheduling risk)
kubectl get pods --all-namespaces -o json | \
  jq '.items[] | select(.spec.containers[].resources.requests == null) | .metadata.name'

# 3. Immediate relief: scale down non-critical deployments
kubectl scale deployment dev-app --replicas=0 -n development

# 4. Add nodes (cloud)
# AWS: modify node group desired capacity
# GCP: resize node pool

# 5. Evict low-priority pods using PriorityClasses
kubectl get priorityclasses

# 6. Long-term: Set resource requests on ALL deployments
```

---

### Scenario 4: Secret was accidentally exposed in pod logs

**Problem:** A developer's code printed `os.environ['DB_PASSWORD']` to logs.

**Answer:**
```bash
# 1. Immediately rotate the exposed credential
# Change DB password in database
# Update Kubernetes secret
kubectl create secret generic db-secret \
  --from-literal=password=newpassword \
  --dry-run=client -o yaml | kubectl apply -f -

# 2. Restart all pods using the secret
kubectl rollout restart deployment/my-app

# 3. Clear logs (if using log aggregation)
# Delete and recreate log stream in your SIEM

# 4. Audit who accessed the secret
kubectl get events | grep secret
kubectl auth can-i get secret db-secret --as=developer@company.com

# 5. Fix the code — never log secrets
# Use structured logging that redacts sensitive fields

# 6. Add OPA/Gatekeeper policy to prevent
# containers with LOG_LEVEL=DEBUG in production
```

---

### Scenario 5: Deployment stuck — minReadySeconds misconfigured

**Problem:** Rolling update is taking 10+ minutes. `maxUnavailable: 0` and `maxSurge: 1`.

**Answer:**
```bash
# Check deployment config
kubectl describe deployment my-app | grep -E "minReadySeconds|Strategy"

# If minReadySeconds is set too high (e.g., 300 = 5 min per pod)
kubectl patch deployment my-app --patch \
  '{"spec":{"minReadySeconds":10}}'

# Resume if paused
kubectl rollout resume deployment/my-app

# Monitor
kubectl rollout status deployment/my-app -w
```

---

### Scenario 6: New developer pushed to production by mistake

**Problem:** Developer ran `kubectl apply` against prod cluster instead of dev.

**Answer:**
```bash
# 1. Check what was changed
kubectl rollout history deployment/my-app -n production

# 2. Rollback immediately
kubectl rollout undo deployment/my-app -n production

# 3. Verify rollback
kubectl rollout status deployment/my-app -n production

# 4. Prevention going forward:
# a) Use separate kubeconfig per environment
# b) Add RBAC: developers can't write to production namespace
kubectl create rolebinding dev-read-only \
  --clusterrole=view \
  --user=developer@company.com \
  --namespace=production

# c) Add kubectl alias that shows current context
# alias k='kubectl --context=$(kubectl config current-context)'
```

---

### Scenario 7: etcd is down — control plane not responding

**Problem:** `kubectl get pods` returns "connection refused". etcd pod is in CrashLoopBackOff.

**Answer:**
```bash
# 1. SSH into control plane node
ssh <master-node>

# 2. Check etcd status
systemctl status etcd
journalctl -u etcd -n 100

# 3. Check etcd data directory
ls -la /var/lib/etcd/

# 4. Try restoring from snapshot
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-latest.db \
  --data-dir=/var/lib/etcd-restore

# 5. Point etcd to restored data
mv /var/lib/etcd /var/lib/etcd.old
mv /var/lib/etcd-restore /var/lib/etcd
systemctl start etcd

# 6. Verify
ETCDCTL_API=3 etcdctl member list
kubectl get nodes   # should work now
```

---

### Scenario 8: Service mesh (Istio) sidecar causing pod to fail

**Problem:** After installing Istio, all pods in production are failing with network errors.

**Answer:**
```bash
# 1. Check sidecar injection
kubectl describe pod <pod> | grep istio

# 2. Check Envoy proxy logs
kubectl logs <pod> -c istio-proxy

# 3. Check if mTLS is blocking
kubectl describe peerauthentication

# 4. Temporarily disable injection for debugging
kubectl label namespace production istio-injection=disabled
kubectl rollout restart deployment/my-app -n production

# 5. Check DestinationRule mTLS settings
kubectl get destinationrule -A

# 6. Common fix: add exception for health check
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-app
spec:
  host: my-app
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
EOF
```

---

### Scenario 9: Database pod losing data on restart

**Problem:** MySQL pod restarts and all data is gone every time.

**Answer:**
```bash
# Root cause: No PersistentVolumeClaim — data stored in container filesystem
kubectl describe pod mysql | grep Volumes

# Fix: Add PVC for MySQL
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: standard
---
# Update deployment to mount PVC
# Add under spec.template.spec:
volumes:
- name: mysql-storage
  persistentVolumeClaim:
    claimName: mysql-pvc
# Add under container:
volumeMounts:
- name: mysql-storage
  mountPath: /var/lib/mysql
EOF
```

---

### Scenario 10: HPA scaling up but pods are Pending — no nodes available

**Problem:** HPA triggered scale-up from 5 to 20 pods. New pods stuck in `Pending`.

**Answer:**
```bash
# 1. Check pending pods
kubectl describe pod <pending-pod> | grep -A 5 Events
# "0/3 nodes are available: 3 Insufficient cpu"

# 2. Check node capacity
kubectl describe nodes | grep -A 5 "Allocated resources"

# 3. Check if Cluster Autoscaler is running
kubectl get pods -n kube-system | grep cluster-autoscaler
kubectl logs -n kube-system <cluster-autoscaler-pod> | tail -20

# 4. Check autoscaler activity
kubectl get events -n kube-system | grep ScaleUp

# 5. If using cloud — check autoscaling group limits
# AWS: Check ASG max size
# GCP: Check node pool max nodes

# 6. Emergency: reduce HPA max or lower resource requests
kubectl patch hpa my-app-hpa --patch '{"spec":{"maxReplicas":10}}'
```

---

### Scenario 11: Certificates expired — API server unreachable

**Problem:** Morning of day: kubectl returns "certificate has expired".

**Answer:**
```bash
# 1. Check certificate expiry
kubeadm certs check-expiration

# 2. Renew all certificates
kubeadm certs renew all

# 3. Restart control plane components
# Static pods (kubeadm): move and restore manifest to trigger restart
mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
sleep 5
mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/

# 4. Update kubeconfig
cp /etc/kubernetes/admin.conf ~/.kube/config

# 5. Verify
kubectl get nodes
kubeadm certs check-expiration

# Prevention: Set up monthly cert renewal cron
0 0 1 * * kubeadm certs renew all && systemctl restart kubelet
```

---

### Scenario 12: Traffic spike — application latency increased 10x

**Problem:** Black Friday. App usually handles 100 RPS, now 1000 RPS. P99 latency went from 100ms to 1000ms.

**Answer:**
```bash
# 1. Check current pod count and CPU
kubectl top pods -n production
kubectl get hpa

# 2. Check if HPA is scaling
kubectl describe hpa my-app-hpa
# If maxReplicas reached → temporarily increase
kubectl patch hpa my-app-hpa --patch '{"spec":{"maxReplicas":50}}'

# 3. Check DB connection pool exhaustion
kubectl logs <pod> | grep -i "connection pool\|timeout"

# 4. Enable rate limiting at ingress
kubectl annotate ingress my-ingress \
  nginx.ingress.kubernetes.io/limit-rps="200"

# 5. Check if static assets cached
kubectl annotate ingress my-ingress \
  nginx.ingress.kubernetes.io/proxy-buffering="on"

# 6. Preemptively scale next time
kubectl scale deployment my-app --replicas=20  # pre-scale before event
```

---

### Scenario 13: Namespace deleted accidentally — recover resources

**Problem:** Someone ran `kubectl delete namespace production` — 20 deployments gone.

**Answer:**
```bash
# 1. Check if namespace is actually deleted or just terminating
kubectl get namespace production

# If Terminating — it can be recovered:
kubectl get namespace production -o json | \
  jq '.spec.finalizers = []' | \
  kubectl replace --raw "/api/v1/namespaces/production/finalize" -f -

# 2. If fully deleted — restore from backup
velero restore create --from-backup daily-backup \
  --include-namespaces production

# 3. If no backup — check git repo for manifests
git log --all --oneline k8s/production/
kubectl apply -f k8s/production/ -R

# 4. Recreate manually from history
kubectl rollout history deployment/my-app   # may not work if namespace gone
# Check GitOps repo or Helm release history

# Prevention:
# Add namespace deletion protection
kubectl annotate namespace production \
  "hnc.x-k8s.io/managed-by=true"
```

---

### Scenario 14: Multi-container pod — one sidecar crashing affects main

**Problem:** A logging sidecar keeps crashing. The main app is fine but the pod restarts.

**Answer:**
```bash
# 1. Identify which container is crashing
kubectl get pods -o json | jq '.items[].status.containerStatuses[] | {name, restartCount, state}'

# 2. Check sidecar logs
kubectl logs <pod> -c logging-sidecar
kubectl logs <pod> -c logging-sidecar --previous

# 3. Check sidecar resource usage
kubectl top pod <pod> --containers

# 4. Fix sidecar without affecting main container
kubectl set image deployment/my-app logging-sidecar=fluent/fluent-bit:2.1

# 5. If sidecar can be made non-critical (K8s 1.29+)
# Use sidecar containers with restartPolicy: Always
# Main container won't restart if sidecar crashes

# 6. Temporary fix: remove sidecar from prod
kubectl patch deployment my-app --type=json \
  -p='[{"op":"remove","path":"/spec/template/spec/initContainers/0"}]'
```

---

### Scenario 15: Pod scheduled on wrong node — violating compliance

**Problem:** A pod containing sensitive EU customer data is running on a US-region node, violating GDPR.

**Answer:**
```bash
# 1. Check current node
kubectl get pod <pod> -o wide

# 2. Label EU nodes
kubectl label node eu-node-1 region=eu
kubectl label node eu-node-2 region=eu

# 3. Add nodeSelector to deployment
kubectl patch deployment my-app --patch '
{"spec":{"template":{"spec":{
  "nodeSelector":{"region":"eu"}
}}}}'

# 4. Verify new pods land on EU nodes
kubectl get pods -o wide | grep my-app

# 5. Evict old pod on US node
kubectl delete pod <old-pod-on-us-node>
# New pod will be created on EU node

# 6. Stricter enforcement: use Node Affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: region
            operator: In
            values:
            - eu
```

---

### Scenario 16: Cluster upgrade broke CRDs (Custom Resource Definitions)

**Problem:** After upgrading K8s from 1.25 to 1.28, CustomResourceDefinitions stopped working.

**Answer:**
```bash
# 1. Check CRD status
kubectl get crds
kubectl describe crd <crd-name>

# 2. Check if API version was deprecated
# K8s 1.25 removed: networking.k8s.io/v1beta1
# K8s 1.26 removed: autoscaling/v2beta2

# 3. Check what's broken
kubectl api-versions | grep <group>

# 4. Migrate CRD apiVersion
kubectl convert -f old-crd.yaml --output-version new-version

# 5. Update operator/controller that manages CRDs
helm upgrade my-operator my-repo/my-operator --version 2.0.0

# 6. Check Helm releases for deprecated API usage
helm list -A
helm upgrade --reset-values my-release my-chart

# Prevention: Run pluto before upgrade
pluto detect-all-in-cluster  # finds deprecated API usage
```

---

### Scenario 17: Production app running as root — security audit finding

**Problem:** Security scan found all containers running as root (UID 0).

**Answer:**
```bash
# 1. Verify current state
kubectl get pods -o json | \
  jq '.items[].spec.containers[].securityContext.runAsUser'

# 2. Check if app works as non-root (test first)
kubectl run test --image=myapp:latest --restart=Never \
  --overrides='{"spec":{"securityContext":{"runAsUser":1000}}}'

# 3. Update deployment with security context
kubectl patch deployment my-app --patch '
{"spec":{"template":{"spec":{
  "securityContext":{
    "runAsUser":1000,
    "runAsGroup":1000,
    "fsGroup":1000,
    "runAsNonRoot":true
  }
}}}}'

# 4. Fix Dockerfile if app requires root at build time
# FROM myapp:latest
# RUN chown -R 1000:1000 /app
# USER 1000

# 5. Enforce with Pod Security Admission
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted
```

---

### Scenario 18: Deployment has no resource limits — node crashed

**Problem:** A new developer deployed an app with no resource limits. The node OOMKilled and crashed.

**Answer:**
```bash
# 1. Identify deployment with no limits
kubectl get pods --all-namespaces -o json | \
  jq -r '.items[] | select(.spec.containers[].resources == {}) | 
  "\(.metadata.namespace)/\(.metadata.name)"'

# 2. Immediately add LimitRange to namespace (applies defaults)
kubectl apply -f - <<EOF
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: production
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    type: Container
EOF

# 3. Fix the specific deployment
kubectl set resources deployment/runaway-app \
  --limits=cpu=500m,memory=512Mi \
  --requests=cpu=100m,memory=128Mi

# 4. Add OPA/Gatekeeper policy to reject pods without limits
# Ensure CI/CD pipeline validates resource limits before deploy

# 5. Recover crashed node
kubectl uncordon <crashed-node>
```

---

### Scenario 19: DNS resolution slow — app latency increased

**Problem:** All app calls to internal services suddenly became 5x slower.

**Answer:**
```bash
# 1. Test DNS resolution time
kubectl run dns-perf --image=busybox --rm -it -- \
  sh -c "time nslookup my-service.production.svc.cluster.local"

# 2. Check CoreDNS performance
kubectl top pod -n kube-system -l k8s-app=kube-dns

# 3. Check DNS queries (if high load)
kubectl logs -n kube-system <coredns-pod> | grep -c "QUERY"

# 4. Scale CoreDNS
kubectl scale deployment coredns -n kube-system --replicas=4

# 5. Reduce DNS lookups in application
# Set ndots: 2 (prevents unnecessary lookups)
spec:
  dnsConfig:
    options:
    - name: ndots
      value: "2"
    - name: single-request-reopen

# 6. Use NodeLocal DNSCache
# Runs a DNS cache on each node — major performance improvement
kubectl apply -f https://k8s.io/examples/admin/dns/nodelocaldns.yaml
```

---

### Scenario 20: GitOps pipeline keeps reverting manual changes

**Problem:** DevOps team made an emergency change directly with `kubectl`. ArgoCD reverted it 2 minutes later.

**Answer:**
```bash
# This is actually CORRECT behavior of GitOps!
# The fix is: update the Git repository, not kubectl directly

# 1. Make the change in Git
git clone https://github.com/org/k8s-manifests
# Edit the manifest
git commit -m "emergency: increase replicas to 10"
git push origin main

# 2. ArgoCD will sync automatically (or trigger manually)
argocd app sync my-app

# 3. If you MUST make emergency kubectl change:
# a) Suspend ArgoCD sync temporarily
argocd app set my-app --sync-policy none

# b) Make the change
kubectl scale deployment my-app --replicas=10

# c) Update Git to match ASAP
# d) Re-enable sync
argocd app set my-app --sync-policy automated

# Best practice: All changes through Git, even emergencies
# Create a fast-track "hotfix" PR process
```

---

### Scenario 21: Pods evicted due to disk pressure

**Problem:** Kubernetes started evicting pods. Nodes are showing DiskPressure.

**Answer:**
```bash
# 1. Check node disk usage
kubectl describe node <node> | grep -A 5 "Conditions"
# DiskPressure: True

ssh <node>
df -h
# Find which partition is full

# 2. Clean up unused Docker images
docker image prune -a --force
docker system prune --force

# 3. Clean up old container logs
# Logs are in /var/log/pods/ or /var/lib/docker/containers/
find /var/log/pods -name "*.log" -size +100M -delete

# 4. Increase eviction thresholds in kubelet config
# /var/lib/kubelet/config.yaml
evictionHard:
  nodefs.available: "10%"
  imagefs.available: "15%"

# 5. Long-term: Set log rotation
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  rotate_age: "7"
  rotate_size: "100M"
EOF
```

---

### Scenario 22: Multiple teams sharing one cluster — resource conflicts

**Problem:** The frontend team's deployment is eating all cluster CPU, starving the backend team.

**Answer:**
```bash
# Solution: Namespace isolation with ResourceQuotas

# 1. Create isolated namespaces
kubectl create namespace frontend-team
kubectl create namespace backend-team

# 2. Apply ResourceQuota per team
kubectl apply -f - <<EOF
apiVersion: v1
kind: ResourceQuota
metadata:
  name: frontend-quota
  namespace: frontend-team
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "30"
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: backend-quota
  namespace: backend-team
spec:
  hard:
    requests.cpu: "15"
    requests.memory: 30Gi
    limits.cpu: "30"
    limits.memory: 60Gi
    pods: "50"
EOF

# 3. Set up RBAC so teams can't touch each other's namespaces
kubectl create rolebinding frontend-admin \
  --clusterrole=admin \
  --group=frontend-team \
  --namespace=frontend-team
```

---

### Scenario 23: Pod stuck in Terminating state

**Problem:** `kubectl delete pod <pod>` ran but pod has been "Terminating" for 20 minutes.

**Answer:**
```bash
# 1. Check if pod has finalizers
kubectl get pod <pod> -o json | jq '.metadata.finalizers'

# 2. Check what's preventing termination
kubectl describe pod <pod>
# Look for: preStop hooks, finalizers, volume unmount issues

# 3. Force delete (use with caution!)
kubectl delete pod <pod> --force --grace-period=0

# 4. If finalizers are blocking
kubectl patch pod <pod> -p \
  '{"metadata":{"finalizers":[]}}' \
  --type=merge

# 5. Check if the node is down (kubelet can't report termination)
kubectl get node $(kubectl get pod <pod> -o jsonpath='{.spec.nodeName}')

# 6. If node is down, delete the pod object
kubectl delete pod <pod> --grace-period=0 --force

# Root cause: Usually preStop hook hanging or storage unmount failing
```

---

### Scenario 24: Job runs forever — no timeout set

**Problem:** A data processing Job has been running for 8 hours. It should take 30 minutes max.

**Answer:**
```bash
# 1. Check job status
kubectl describe job my-data-job
kubectl logs job/my-data-job

# 2. Kill the runaway job
kubectl delete job my-data-job

# 3. Fix the job spec with timeout
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: my-data-job
spec:
  activeDeadlineSeconds: 1800    # 30 min max
  backoffLimit: 3                # retry 3 times on failure
  template:
    spec:
      containers:
      - name: processor
        image: my-processor:v1
        resources:
          limits:
            cpu: "2"
            memory: "4Gi"
      restartPolicy: OnFailure
EOF

# 4. Add CronJob monitoring
kubectl get cronjobs -w
```

---

### Scenario 25: Kubernetes API server slow — all kubectl commands timeout

**Problem:** Every kubectl command takes 30+ seconds. Cluster has 500 nodes and 10,000 pods.

**Answer:**
```bash
# 1. Check API server metrics
kubectl get --raw /metrics | grep apiserver_request_duration

# 2. Check etcd health and latency
ETCDCTL_API=3 etcdctl endpoint health
ETCDCTL_API=3 etcdctl check perf

# 3. Check etcd disk speed (needs fast SSD)
fio --name=etcd-test --rw=write --ioengine=sync --fdatasync=1 \
  --directory=/var/lib/etcd --size=22m --bs=2300 --iodepth=1

# 4. Check watch cache
# Large clusters need larger watch cache
# Add --watch-cache-sizes to kube-apiserver

# 5. Check LIST operations (expensive on large clusters)
kubectl get pods --all-namespaces | wc -l  # if very large → slow

# 6. Fix: Enable API priority and fairness
# Ensure APF is configured (default in 1.20+)
kubectl get flowschemas
kubectl get prioritylevelconfigurations

# 7. Optimize etcd — compact and defrag
ETCDCTL_API=3 etcdctl compact $(etcdctl endpoint status --write-out="json" | \
  jq '.[0].Status.header.revision')
ETCDCTL_API=3 etcdctl defrag
```

---

### Scenario 26: Canary deployment — 5% traffic to new version

**Problem:** You want to release v2 to only 5% of users first, then gradually increase.

**Answer:**
```bash
# Using weighted routing via replica count trick:
# 19 pods of v1 + 1 pod of v2 = ~5% to v2

kubectl apply -f - <<EOF
# v1 deployment (95%)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v1
spec:
  replicas: 19
  selector:
    matchLabels:
      app: my-app
      version: v1
  template:
    metadata:
      labels:
        app: my-app    # Both share this label
        version: v1
    spec:
      containers:
      - name: app
        image: myapp:v1
---
# v2 canary (5%)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: v2
  template:
    metadata:
      labels:
        app: my-app    # Same label — Service will route to both
        version: v2
    spec:
      containers:
      - name: app
        image: myapp:v2
---
# Single service routes to both
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app    # Matches both v1 and v2 pods
EOF

# Gradually increase canary
kubectl scale deployment my-app-v2 --replicas=5   # 20%
kubectl scale deployment my-app-v1 --replicas=15  # 75%

# Full rollout
kubectl scale deployment my-app-v2 --replicas=20
kubectl delete deployment my-app-v1
```

---

### Scenario 27: Security breach — unauthorized pod created in cluster

**Problem:** Security team detected an unknown pod running in the `kube-system` namespace.

**Answer:**
```bash
# 1. Immediately capture evidence
kubectl get pod <suspicious-pod> -o yaml > evidence.yaml
kubectl logs <suspicious-pod> > pod-logs.txt
kubectl describe pod <suspicious-pod>

# 2. Isolate the pod (block network without deleting evidence)
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: isolate-suspicious
  namespace: kube-system
spec:
  podSelector:
    matchLabels:
      <suspicious-pod-label>: "true"
  policyTypes:
  - Ingress
  - Egress
EOF

# 3. Check how it was created
kubectl get events -n kube-system --sort-by='.lastTimestamp'
kubectl get rolebindings,clusterrolebindings -A | grep suspicious

# 4. Audit API server logs
grep "suspicious-pod" /var/log/kube-apiserver.log

# 5. Remove the pod
kubectl delete pod <suspicious-pod> -n kube-system --force

# 6. Rotate credentials, audit RBAC
kubectl delete clusterrolebinding <compromised-binding>
```

---

### Scenario 28: Liveness probe kills healthy pods under high load

**Problem:** Under traffic spikes, liveness probe times out and Kubernetes keeps killing healthy pods.

**Answer:**
```bash
# Current bad config:
# livenessProbe:
#   httpGet:
#     path: /health
#   timeoutSeconds: 1      ← too short
#   periodSeconds: 5
#   failureThreshold: 1    ← too aggressive

# Fix: Make liveness probe more tolerant
kubectl patch deployment my-app --patch '
{"spec":{"template":{"spec":{"containers":[{
  "name":"app",
  "livenessProbe":{
    "httpGet":{"path":"/health","port":8080},
    "initialDelaySeconds":30,
    "periodSeconds":10,
    "timeoutSeconds":5,
    "failureThreshold":3,
    "successThreshold":1
  },
  "readinessProbe":{
    "httpGet":{"path":"/ready","port":8080},
    "periodSeconds":5,
    "timeoutSeconds":3,
    "failureThreshold":3
  }
}]}}}}'

# Key principle:
# Liveness = "is the app alive?" → should be lenient
# Readiness = "can the app handle traffic?" → can be stricter
```

---

### Scenario 29: Kubernetes version incompatibility with cloud provider LB

**Problem:** After upgrading K8s, LoadBalancer services stop getting external IPs.

**Answer:**
```bash
# 1. Check service status
kubectl describe service my-lb-service
# Events: "Error creating load balancer"

# 2. Check cloud provider controller logs
kubectl logs -n kube-system -l app=cloud-controller-manager

# 3. Check if CCM version is compatible
# AWS: check aws-load-balancer-controller version
kubectl describe deployment aws-load-balancer-controller -n kube-system | grep Image

# 4. Upgrade CCM to compatible version
helm upgrade aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=my-cluster

# 5. Check IAM permissions
# AWS: ensure node role has elasticloadbalancing:* permissions

# 6. Verify after fix
kubectl get service my-lb-service -w
# Wait for EXTERNAL-IP to appear
```

---

### Scenario 30: Pod can reach internet but can't reach internal services after CNI change

**Problem:** After migrating from Flannel to Calico, pods can reach the internet but not internal services.

**Answer:**
```bash
# 1. Test connectivity
kubectl exec -it <pod> -- curl https://google.com          # works
kubectl exec -it <pod> -- curl http://my-service:8080     # fails
kubectl exec -it <pod> -- nslookup my-service             # works or fails?

# 2. Check if Calico NetworkPolicy is blocking
kubectl get networkpolicies -A
# New Calico policy might be too restrictive

# 3. Check Calico BGP peers
kubectl exec -n kube-system <calico-pod> -- calicoctl node status

# 4. Check pod CIDR overlap
kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'
# Must not overlap with service CIDR

# 5. Test with no NetworkPolicy
kubectl delete networkpolicy --all -n <namespace>   # test only!

# 6. Add allow rule for internal traffic
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-internal
spec:
  podSelector: {}
  ingress:
  - from:
    - namespaceSelector: {}   # allow from all namespaces
EOF
```

---

### Scenario 31: Helm upgrade failed mid-release — release in broken state

**Problem:** `helm upgrade` failed halfway through. Some resources were updated, others weren't.

**Answer:**
```bash
# 1. Check Helm release status
helm status my-release -n production
# Status: failed

# 2. Check what's currently deployed
helm history my-release -n production

# 3. Option A: Rollback Helm release
helm rollback my-release 3 -n production   # rollback to revision 3

# 4. Option B: Fix and force upgrade
helm upgrade my-release ./chart \
  --atomic \         # auto-rollback on failure
  --timeout 5m \
  --cleanup-on-fail \
  -n production

# 5. If release is stuck in pending
helm history my-release | grep pending
# Delete the failed release history
kubectl delete secret sh.helm.release.v1.my-release.v5 -n production

# Prevention: Always use --atomic flag
helm upgrade my-app ./chart --atomic --timeout 10m
```

---

### Scenario 32: Pods in different namespaces can't communicate

**Problem:** Frontend (namespace: web) can't reach backend (namespace: api).

**Answer:**
```bash
# By default, pods across namespaces CAN communicate
# Problem is likely NetworkPolicy blocking it

# 1. Test connectivity
kubectl exec -it <frontend-pod> -n web -- \
  curl http://backend-service.api.svc.cluster.local:8080

# 2. Check NetworkPolicy in api namespace
kubectl get networkpolicies -n api
kubectl describe networkpolicy -n api

# 3. If NetworkPolicy is blocking, add rule
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-web-namespace
  namespace: api
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: web   # allow from web namespace
    ports:
    - protocol: TCP
      port: 8080
EOF

# 4. Test again
kubectl exec -it <frontend-pod> -n web -- \
  curl http://backend-service.api.svc.cluster.local:8080
```

---

### Scenario 33: StatefulSet pods not getting stable network identities

**Problem:** StatefulSet pods are getting random names instead of `pod-0`, `pod-1`.

**Answer:**
```bash
# Check StatefulSet
kubectl describe statefulset my-db

# Common issue: using Deployment instead of StatefulSet
# StatefulSet gives: my-db-0, my-db-1, my-db-2
# Deployment gives: my-db-abc12, my-db-def34 (random)

# Check if headless service exists (required for StatefulSet DNS)
kubectl get service my-db-headless
# If missing:
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: my-db-headless
spec:
  clusterIP: None    # Headless!
  selector:
    app: my-db
  ports:
  - port: 5432
EOF

# Each pod now has DNS:
# my-db-0.my-db-headless.namespace.svc.cluster.local
# my-db-1.my-db-headless.namespace.svc.cluster.local

# Check pod ordering
kubectl get pods -l app=my-db --sort-by='.metadata.name'
```

---

### Scenario 34: Running out of IP addresses in the cluster

**Problem:** `kubectl get pods` shows pods stuck in `Pending` with error "no IP addresses available".

**Answer:**
```bash
# 1. Check IP exhaustion
kubectl describe node <node> | grep "Allocatable"
# pods: 110   ← default max pods per node

# 2. Check current pod count per node
kubectl get pods --all-namespaces -o wide | \
  awk '{print $8}' | sort | uniq -c | sort -rn

# 3. Check CIDR blocks
kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'
# /24 = 256 IPs per node

# 4. Fix: Expand pod CIDR (requires CNI reconfiguration)
# This is complex — plan ahead with /16 or /22 per node

# 5. Short-term: Add more nodes
# New nodes = new CIDR blocks = more IPs

# 6. Clean up unused pods/completed jobs
kubectl delete pods --field-selector status.phase=Succeeded -A
kubectl delete pods --field-selector status.phase=Failed -A

# 7. Increase max pods per node (EKS)
# Requires VPC CNI configuration change
kubectl set env daemonset aws-node -n kube-system WARM_IP_TARGET=3
```

---

### Scenario 35: Monitoring alert — pod restart count > 10 in 1 hour

**Problem:** PagerDuty alert fired. `my-auth-service` has restarted 14 times in the last hour.

**Answer:**
```bash
# 1. Acknowledge alert, start investigation
kubectl get pods -n production | grep auth-service

# 2. Get crash details
kubectl describe pod <auth-service-pod> -n production
# Key: Exit Code, Last State

# 3. Get logs from crashed instance
kubectl logs <auth-service-pod> --previous -n production | tail -50

# 4. Check if specific time pattern
kubectl get events -n production | grep auth-service

# 5. Check resource pressure
kubectl top pod <auth-service-pod> -n production

# 6. Check dependencies (DB, Redis, external APIs)
kubectl logs <auth-service-pod> | grep -i "connection\|timeout\|refused"

# 7. Quick mitigation — add liveness probe grace period
kubectl patch deployment auth-service -n production --patch '
{"spec":{"template":{"spec":{"containers":[{
  "name":"auth",
  "livenessProbe":{"failureThreshold":5,"periodSeconds":10}
}]}}}}'

# 8. If DB connection issue — check DB pod/service
kubectl get pods -n production | grep database
kubectl exec -it <auth-pod> -- nc -zv db-service 5432

# 9. Escalation path:
# - Notify on-call dev to check application logs
# - Check recent deployments
# - If critical: rollback last deployment
```

---

## 📊 Quick Reference Diagrams

### Pod Troubleshooting Flow
```
kubectl get pods
       │
   STATUS?
   ┌────┴────────────────────┐
Pending              CrashLoopBackOff
   │                        │
No resources?       kubectl logs --previous
Node selector?             │
Taints/tolerations?  Exit code?
                    0→app exits  137→OOM  1→app error
```

### Networking Debug Flow
```
kubectl exec pod -- curl service
        │
    FAIL?
    │
    Check endpoints  →  Empty? → Service selector mismatch
    │
    Check NetworkPolicy → Blocking? → Add allow rule
    │
    Check CoreDNS → Down? → kubectl rollout restart coredns
```

### Storage Debug Flow
```
PVC Pending
    │
kubectl describe pvc
    │
StorageClass exists? → No → Create StorageClass
    │
Provisioner running? → No → Fix provisioner pod
    │
Node has capacity? → No → Add storage
```

---

## 🧠 Quick Commands Reference

```bash
# Pod debugging
kubectl get pods -A                              # all namespaces
kubectl describe pod <pod>                       # full details
kubectl logs <pod> --previous                    # last crash logs
kubectl exec -it <pod> -- /bin/sh               # shell into pod
kubectl top pod <pod>                            # resource usage

# Deployment management
kubectl rollout status deployment/<name>         # check rollout
kubectl rollout undo deployment/<name>           # rollback
kubectl rollout history deployment/<name>        # revision history
kubectl scale deployment/<name> --replicas=5     # manual scale

# Node management
kubectl cordon <node>                            # stop scheduling
kubectl drain <node> --ignore-daemonsets        # evict pods
kubectl uncordon <node>                         # resume scheduling
kubectl top nodes                               # node resources

# Networking
kubectl get endpoints <service>                 # check service targets
kubectl exec <pod> -- nslookup <service>       # DNS test
kubectl exec <pod> -- curl http://<service>    # connectivity test

# Storage
kubectl get pvc -A                             # all PVCs
kubectl describe pvc <name>                    # PVC details
kubectl get pv                                 # persistent volumes

# Security
kubectl auth can-i <verb> <resource>          # permission check
kubectl get networkpolicies -A                # list policies
kubectl get secrets -A                        # list secrets
```

---

*© Kubernetes & Docker Complete Interview Guide*
*Duplicates removed · 35+ scenarios · All answers in English · Production-ready*
MDEOF
echo "Done! Lines: $(wc -l < /mnt/user-data/outputs/Kubernetes-Docker-Complete-Guide.md)"
Output

Done! Lines: 2991
