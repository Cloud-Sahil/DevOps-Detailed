# ☸️ Kubernetes  — Complete Interview Guide
---

## 📁 Table of Contents

| # | Section | Questions |
|---|---------|-----------|
| 1 | [What is Kubernetes?](#1-what-is-kubernetes) | K8s overview, Docker Swarm vs K8s |
| 2 | [Pod Troubleshooting](#2-pod-troubleshooting) | Failed pods, CrashLoopBackOff, probes |
| 3 | [Resource Management](#3-resource-management) | CPU, memory, HPA, limits |
| 4 | [Deployments & Updates](#4-deployments--updates) | Rolling, blue-green, zero-downtime |
| 5 | [Node Management](#5-node-management) | Node failure, NotReady, upgrades |
| 6 | [Networking](#6-networking) | Pods, DNS, overlay, ingress, services |
| 7 | [Storage](#7-storage) | PVC, PV, StorageClass |
| 8 | [Security & RBAC](#8-security--rbac) | Secrets, RBAC, hardening |
| 9 | [Monitoring & Observability](#9-monitoring--observability) | Prometheus, Grafana, alerts |
| 10 | [Disaster Recovery](#10-disaster-recovery) | etcd backup, Velero, multi-env |
| 11 | [Docker Specific](#11-docker-specific) | Image issues, local vs K8s |
| 12 | [Scenario-Based Questions](#12-scenario-based-questions) | 35 real-world scenarios |
| 13 | [HCL — Terraform + K8s](#13-hcl--terraform--kubernetes) | IaC for K8s resources |
| 14 | [Quick Commands Reference](#14-quick-commands-reference) | Shell cheatsheet |

---


## 1. What is Kubernetes?

### What is K8s?

Kubernetes (K8s) is an open-source **container orchestration platform** that automates deployment, scaling, and management of containerized applications.

```
Without K8s:  Manual container management → error-prone, unscalable
With K8s:     Automated scheduling, healing, scaling → production-grade
```

### Docker Swarm vs Kubernetes

| Feature | Docker Swarm 🐳 | Kubernetes ☸️ |
|---------|----------------|--------------|
| Setup | Easy & quick | Complex |
| Learning Curve | Low | High |
| Scalability | Limited | Highly scalable |
| Auto Scaling | Basic | Advanced (HPA, VPA) |
| Load Balancing | Built-in | Advanced |
| Networking | Simple overlay | Advanced (CNI) |
| Self-Healing | Basic | Strong (auto-restart, reschedule) |
| Ecosystem | Limited | Huge (Helm, Istio, ArgoCD) |
| Use Case | Small projects | Enterprise / production |

### Architecture Comparison

```
Docker Swarm                    Kubernetes
--------------                  ------------------
Manager Node                    Master Node (Control Plane)
   │                               │
   ├── Worker Node 1               ├── Worker Node 1
   ├── Worker Node 2               ├── Worker Node 2
   └── Worker Node 3               └── Worker Node 3
         │                               │
   Containers                      Pods (Containers)
```

### K8s Control Plane Components

```
┌─────────────── Control Plane ────────────────┐
│  API Server  ←── kubectl (you)               │
│      │                                       │
│  Scheduler   etcd   Controller Manager       │
└──────────────────────┬───────────────────────┘
                       │
         ┌─────────────┼─────────────┐
         ▼             ▼             ▼
    Worker Node 1  Worker Node 2  Worker Node 3
    (kubelet)      (kubelet)      (kubelet)
    (kube-proxy)   (kube-proxy)   (kube-proxy)
    Pod Pod Pod    Pod Pod Pod    Pod Pod Pod
```

---

## 2. Pod Troubleshooting

### Q1. How would you troubleshoot a failed Pod / pods restarting continuously?

**Step 1 — Check pod status:**
```bash
#!/bin/bash
# Check pod status in namespace
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
# Look for: Status, Restart Count, Last State, Events
```

**Step 2 — Check logs:**
```bash
#!/bin/bash
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous   # logs from last crash
kubectl logs <pod-name> -c <container-name>         # multi-container pod
```

**Step 3 — Common causes and fixes:**

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `OOMKilled` | Memory limit too low | Increase `resources.limits.memory` |
| `Error 1` / `Error 137` | Application crash | Check app logs |
| `CrashLoopBackOff` | App keeps crashing | Fix app code / config |
| `Pending` | No node has capacity | Scale cluster |
| `ImagePullBackOff` | Image not found | Check image name/tag |
| `ContainerCreating` | Volume mount issue | Check PVC, ConfigMap |

**Step 4 — Events and debug:**
```bash
#!/bin/bash
# Check events sorted by time
kubectl get events -n <namespace> --sort-by='.lastTimestamp'

# Shell into pod for debugging
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh

# Resource usage
kubectl top pod <pod-name> -n <namespace>
```

---

### Q2. How would you troubleshoot CrashLoopBackOff / containers exiting immediately?

`CrashLoopBackOff` = container starts → crashes → K8s restarts with backoff: 10s → 20s → 40s → 80s → 5min

```bash
#!/bin/bash
# Get crash details
kubectl describe pod <pod-name>

# Current logs
kubectl logs <pod-name>

# Logs from BEFORE the crash (most useful!)
kubectl logs <pod-name> --previous

# All containers
kubectl logs <pod-name> --all-containers=true
```

**Exit Code Reference:**

| Exit Code | Meaning | Solution |
|-----------|---------|---------|
| `0` | Unexpected success exit | Add `command: ["sleep","infinity"]` to debug |
| `1` | Application error | Check app logs |
| `137` | OOM Killed (SIGKILL) | Increase memory limits |
| `139` | Segfault | Application bug |
| `143` | SIGTERM | Check liveness probe timing |

**Root causes:**
1. Wrong command/entrypoint → exits immediately
2. Missing env vars → app crashes on startup
3. Wrong ConfigMap/Secret → bad config
4. DB not reachable → connection refused on startup
5. Resource limits too low → OOMKilled
6. Liveness probe too aggressive → kills before app ready

```yaml
# Fix: Add startup probe for slow-starting apps
startupProbe:
  httpGet:
    path: /health
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

---

### Q3. What will you verify if readiness probe fails unexpectedly?

A failing readiness probe removes pod from Service endpoints — traffic stops but pod keeps running.

```bash
#!/bin/bash
# Check probe configuration
kubectl describe pod <pod-name> | grep -A 10 "Readiness"

# Check events
kubectl describe pod <pod-name> | grep -A 5 "Events"

# Test endpoint manually
kubectl exec -it <pod> -- curl localhost:8080/health
```

| Check | Command |
|-------|---------|
| App listening? | `kubectl exec <pod> -- curl localhost:8080/health` |
| Port correct? | `kubectl get pod <pod> -o yaml \| grep readiness` |
| Path correct? | Check app has `/health` or `/ready` route |
| initialDelay long enough? | Increase `initialDelaySeconds` |

```yaml
# Fix: Proper readiness probe config
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 5
  failureThreshold: 3
  successThreshold: 1
```

---

## 3. Resource Management

### Q4. How would you handle high CPU or memory usage in a cluster?

```bash
#!/bin/bash
# Node-level usage
kubectl top nodes

# Pod-level usage, sorted
kubectl top pods --all-namespaces --sort-by=cpu
kubectl top pods --all-namespaces --sort-by=memory

# Find resource-hungry pods
kubectl top pods --all-namespaces | sort -k4 -rn | head -20

# Check if limits are set
kubectl describe pod <pod> | grep -A 5 "Limits"
kubectl describe pod <pod> | grep -A 5 "Requests"
```

| Problem | Fix |
|---------|-----|
| No resource limits | Add `resources.limits` |
| Memory leak | Restart pod + fix app + set limits |
| CPU throttling | Increase limits or optimize code |
| Namespace consuming all | Apply ResourceQuota |
| Node pressure | Scale cluster |

```yaml
# Apply ResourceQuota to namespace
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
---
# Set default LimitRange
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

```bash
#!/bin/bash
# Manual scaling
kubectl scale deployment my-app --replicas=10

# Check HPA status
kubectl get hpa
kubectl describe hpa my-app-hpa

# Install metrics-server if HPA not working
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

```yaml
# Horizontal Pod Autoscaler
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

```yaml
# Vertical Pod Autoscaler
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

---

### Q6. How would you optimize Kubernetes resource utilization?

```yaml
# 1. Set accurate resource requests and limits
resources:
  requests:
    cpu: "100m"      # Scheduler uses this for placement
    memory: "128Mi"
  limits:
    cpu: "500m"      # Hard cap
    memory: "512Mi"
```

```yaml
# 2. Pod Disruption Budget for safe scaling
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

```yaml
# 3. Pod Anti-Affinity for spreading across nodes
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

Other optimizations:
- Use spot/preemptible nodes for non-critical workloads
- Schedule batch jobs during off-peak hours using CronJobs
- Use VPA recommendations for right-sizing

---

### Q7. What will you check if HPA is not scaling during traffic spikes?

```bash
#!/bin/bash
# Check HPA status and conditions
kubectl describe hpa <hpa-name>
kubectl get hpa

# Check metrics-server
kubectl get pods -n kube-system | grep metrics-server

# Check resource requests defined on deployment
kubectl describe deploy <deployment-name> | grep -A 5 "Limits\|Requests"

# Check current CPU
kubectl top pods
```

| Check | Fix |
|-------|-----|
| metrics-server missing | Install metrics-server |
| No resource requests | Add `resources.requests` to deployment |
| Already at maxReplicas | Increase `maxReplicas` |
| Wrong target utilization | Adjust `averageUtilization` |

---

## 4. Deployments & Updates

### Q8. How would you manage zero-downtime deployments / troubleshoot failed rolling updates?

```yaml
# Zero-downtime rolling update config
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Extra pods during update
      maxUnavailable: 0  # Never take down before new ones ready
  template:
    spec:
      containers:
      - name: app
        image: myapp:v2
        readinessProbe:      # REQUIRED for zero-downtime
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
```

```bash
#!/bin/bash
# Monitor rolling update
kubectl rollout status deployment/my-app
kubectl rollout history deployment/my-app

# Pause and investigate if needed
kubectl rollout pause deployment/my-app
kubectl rollout resume deployment/my-app

# Rollback
kubectl rollout undo deployment/my-app
kubectl rollout undo deployment/my-app --to-revision=3
```

**Blue-Green Deployment:**
```yaml
# Blue (current live)
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
# Green (new version)
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
# Service — switch by changing version label
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
    version: blue   # Change to 'green' to switch traffic
  ports:
  - port: 80
    targetPort: 8080
```

```bash
#!/bin/bash
# Switch to green
kubectl patch service my-app-service \
  -p '{"spec":{"selector":{"version":"green"}}}'

# Rollback to blue
kubectl patch service my-app-service \
  -p '{"spec":{"selector":{"version":"blue"}}}'
```

| | Blue-Green | Rolling |
|--|-----------|---------|
| Downtime | Zero | Near-zero |
| Resource cost | Double | Normal |
| Rollback speed | Instant | Slow |

---

### Q9. What will you investigate if deployment rollout gets stuck midway?

```bash
#!/bin/bash
# Check rollout
kubectl rollout status deployment/my-app

# Investigate
kubectl describe deployment my-app | tail -20
kubectl get pods | grep my-app
kubectl logs <new-pod-name> --previous
kubectl describe pod <new-pod-name>

# Check resource pressure
kubectl describe nodes | grep -A 5 "Conditions"
```

---

### Q10. What will you verify if rollback still does not restore stability?

If rollback doesn't fix it, the problem is NOT in the application code.

```bash
#!/bin/bash
# 1. Verify which revision is running
kubectl rollout history deployment/my-app
kubectl describe pods | grep "Image:"

# 2. Check external config changes
kubectl describe configmap <app-config>
kubectl get secret <app-secret> -o yaml

# 3. Check infrastructure changes
kubectl get nodes
kubectl get networkpolicies

# 4. Check data corruption
kubectl describe pvc
kubectl exec -it <pod> -- ls /data
```

Common non-code causes: DB schema migration, Secret update, new NetworkPolicy, corrupt PV data.

---

## 5. Node Management

### Q11. How would you handle node failure in production?

```bash
#!/bin/bash
# 1. Identify failed node
kubectl get nodes
# STATUS: NotReady

# 2. Get details
kubectl describe node <node-name>

# 3. Check pods on failed node
kubectl get pods --all-namespaces -o wide | grep <node-name>

# 4. Cordon + Drain (speed up recovery)
kubectl cordon <failed-node>
kubectl drain <failed-node> \
  --ignore-daemonsets \
  --delete-emptydir-data \
  --force

# 5. After node repair
kubectl uncordon <node-name>
```

K8s auto-heals after 5 min (`pod-eviction-timeout`) — Deployments/ReplicaSets reschedule automatically. DaemonSets do NOT reschedule.

---

### Q12. What will you investigate if node becomes NotReady intermittently?

```bash
#!/bin/bash
# Check node conditions
kubectl describe node <node-name>
# Look for: MemoryPressure, DiskPressure, PIDPressure, NetworkUnavailable

# Check node events
kubectl get events --field-selector involvedObject.name=<node-name>

# SSH and check kubelet
ssh <node-ip>
systemctl status kubelet
journalctl -u kubelet -n 100
```

| Condition | Cause | Fix |
|-----------|-------|-----|
| `MemoryPressure` | Node OOM | Evict pods, add node |
| `DiskPressure` | Disk full | `docker system prune`, expand disk |
| `PIDPressure` | Too many processes | Find process leak |
| `NetworkUnavailable` | CNI issue | Restart CNI plugin |
| `kubelet not reporting` | kubelet crash | `systemctl restart kubelet` |

---

### Q13. How would you reduce downtime during cluster upgrades?

```bash
#!/bin/bash
# Pre-upgrade: backup etcd
ETCDCTL_API=3 etcdctl snapshot save backup.db

# Upgrade one node at a time
kubectl cordon <node-name>
kubectl drain <node-name> \
  --ignore-daemonsets \
  --delete-emptydir-data

# Upgrade node (kubeadm)
sudo kubeadm upgrade node
sudo systemctl restart kubelet

# Uncordon
kubectl uncordon <node-name>
kubectl get nodes
# Repeat for each node
```

```yaml
# PodDisruptionBudget protects availability during upgrade
apiVersion: policy/v1
kind: PodDisruptionBudget
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-critical-app
```

---

## 6. Networking

### Q14. How would you troubleshoot networking issues between Pods?

```bash
#!/bin/bash
# Test pod-to-pod connectivity
kubectl get pods -o wide
kubectl exec -it <pod-a> -- curl http://<pod-b-ip>:8080
kubectl exec -it <pod-a> -- ping <pod-b-ip>

# Test via Service DNS
kubectl exec -it <pod> -- nslookup my-service
kubectl exec -it <pod> -- curl http://my-service:8080
kubectl exec -it <pod> -- curl http://my-service.namespace.svc.cluster.local:8080

# Check NetworkPolicy
kubectl get networkpolicies -n <namespace>
kubectl describe networkpolicy <policy-name>

# Check Service endpoints
kubectl get endpoints my-service
# Empty = selector mismatch!
kubectl get pods --show-labels
kubectl describe service my-service | grep Selector

# CoreDNS check
kubectl get pods -n kube-system | grep coredns
kubectl exec -it <pod> -- nslookup kubernetes.default
```

| Problem | Fix |
|---------|-----|
| Empty endpoints | Fix service selector or pod labels |
| NetworkPolicy blocking | Add ingress/egress rule |
| CoreDNS crash | `kubectl rollout restart deployment/coredns -n kube-system` |
| CNI plugin issue | Restart CNI daemonset |

---

### Q15. What will you investigate if CoreDNS fails intermittently?

```bash
#!/bin/bash
# Check CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system <coredns-pod>

# Test DNS
kubectl run dns-test --image=busybox --rm -it -- nslookup kubernetes.default

# Check ConfigMap
kubectl describe configmap coredns -n kube-system

# Scale CoreDNS
kubectl scale deployment coredns -n kube-system --replicas=4
```

| Cause | Fix |
|-------|-----|
| Memory pressure | Increase CoreDNS memory limits |
| High query load | Scale CoreDNS replicas |
| ndots misconfiguration | Set `ndots: 2` in pod spec |

---

### Q16. What will you verify if overlay network causes communication issues?

```bash
#!/bin/bash
# Check CNI pods
kubectl get pods -n kube-system | grep -E "flannel|calico|weave|cilium"
kubectl logs -n kube-system <cni-pod>

# Test cross-node connectivity
kubectl exec -it <pod-on-node1> -- ping <pod-ip-on-node2>

# Check MTU mismatch
kubectl exec -it <pod> -- ip link show eth0 | grep mtu

# Check routing on node
ssh <node>
ip route show

# Restart CNI daemonset
kubectl rollout restart daemonset <cni-name> -n kube-system
```

CNI port requirements:
- Flannel: UDP 8285, 8472
- Calico: TCP 179 (BGP), IP-IP protocol
- Weave: TCP/UDP 6783, 6784

---

### Q17. How would you troubleshoot Ingress controller / ingress stops routing traffic?

```bash
#!/bin/bash
# Check ingress resource
kubectl describe ingress <ingress-name>

# Check ingress controller
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx <ingress-controller-pod>

# Check backend service + endpoints
kubectl get service <backend-service>
kubectl get endpoints <backend-service>
```

Troubleshooting flow:
```
Browser → Ingress Controller → Service → Pod
   1. Is ingress-nginx pod running?
   2. Is LB service working?  → kubectl get svc -n ingress-nginx
   3. Right host/path?        → kubectl describe ingress
   4. Backend selector match? → kubectl get endpoints
   5. Backend pods healthy?   → kubectl get pods
```

| Issue | Cause | Fix |
|-------|-------|-----|
| 404 Not Found | Wrong path | Check `path` and `pathType` |
| 502 Bad Gateway | Backend crashing | Fix backend pod |
| 503 Unavailable | No healthy endpoints | Check pod readiness |
| SSL errors | Cert issue | Check TLS secret |

---

### Q18. What will you verify if service works internally but fails externally?

```bash
#!/bin/bash
# Test internal
kubectl exec -it <pod> -- curl http://my-service:8080

# Check service type
kubectl get service my-service
# TYPE: ClusterIP = internal only!
```

| Type | Access | Use |
|------|--------|-----|
| `ClusterIP` | Internal only | Pod-to-pod |
| `NodePort` | NodeIP:Port | Dev, on-prem |
| `LoadBalancer` | Cloud LB IP | Production |
| `Ingress` | Domain/path | HTTP/HTTPS |

```bash
#!/bin/bash
# Check if NodePort works
curl http://<node-ip>:<node-port>

# Check cloud LB
kubectl get svc my-service
# EXTERNAL-IP: <pending> = cloud LB not provisioned

# Check DNS
nslookup my-app.example.com
```

---

## 7. Storage

### Q19. How would you handle persistent storage failures / PVC stuck in Pending?

```bash
#!/bin/bash
# Diagnose PVC
kubectl describe pvc <pvc-name>
# Look at Events section

# Check StorageClasses
kubectl get storageclass

# Check provisioner
kubectl get pods -n kube-system | grep provisioner
kubectl logs -n kube-system <provisioner-pod>

# Check PV availability
kubectl get pv
```

| Cause | Event message | Fix |
|-------|--------------|-----|
| No matching StorageClass | `no volume plugin matched` | Check `storageClassName` |
| StorageClass missing | `storageclass not found` | Create StorageClass |
| Node capacity | `no nodes available` | Add storage |
| Access mode wrong | `incompatible access mode` | Fix `accessModes` |

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

```bash
#!/bin/bash
# Check volume mounted in running pod
kubectl exec -it <pod> -- df -h
kubectl exec -it <pod> -- ls /data

# Check CSI driver logs
kubectl logs -n kube-system <csi-driver-pod>
```

---

## 8. Security & RBAC

### Q20. How would you secure Kubernetes clusters in production?

```
Security Layers:
1. API Server    → Authentication + Authorization
2. RBAC          → Who can do what
3. NetworkPolicy → Who can talk to whom
4. Pod Security  → What containers can do
5. Secrets       → How sensitive data stored
6. Image scan    → What images can run
```

```bash
#!/bin/bash
# Check RBAC enabled
kubectl api-versions | grep rbac

# Scan image
trivy image myapp:latest
```

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
---
# Namespace pod security
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
---
# Secure container config
securityContext:
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 1000
  automountServiceAccountToken: false
```

---

### Q21. How would you manage RBAC permissions?

RBAC = **Who (Subject) can do What (Verb) on Which (Resource)**

| Object | Scope | Description |
|--------|-------|-------------|
| Role | Namespace | Permissions in one namespace |
| ClusterRole | Cluster-wide | Permissions across all namespaces |
| RoleBinding | Namespace | Bind Role to subject |
| ClusterRoleBinding | Cluster-wide | Bind ClusterRole to subject |

```yaml
# Read-only Role
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

```bash
#!/bin/bash
# Check permissions
kubectl auth can-i create pods --as=developer@company.com
kubectl auth can-i delete deployments --as=developer@company.com -n production
kubectl auth can-i --list
kubectl auth can-i --list -n production
```

---

### Q22. How would you manage secrets securely?

```bash
#!/bin/bash
# Create secret from file (safer)
echo -n "mypassword" > password.txt
kubectl create secret generic db-secret \
  --from-file=password=password.txt
rm password.txt

# Check secret
kubectl get secret db-secret -o yaml
```

```yaml
# Use as env var
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password

# Use as mounted file (more secure)
volumes:
- name: secret-volume
  secret:
    secretName: db-secret
volumeMounts:
- name: secret-volume
  mountPath: /etc/secrets
  readOnly: true
```

| Tool | Use case |
|------|---------|
| HashiCorp Vault | Enterprise secret management |
| AWS Secrets Manager | AWS-native |
| Sealed Secrets | GitOps-safe |
| External Secrets Operator | Sync from external providers |

```yaml
# Encrypt secrets at rest (etcd)
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

## 9. Monitoring & Observability

### Q23. How would you monitor Kubernetes clusters effectively?

```
Production monitoring stack:
  Metrics:  Prometheus + Grafana
  Logs:     EFK (Elasticsearch + Fluentd + Kibana) OR Loki + Grafana
  Tracing:  Jaeger OR Zipkin
  Alerts:   Alertmanager
```

```bash
#!/bin/bash
# Install kube-prometheus-stack
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace

# Check metrics manually
kubectl top nodes
kubectl top pods --all-namespaces
```

```yaml
# Critical Prometheus alerts
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

| Category | Metrics to monitor |
|----------|-------------------|
| Node | CPU%, Memory%, Disk%, Network I/O |
| Pod | CPU throttling, Memory, Restart count |
| Cluster | API server latency, etcd health |
| App | Request rate, Error rate, Latency |

---

### Q24. What will you check if ConfigMap changes are not reflected?

K8s does NOT auto-restart pods when ConfigMap changes.

| Method | Auto-update? | Restart needed? |
|--------|-------------|----------------|
| Env var | ❌ Never | Yes |
| Volume mount | ✅ ~1min | No |

```bash
#!/bin/bash
# Check ConfigMap updated
kubectl describe configmap <cm-name>
kubectl get configmap <cm-name> -o yaml

# Restart pods using env var ConfigMap
kubectl rollout restart deployment/<name>

# Verify inside pod
kubectl exec -it <pod> -- env | grep MY_KEY       # env var
kubectl exec -it <pod> -- cat /etc/config/my-key  # volume
```

```bash
#!/bin/bash
# Auto-restart with Reloader
helm install reloader stakater/reloader
kubectl annotate deployment my-app \
  configmap.reloader.stakater.com/reload="my-configmap"
```

---

## 10. Disaster Recovery

### Q25. How would you implement disaster recovery?

```bash
#!/bin/bash
# 1. etcd backup
ETCDCTL_API=3 etcdctl snapshot save \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/server.crt \
  --key=/etc/etcd/server.key \
  /backup/etcd-$(date +%Y%m%d).db

# Verify backup
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-20260522.db

# 2. Full cluster backup with Velero
velero install \
  --provider aws \
  --bucket my-backup-bucket \
  --backup-location-config region=us-east-1

velero backup create daily-backup --include-namespaces production

velero schedule create daily \
  --schedule="0 2 * * *" \
  --include-namespaces production

# Restore
velero restore create --from-backup daily-backup
```

```yaml
# Multi-AZ deployment
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: critical-app
        topologyKey: topology.kubernetes.io/zone
```

| Tier | RPO | RTO | Strategy |
|------|-----|-----|---------|
| Critical | < 1 min | < 5 min | Active-active multi-region |
| Important | < 1 hour | < 30 min | Active-passive + Velero |
| Normal | < 24 hours | < 2 hours | Daily backup + restore |

---

### Q26. How would you manage multi-environment deployments?

```bash
#!/bin/bash
# Option 1: Namespaces
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace production

# Option 2: Helm values per env
helm install my-app ./chart -f values-dev.yaml -n dev
helm install my-app ./chart -f values-staging.yaml -n staging
helm install my-app ./chart -f values-prod.yaml -n production
```

```yaml
# Option 3: ArgoCD GitOps
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

## 11. Docker Specific

### Q27. What will you investigate if Docker image works locally but fails in Kubernetes?

| | Local Docker | Kubernetes |
|--|-------------|-----------|
| User | Root (default) | Non-root (enforced) |
| Filesystem | Writable | Read-only possible |
| Env vars | `.env` file | ConfigMap/Secret |
| Networking | Bridge | Pod network |
| Storage | Local paths | PVCs |

```bash
#!/bin/bash
# Test image accessibility from cluster
kubectl run test --image=myapp:v1 --restart=Never
kubectl describe pod test

# Check user
kubectl exec -it <pod> -- whoami

# Check env vars
kubectl exec -it <pod> -- env

# Check file permissions
kubectl exec -it <pod> -- ls -la /app

# Check security context
kubectl describe pod <pod> | grep -A 10 "Security Context"
```

```yaml
# Fix: Add security context
securityContext:
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000
  readOnlyRootFilesystem: false
```

---

### Q28. How would you handle image pull errors / ImagePullBackOff?

`ImagePullBackOff` = K8s tried to pull image, failed, retrying with backoff.

```bash
#!/bin/bash
# Diagnose
kubectl describe pod <pod-name>
# Look at Events: Failed to pull image...
```

| Error | Cause | Fix |
|-------|-------|-----|
| `not found` | Wrong name/tag | Fix image name |
| `unauthorized` | Registry auth failed | Add imagePullSecret |
| `connection refused` | Private registry down | Check registry URL |
| `toomanyrequests` | Docker Hub rate limit | Authenticated pull |

```bash
#!/bin/bash
# Fix private registry auth
kubectl create secret docker-registry regcred \
  --docker-server=registry.company.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=me@company.com
```

```yaml
# Reference in pod spec
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: app
    image: registry.company.com/myapp:v2
```

---

## 12. Scenario-Based Questions

### Scenario 1: Production pods OOMKilled every 2 hours
```bash
#!/bin/bash
kubectl describe pod <pod> | grep -A 5 "Limits"
kubectl top pod <pod>
kubectl get cronjobs                        # Check 14:00 batch job?
kubectl patch deployment my-app --patch '
{"spec":{"template":{"spec":{"containers":[{
  "name":"app","resources":{"limits":{"memory":"1Gi"}}
}]}}}}'
```

### Scenario 2: 100% requests failing — all pods Running
```bash
#!/bin/bash
kubectl get pods                            # Check READY: 1/1 or 0/1?
kubectl get endpoints my-service           # Empty = selector mismatch
kubectl describe pod <pod> | grep Conditions
kubectl exec -it <pod> -- curl localhost:8080/health
kubectl rollout undo deployment/my-app
```

### Scenario 3: Cluster ran out of CPU
```bash
#!/bin/bash
kubectl top nodes
kubectl top pods --all-namespaces | sort -k3 -rn | head -20
kubectl scale deployment dev-app --replicas=0 -n development
kubectl get priorityclasses
```

### Scenario 4: Secret accidentally exposed in logs
```bash
#!/bin/bash
# 1. ROTATE the credential immediately in DB/provider
# 2. Update K8s secret
kubectl create secret generic db-secret \
  --from-literal=password=newpassword \
  --dry-run=client -o yaml | kubectl apply -f -
kubectl rollout restart deployment/my-app
kubectl auth can-i get secret db-secret --as=developer@company.com
```

### Scenario 5: Deployment stuck — minReadySeconds too high
```bash
#!/bin/bash
kubectl describe deployment my-app | grep -E "minReadySeconds|Strategy"
kubectl patch deployment my-app --patch '{"spec":{"minReadySeconds":10}}'
kubectl rollout resume deployment/my-app
kubectl rollout status deployment/my-app -w
```

### Scenario 6: Developer pushed to production by mistake
```bash
#!/bin/bash
kubectl rollout history deployment/my-app -n production
kubectl rollout undo deployment/my-app -n production
kubectl rollout status deployment/my-app -n production
kubectl create rolebinding dev-read-only \
  --clusterrole=view \
  --user=developer@company.com \
  --namespace=production
```

### Scenario 7: etcd down — control plane not responding
```bash
#!/bin/bash
ssh <master-node>
systemctl status etcd
journalctl -u etcd -n 100
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-latest.db \
  --data-dir=/var/lib/etcd-restore
mv /var/lib/etcd /var/lib/etcd.old
mv /var/lib/etcd-restore /var/lib/etcd
systemctl start etcd
ETCDCTL_API=3 etcdctl member list
```

### Scenario 8: Istio sidecar breaking pods
```bash
#!/bin/bash
kubectl describe pod <pod> | grep istio
kubectl logs <pod> -c istio-proxy
kubectl label namespace production istio-injection=disabled
kubectl rollout restart deployment/my-app -n production
```

### Scenario 9: Database pod losing data on restart
```bash
#!/bin/bash
# Root cause: no PVC — data in container filesystem
kubectl describe pod mysql | grep Volumes
# Fix: Add PVC with ReadWriteOnce, mount at /var/lib/mysql
```

### Scenario 10: HPA scaling but pods Pending
```bash
#!/bin/bash
kubectl describe pod <pending-pod> | grep -A 5 Events
kubectl describe nodes | grep -A 5 "Allocated resources"
kubectl get pods -n kube-system | grep cluster-autoscaler
kubectl patch hpa my-app-hpa --patch '{"spec":{"maxReplicas":10}}'
```

### Scenario 11: Certificates expired
```bash
#!/bin/bash
kubeadm certs check-expiration
kubeadm certs renew all
mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
sleep 5
mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
cp /etc/kubernetes/admin.conf ~/.kube/config
kubectl get nodes
```

### Scenario 12: Black Friday — latency 10x
```bash
#!/bin/bash
kubectl top pods -n production
kubectl patch hpa my-app-hpa --patch '{"spec":{"maxReplicas":50}}'
kubectl annotate ingress my-ingress \
  nginx.ingress.kubernetes.io/limit-rps="200"
kubectl scale deployment my-app --replicas=20
```

### Scenario 13: Namespace deleted accidentally
```bash
#!/bin/bash
kubectl get namespace production
velero restore create --from-backup daily-backup \
  --include-namespaces production
kubectl apply -f k8s/production/ -R
```

### Scenario 14: Sidecar crashing — main pod restarts
```bash
#!/bin/bash
kubectl get pods -o json | \
  jq '.items[].status.containerStatuses[] | {name, restartCount}'
kubectl logs <pod> -c logging-sidecar --previous
kubectl set image deployment/my-app logging-sidecar=fluent/fluent-bit:2.1
```

### Scenario 15: Pod on wrong node — GDPR violation
```bash
#!/bin/bash
kubectl label node eu-node-1 region=eu
kubectl label node eu-node-2 region=eu
kubectl patch deployment my-app --patch '
{"spec":{"template":{"spec":{"nodeSelector":{"region":"eu"}}}}}'
kubectl delete pod <old-pod-on-us-node>
```

### Scenario 16: CRD broke after K8s upgrade
```bash
#!/bin/bash
kubectl get crds
kubectl api-versions | grep <group>
pluto detect-all-in-cluster     # Find deprecated APIs before upgrade
helm upgrade my-operator my-repo/my-operator --version 2.0.0
```

### Scenario 17: Containers running as root — security audit
```bash
#!/bin/bash
kubectl patch deployment my-app --patch '
{"spec":{"template":{"spec":{"securityContext":{
  "runAsUser":1000,"runAsGroup":1000,
  "fsGroup":1000,"runAsNonRoot":true
}}}}}'
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted
```

### Scenario 18: No resource limits — node crashed
```bash
#!/bin/bash
kubectl set resources deployment/runaway-app \
  --limits=cpu=500m,memory=512Mi \
  --requests=cpu=100m,memory=128Mi
kubectl uncordon <crashed-node>
```

### Scenario 19: DNS slow — latency 5x
```bash
#!/bin/bash
kubectl run dns-perf --image=busybox --rm -it -- \
  sh -c "time nslookup my-service.production.svc.cluster.local"
kubectl scale deployment coredns -n kube-system --replicas=4
# Add ndots: 2 to pod dnsConfig
```

### Scenario 20: ArgoCD reverting manual kubectl changes
```bash
#!/bin/bash
# CORRECT GitOps behavior — fix in Git, not kubectl
git commit -m "emergency: increase replicas to 10"
git push origin main
argocd app sync my-app
# For emergency kubectl: suspend sync → change → update Git → re-enable
argocd app set my-app --sync-policy none
kubectl scale deployment my-app --replicas=10
argocd app set my-app --sync-policy automated
```

### Scenario 21: Pods evicted — disk pressure
```bash
#!/bin/bash
kubectl describe node <node> | grep -A 5 Conditions
ssh <node>
df -h
docker image prune -a --force
docker system prune --force
find /var/log/pods -name "*.log" -size +100M -delete
```

### Scenario 22: Teams fighting for cluster resources
```bash
#!/bin/bash
kubectl create namespace frontend-team
kubectl create namespace backend-team
# Apply separate ResourceQuotas per namespace
kubectl create rolebinding frontend-admin \
  --clusterrole=admin \
  --group=frontend-team \
  --namespace=frontend-team
```

### Scenario 23: Pod stuck in Terminating
```bash
#!/bin/bash
kubectl get pod <pod> -o json | jq '.metadata.finalizers'
kubectl delete pod <pod> --force --grace-period=0
kubectl patch pod <pod> -p '{"metadata":{"finalizers":[]}}' --type=merge
```

### Scenario 24: Job running forever
```bash
#!/bin/bash
kubectl delete job my-data-job
# Fix: add activeDeadlineSeconds: 1800 to job spec
```

### Scenario 25: API server slow — 500 nodes
```bash
#!/bin/bash
ETCDCTL_API=3 etcdctl endpoint health
ETCDCTL_API=3 etcdctl check perf
ETCDCTL_API=3 etcdctl defrag
kubectl get flowschemas
kubectl get prioritylevelconfigurations
```

### Scenario 26: Canary — 5% traffic to v2
```bash
#!/bin/bash
# 19 pods v1 + 1 pod v2 = 5% canary
# Both share label: app: my-app
# Service selector: app: my-app (routes to both)
kubectl scale deployment my-app-v2 --replicas=5   # 20%
kubectl scale deployment my-app-v1 --replicas=15  # 75%
kubectl scale deployment my-app-v2 --replicas=20
kubectl delete deployment my-app-v1
```

### Scenario 27: Unknown pod in kube-system — breach
```bash
#!/bin/bash
kubectl get pod <suspicious-pod> -o yaml > evidence.yaml
kubectl logs <suspicious-pod> > pod-logs.txt
# Apply NetworkPolicy to isolate pod
kubectl get events -n kube-system --sort-by='.lastTimestamp'
grep "suspicious-pod" /var/log/kube-apiserver.log
kubectl delete pod <suspicious-pod> -n kube-system --force
kubectl delete clusterrolebinding <compromised-binding>
```

### Scenario 28: Liveness probe killing healthy pods under load
```bash
#!/bin/bash
# Fix: More tolerant probe settings
# timeoutSeconds: 5 (was 1)
# failureThreshold: 3 (was 1)
# Liveness = lenient (is app alive?)
# Readiness = strict (can app handle traffic?)
```

### Scenario 29: K8s upgrade broke LoadBalancer services
```bash
#!/bin/bash
kubectl describe service my-lb-service
kubectl logs -n kube-system -l app=cloud-controller-manager
helm upgrade aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system --set clusterName=my-cluster
kubectl get service my-lb-service -w
```

### Scenario 30: After CNI change — internal services unreachable
```bash
#!/bin/bash
kubectl get networkpolicies -A
kubectl delete networkpolicy --all -n <namespace>  # test only!
# Add allow-internal NetworkPolicy
# Check pod CIDR doesn't overlap service CIDR
kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'
```

### Scenario 31: Helm upgrade failed mid-release
```bash
#!/bin/bash
helm status my-release -n production
helm history my-release -n production
helm rollback my-release 3 -n production
# Prevention: helm upgrade --atomic --timeout 10m
```

### Scenario 32: Pods in different namespaces can't communicate
```bash
#!/bin/bash
kubectl exec -it <frontend-pod> -n web -- \
  curl http://backend-service.api.svc.cluster.local:8080
kubectl get networkpolicies -n api
# Add NetworkPolicy: allow from web namespace to api namespace
```

### Scenario 33: StatefulSet not getting stable names
```bash
#!/bin/bash
kubectl describe statefulset my-db
kubectl get service my-db-headless
# StatefulSet needs: clusterIP: None (headless service)
# DNS: my-db-0.my-db-headless.namespace.svc.cluster.local
```

### Scenario 34: Running out of Pod IPs
```bash
#!/bin/bash
kubectl describe node <node> | grep "Allocatable"
kubectl delete pods --field-selector status.phase=Succeeded -A
kubectl delete pods --field-selector status.phase=Failed -A
# Add more nodes (new CIDR blocks = more IPs)
```

### Scenario 35: Alert — pod restarted 14 times in 1 hour
```bash
#!/bin/bash
kubectl describe pod <auth-pod> -n production
kubectl logs <auth-pod> --previous -n production | tail -50
kubectl top pod <auth-pod> -n production
kubectl exec -it <auth-pod> -- nc -zv db-service 5432
kubectl rollout undo deployment/auth-service  # if recent deploy caused it
```

---

## 13. HCL — Terraform + Kubernetes

```hcl
# ============================================
# Terraform backend for K8s state
# Language: HCL
# ============================================

terraform {
  required_version = ">= 1.5.0"

  required_providers {
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "k8s-infra/terraform.tfstate"
    region         = "eu-north-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

```hcl
# ============================================
# EKS cluster via Terraform
# Language: HCL
# ============================================

provider "aws" {
  region = var.aws_region
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = var.cluster_name
  cluster_version = "1.29"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    production = {
      instance_types = ["t3.medium"]
      min_size       = 2
      max_size       = 10
      desired_size   = 3

      labels = {
        environment = "production"
        role        = "worker"
      }
    }
  }

  cluster_addons = {
    coredns    = { most_recent = true }
    kube-proxy = { most_recent = true }
    vpc-cni    = { most_recent = true }
  }

  tags = {
    Environment = "production"
    Terraform   = "true"
  }
}
```

```hcl
# ============================================
# K8s namespace + RBAC via Terraform
# Language: HCL
# ============================================

resource "kubernetes_namespace" "production" {
  metadata {
    name = "production"
    labels = {
      "pod-security.kubernetes.io/enforce" = "restricted"
    }
  }
}

resource "kubernetes_resource_quota" "production_quota" {
  metadata {
    name      = "production-quota"
    namespace = kubernetes_namespace.production.metadata[0].name
  }

  spec {
    hard = {
      "requests.cpu"    = "10"
      "requests.memory" = "20Gi"
      "limits.cpu"      = "20"
      "limits.memory"   = "40Gi"
      "pods"            = "50"
    }
  }
}

resource "kubernetes_cluster_role" "readonly" {
  metadata {
    name = "cluster-readonly"
  }

  rule {
    api_groups = [""]
    resources  = ["pods", "services", "configmaps"]
    verbs      = ["get", "list", "watch"]
  }
}
```

```hcl
# ============================================
# Variables
# Language: HCL
# ============================================

variable "cluster_name" {
  type        = string
  description = "EKS cluster name"
  default     = "production-cluster"
}

variable "aws_region" {
  type        = string
  description = "AWS region"
  default     = "eu-north-1"
}

variable "environment" {
  type        = string
  description = "Deployment environment"
  default     = "production"

  validation {
    condition     = contains(["dev", "staging", "production"], var.environment)
    error_message = "Environment must be dev, staging, or production."
  }
}
```

```hcl
# ============================================
# Outputs
# Language: HCL
# ============================================

output "cluster_endpoint" {
  description = "EKS cluster endpoint"
  value       = module.eks.cluster_endpoint
}

output "cluster_name" {
  description = "EKS cluster name"
  value       = module.eks.cluster_name
}

output "kubeconfig_command" {
  description = "Command to update kubeconfig"
  value       = "aws eks update-kubeconfig --region ${var.aws_region} --name ${module.eks.cluster_name}"
}
```

---

## 14. Quick Commands Reference

```bash
#!/bin/bash
# ============================================
# KUBERNETES DAILY COMMANDS — CHEATSHEET
# Language: Shell (Bash)
# ============================================

# === POD DEBUGGING ===
kubectl get pods -A                               # all namespaces
kubectl get pods -o wide                          # with node info
kubectl describe pod <pod>                        # full details + events
kubectl logs <pod> --previous                     # last crash logs
kubectl logs <pod> -f                             # follow live logs
kubectl exec -it <pod> -- /bin/sh                # shell into pod
kubectl top pod <pod>                             # CPU/memory usage
kubectl top pod <pod> --containers                # per-container usage

# === DEPLOYMENT MANAGEMENT ===
kubectl rollout status deployment/<name>          # check rollout
kubectl rollout undo deployment/<name>            # rollback
kubectl rollout history deployment/<name>         # revision history
kubectl rollout pause deployment/<name>           # pause update
kubectl rollout resume deployment/<name>          # resume update
kubectl scale deployment/<name> --replicas=5      # manual scale
kubectl set image deployment/<name> app=img:v2   # update image

# === NODE MANAGEMENT ===
kubectl get nodes                                 # node status
kubectl describe node <node>                      # node details
kubectl cordon <node>                             # stop scheduling
kubectl drain <node> --ignore-daemonsets         # evict pods safely
kubectl uncordon <node>                          # resume scheduling
kubectl top nodes                                # node resource usage

# === NETWORKING ===
kubectl get service -A                           # all services
kubectl get endpoints <service>                  # service targets
kubectl get networkpolicies -A                   # all policies
kubectl exec <pod> -- nslookup <service>        # DNS test
kubectl exec <pod> -- curl http://<service>     # connectivity test
kubectl port-forward pod/<pod> 8080:8080        # local port forward

# === STORAGE ===
kubectl get pvc -A                               # all PVCs
kubectl describe pvc <name>                      # PVC details
kubectl get pv                                   # persistent volumes
kubectl get storageclass                         # storage classes

# === SECURITY ===
kubectl auth can-i <verb> <resource>            # permission check
kubectl auth can-i --list                       # list all permissions
kubectl get secrets -A                          # list secrets
kubectl get serviceaccounts -A                 # service accounts

# === CLUSTER INFO ===
kubectl cluster-info                            # cluster details
kubectl get events --sort-by='.lastTimestamp'  # recent events
kubectl api-resources                          # all API resources
kubectl api-versions                           # all API versions
kubectl version                               # K8s version

# === HELM ===
helm list -A                                  # all releases
helm status <release>                         # release status
helm history <release>                        # release history
helm rollback <release> <revision>            # rollback release
helm upgrade --atomic --timeout 10m           # safe upgrade

# === ETCD ===
ETCDCTL_API=3 etcdctl snapshot save backup.db              # backup
ETCDCTL_API=3 etcdctl snapshot status backup.db            # verify
ETCDCTL_API=3 etcdctl endpoint health                      # health
ETCDCTL_API=3 etcdctl defrag                               # optimize

# === CERT-MANAGER ===
kubeadm certs check-expiration                # check expiry
kubeadm certs renew all                       # renew all certs

# === VELERO ===
velero backup create my-backup --include-namespaces prod   # backup
velero restore create --from-backup my-backup              # restore
velero schedule create daily --schedule="0 2 * * *"        # schedule

# === DEBUG TOOLS ===
kubectl run debug --image=busybox --rm -it -- /bin/sh     # debug pod
kubectl run curl --image=curlimages/curl --rm -it -- sh   # curl pod
kubectl get pods --field-selector status.phase=Failed -A  # failed pods
kubectl delete pods --field-selector status.phase=Failed -A # cleanup
```

---

## 📊 Troubleshooting Flow Diagrams

### Pod Debug Flow
```
kubectl get pods
       │
   STATUS?
   ┌────┴──────────────────────────────┐
Pending                          CrashLoopBackOff
   │                                   │
No resources?               kubectl logs --previous
Node selector?                         │
Taint/toleration?             Exit Code?
                     0→exits  137→OOM  1→app error  139→segfault
```

### Networking Debug Flow
```
kubectl exec pod -- curl service:port
              │
          FAILS?
          │
   Check endpoints → Empty? → Service selector mismatch
          │
   Check NetworkPolicy → Blocking? → Add allow rule
          │
   Check CoreDNS → Down? → kubectl rollout restart coredns
          │
   Check CNI plugin → Issue? → Restart CNI daemonset
```

### Storage Debug Flow
```
PVC stuck in Pending
        │
kubectl describe pvc
        │
StorageClass exists? → No → Create StorageClass
        │
Provisioner running? → No → Fix provisioner pod
        │
AccessMode correct? → No → Fix accessModes
        │
Node has capacity? → No → Add storage / new node
```

---

*© Kubernetes & Docker Complete Interview Guide*
*8 duplicates removed · 235 unique images analyzed · 35 scenarios · Shell + HCL + YAML*
MDEOF
echo "Done! Lines: $(wc -l < /mnt/user-data/outputs/Kubernetes-Interview-Guide.md)"
