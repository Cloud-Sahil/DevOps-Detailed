# ☸️ Kubernetes — Complete Interview Guide

---

## 📁 Table of Contents

| # | Section | Topics |
|---|---------|--------|
| 1 | [What is Kubernetes](#1-what-is-kubernetes) | Overview, Docker Swarm vs K8s, Architecture |
| 2 | [Core Components](#2-core-components) | Control plane, kubelet, kube-proxy, etcd |
| 3 | [Pods & Workloads](#3-pods--workloads) | Pod, Deployment, StatefulSet, DaemonSet, Job |
| 4 | [Networking](#4-networking) | Services, Ingress, DNS, NetworkPolicy, CNI |
| 5 | [Storage](#5-storage) | PV, PVC, StorageClass, ConfigMap, Secret |
| 6 | [Security & RBAC](#6-security--rbac) | RBAC, ServiceAccount, Taints, Probes, PDB |
| 7 | [Scaling & Resources](#7-scaling--resources) | HPA, VPA, ResourceQuota, LimitRange |
| 8 | [Advanced Concepts](#8-advanced-concepts) | Helm, CRD, Operators, GitOps, Istio |
| 9 | [Monitoring & Observability](#9-monitoring--observability) | Prometheus, Grafana, alerts |
| 10 | [Disaster Recovery](#10-disaster-recovery) | etcd backup, Velero, multi-env |
| 11 | [Scenario-Based Questions](#11-scenario-based-questions) | 65 real-world scenarios with answers |
| 12 | [HCL — Terraform + K8s](#12-hcl--terraform--kubernetes) | EKS, namespaces, RBAC via IaC |
| 13 | [Quick Commands Reference](#13-quick-commands-reference) | Shell cheatsheet |

---

## Duplicates Removed (14 merged)

| Removed | Merged into |
|---------|-------------|
| "What is Kubernetes" (doc2) | Section 1 — Overview |
| "Pod troubleshooting" (doc2) | Scenario 1-2 + Section 3 |
| "Readiness probe fails" (doc2) | Q33 — Probes |
| "HPA not scaling" (doc2) | Q18 — HPA |
| "Zero-downtime deployments" (doc2) | Q11 — Deployment + Scenario 55 |
| "Blue-green deployment" (doc2) | Scenario 63 |
| "Rollout stuck/rollback" (doc2) | Scenario 48 |
| "Node failure/NotReady" (doc2) | Scenario 50 + Q12-13 |
| "Networking issues/CoreDNS/overlay" (doc2) | Section 4 |
| "Service internal/external" (doc2) | Q21 — Services |
| "Storage failures/PVC pending" (doc2) | Section 5 |
| "RBAC/secrets" (doc2) | Section 6 |
| "ConfigMap not reflected" (doc2) | Q28 — ConfigMap |
| "ImagePullBackOff" (doc2) | Scenario 61 |

---

## 1. What is Kubernetes

### Q1. What is Kubernetes and why is it used?

Kubernetes (K8s) is an open-source **container orchestration platform** originally developed by Google. It automates deployment, scaling, load balancing, and management of containerized applications.

**Why it's used:**
- **Self-healing** — restarts failed containers, replaces unhealthy nodes
- **Auto-scaling** — scales up/down based on load (HPA, VPA, Cluster Autoscaler)
- **Service discovery** and load balancing built-in
- **Declarative config** — YAML manifests describe desired state
- **Multi-cloud** — works on-prem, cloud, and hybrid

```
Without K8s:  Manual container management → error-prone, unscalable
With K8s:     Automated scheduling, healing, scaling → production-grade
```

---

### Q2. Docker Swarm vs Kubernetes

| Feature | Docker Swarm 🐳 | Kubernetes ☸️ |
|---------|----------------|--------------|
| Setup | Easy & quick | Complex |
| Learning Curve | Low | High |
| Scalability | Limited | Highly scalable |
| Auto Scaling | Basic | Advanced (HPA, VPA) |
| Load Balancing | Built-in | Advanced |
| Networking | Simple overlay | Advanced (CNI plugins) |
| Self-Healing | Basic | Strong (reschedule, restart) |
| Ecosystem | Limited | Huge (Helm, Istio, ArgoCD) |
| Use Case | Small projects | Enterprise / production |

---

### Q3. Kubernetes Architecture

```
┌─────────────────── Control Plane (Master Node) ──────────────────┐
│                                                                   │
│  kubectl ──► API Server ──► etcd (cluster state DB)              │
│                  │                                                │
│           Scheduler    Controller Manager   Cloud Controller      │
└───────────────────────────────┬───────────────────────────────────┘
                                │
              ┌─────────────────┼─────────────────┐
              ▼                 ▼                 ▼
       Worker Node 1      Worker Node 2      Worker Node 3
       ┌───────────┐      ┌───────────┐      ┌───────────┐
       │ kubelet   │      │ kubelet   │      │ kubelet   │
       │ kube-proxy│      │ kube-proxy│      │ kube-proxy│
       │ containerd│      │ containerd│      │ containerd│
       │ Pod Pod   │      │ Pod Pod   │      │ Pod Pod   │
       └───────────┘      └───────────┘      └───────────┘
```

---

## 2. Core Components

### Q4. Control Plane Components

| Component | Role |
|-----------|------|
| `kube-apiserver` | Entry point — all kubectl/API calls go through it |
| `etcd` | Distributed key-value DB — stores ALL cluster state |
| `kube-scheduler` | Assigns pods to nodes (filtering + scoring) |
| `kube-controller-manager` | Runs all controllers (node, replication, etc.) |
| `cloud-controller-manager` | Integrates with cloud provider APIs (AWS, GCP) |

### Q5. Worker Node Components

| Component | Role |
|-----------|------|
| `kubelet` | Agent; starts/stops containers; reports status to control plane |
| `kube-proxy` | Maintains iptables/IPVS rules; routes Service traffic to pods |
| Container Runtime | Actually runs containers (`containerd`, `CRI-O`) |

```bash
#!/bin/bash
# Node management commands
kubectl get nodes                    # List all nodes
kubectl describe node <name>         # Details: capacity, conditions, pods
kubectl cordon <node>                # Mark unschedulable
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
kubectl uncordon <node>              # Re-enable scheduling
kubectl top nodes                    # CPU/memory per node
```

---

### Q6. What is etcd and why is it critical?

`etcd` is a distributed key-value store — Kubernetes' entire state lives here: all resources, configs, secrets, deployments.

```bash
#!/bin/bash
# Backup etcd (CRITICAL — do this regularly!)
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%Y%m%d).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/etcd-server.crt \
  --key=/etc/etcd/etcd-server.key

# Verify backup
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-20260524.db

# Health check
ETCDCTL_API=3 etcdctl endpoint health

# Defrag (reclaim disk space)
ETCDCTL_API=3 etcdctl defrag

# Compact (remove old revisions)
ETCDCTL_API=3 etcdctl compact \
  $(ETCDCTL_API=3 etcdctl endpoint status --write-out=json | \
    jq '.[0].Status.header.revision')
```

**Why critical:** If etcd is lost without backup → entire cluster state gone. Uses **Raft consensus** for HA — run 3 or 5 replicas.

---

### Q7. What is a Namespace?

Namespaces are virtual clusters within a physical cluster — used for isolation, resource quotas, and multi-team access control.

```bash
#!/bin/bash
kubectl create namespace dev-team
kubectl get pods -n dev-team
kubectl get pods --all-namespaces        # or -A
kubectl config set-context --current --namespace=dev-team  # set default
```

**Default namespaces:**

| Namespace | Purpose |
|-----------|---------|
| `default` | Resources with no namespace specified |
| `kube-system` | K8s system components (API server, CoreDNS, etc.) |
| `kube-public` | Readable by all users |
| `kube-node-lease` | Node heartbeat objects |

---

## 3. Pods & Workloads

### Q8. Pod → ReplicaSet → Deployment

```
Deployment → manages → ReplicaSet → manages → Pods
```

| Resource | Purpose | Self-healing |
|----------|---------|--------------|
| **Pod** | Smallest unit; runs 1+ containers sharing network + storage | ❌ |
| **ReplicaSet** | Keeps N pod replicas running | ✅ |
| **Deployment** | Manages ReplicaSets + rolling updates + rollbacks | ✅ |

```yaml
# Minimal Pod
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx:1.25
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
```

```yaml
# Production Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0    # Zero-downtime
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app:v2
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
          failureThreshold: 3
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
```

```bash
#!/bin/bash
kubectl rollout status deployment/my-app
kubectl rollout history deployment/my-app
kubectl rollout undo deployment/my-app
kubectl rollout undo deployment/my-app --to-revision=3
```

---

### Q9. StatefulSet vs Deployment

| Feature | Deployment | StatefulSet |
|---------|-----------|-------------|
| Pod names | Random: `pod-abc12` | Stable: `pod-0`, `pod-1` |
| Pod creation | Simultaneous | Ordered (0→1→2) |
| Pod deletion | Simultaneous | Reverse order (2→1→0) |
| Storage | Shared PVC | Individual PVC per pod |
| DNS | Single service DNS | Per-pod DNS via headless service |
| Use case | Stateless apps | Databases, Kafka, ZooKeeper |

```yaml
# StatefulSet for MySQL cluster
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql-headless"
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

---

### Q10. DaemonSet, Job, CronJob

**DaemonSet** — Exactly one pod per node (or selected nodes):

```yaml
# DaemonSet for log collection
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.16
```

Use cases: log collectors (Fluentd), monitoring agents (Prometheus Node Exporter), CNI plugins.

**CronJob:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup
spec:
  schedule: "0 2 * * *"       # Every day at 2 AM
  concurrencyPolicy: Forbid    # Don't run if previous still running
  jobTemplate:
    spec:
      activeDeadlineSeconds: 1800   # 30 min timeout
      backoffLimit: 3
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:latest
            command: ["/bin/sh", "-c", "pg_dump mydb > /backup/dump.sql"]
          restartPolicy: OnFailure
```

---

### Q11. Init Containers & Sidecar Pattern

```yaml
spec:
  # Init containers run BEFORE main containers (sequentially)
  initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'until nc -z db-service 5432; do sleep 2; done']
  - name: db-migrate
    image: my-app:latest
    command: ['python', 'manage.py', 'migrate']

  # Main containers (run in parallel)
  containers:
  - name: app
    image: my-app:latest
  - name: log-collector       # Sidecar
    image: fluent/fluent-bit
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
```

**Common sidecars:**

| Sidecar | Purpose |
|---------|---------|
| Fluentd/Fluent Bit | Log collection + shipping |
| Envoy (Istio) | Service mesh proxy |
| Prometheus exporter | Expose app metrics |
| Git sync | Keep files up to date |

---

## 4. Networking

### Q12. Kubernetes Service Types

| Type | Access | Use Case |
|------|--------|---------|
| `ClusterIP` | Internal cluster only | Service-to-service |
| `NodePort` | NodeIP:Port (30000-32767) | Dev/testing |
| `LoadBalancer` | Cloud LB with external IP | Production |
| `ExternalName` | DNS CNAME mapping | External service alias |

```yaml
# ClusterIP Service
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: my-app         # Must match pod labels!
  ports:
  - protocol: TCP
    port: 80            # Service port
    targetPort: 8080    # Container port
---
# Headless Service (for StatefulSets)
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None       # No virtual IP — DNS returns pod IPs
  selector:
    app: mysql
  ports:
  - port: 3306
# Pod DNS: mysql-0.mysql-headless.default.svc.cluster.local
```

---

### Q13. Ingress & Ingress Controller

```yaml
# Ingress resource (nginx example)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - api.myapp.com
    secretName: api-tls-secret
  rules:
  - host: api.myapp.com
    http:
      paths:
      - path: /users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 80
```

```bash
#!/bin/bash
# Install nginx ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# Check ingress
kubectl describe ingress my-ingress
kubectl get svc -n ingress-nginx   # Get external IP
```

**Ingress troubleshooting flow:**
```
Browser → Ingress Controller Pod → Service → Pod

1. Is ingress-nginx pod running?     kubectl get pods -n ingress-nginx
2. Is LB service working?            kubectl get svc -n ingress-nginx
3. Right host/path in Ingress?       kubectl describe ingress
4. Backend selector matches pods?    kubectl get endpoints <svc>
5. Backend pods healthy?             kubectl get pods -l app=my-app
```

---

### Q14. NetworkPolicy

```yaml
# Default deny all ingress (apply this first!)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
# Allow frontend to reach backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: production
    ports:
    - protocol: TCP
      port: 8080
```

⚠️ Requires CNI that supports NetworkPolicy: **Calico, Cilium, Weave**. Flannel does NOT support it.

---

### Q15. DNS in Kubernetes

```bash
#!/bin/bash
# Service DNS format:
# <service>.<namespace>.svc.cluster.local

# Within same namespace
curl http://my-service

# Cross-namespace
curl http://my-service.other-namespace.svc.cluster.local

# Test DNS from pod
kubectl exec -it <pod> -- nslookup my-service
kubectl exec -it <pod> -- nslookup kubernetes.default

# Debug CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
kubectl describe configmap coredns -n kube-system
```

---

## 5. Storage

### Q16. PV, PVC, StorageClass

```
StorageClass → defines HOW storage is provisioned
PVC → request for storage (what you want)
PV → actual storage (what you get)
Pod → uses PVC
```

```yaml
# StorageClass (dynamic provisioning)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
reclaimPolicy: Delete             # Delete PV when PVC deleted
volumeBindingMode: WaitForFirstConsumer
---
# PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-ssd      # Must match StorageClass name
  resources:
    requests:
      storage: 10Gi
```

**Access Modes:**

| Mode | Abbreviation | Meaning |
|------|-------------|---------|
| ReadWriteOnce | RWO | One node reads/writes |
| ReadOnlyMany | ROX | Many nodes read |
| ReadWriteMany | RWX | Many nodes read/write |
| ReadWriteOncePod | RWOP | Only one pod reads/writes |

```bash
#!/bin/bash
kubectl get pvc -A                    # All PVCs
kubectl describe pvc my-pvc           # Why pending?
kubectl get pv                        # Available PVs
kubectl get storageclass              # Available storage classes
```

---

### Q17. ConfigMap and Secret

```yaml
# ConfigMap — non-sensitive config
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"
  config.yaml: |
    server:
      port: 8080
      timeout: 30s
---
# Secret — sensitive data (base64 encoded)
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:          # Use stringData — auto-encodes to base64
  DB_PASSWORD: "mysecretpassword"
  DB_HOST: "prod-db.example.com"
```

```yaml
# Use in Pod
spec:
  containers:
  - name: app
    image: my-app:latest
    # Method 1: Environment variables
    envFrom:
    - configMapRef:
        name: app-config
    - secretRef:
        name: db-secret

    # Method 2: Individual env vars
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: DB_PASSWORD

    # Method 3: Volume mount (auto-updates for ConfigMap!)
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
      readOnly: true
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

**Auto-update behavior:**

| Method | Auto-updates? | Restart needed? |
|--------|-------------|----------------|
| `envFrom` / `env` | ❌ Never | Yes — `kubectl rollout restart` |
| Volume mount | ✅ ~1 min | No (app must reload) |

---

## 6. Security & RBAC

### Q18. RBAC

```
Who (Subject) can do What (Verb) on Which (Resource)
```

| Object | Scope | Purpose |
|--------|-------|---------|
| `Role` | Namespace | Permissions within one namespace |
| `ClusterRole` | Cluster-wide | Permissions across all namespaces |
| `RoleBinding` | Namespace | Attaches Role to subject |
| `ClusterRoleBinding` | Cluster-wide | Attaches ClusterRole to subject |

```yaml
# Read-only Role for developers
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-view
  namespace: production
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log", "services", "configmaps"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-binding
  namespace: production
subjects:
- kind: User
  name: developer@company.com
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: dev-team
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-view
  apiGroup: rbac.authorization.k8s.io
```

```bash
#!/bin/bash
# Check permissions
kubectl auth can-i create pods --as=developer@company.com -n production
kubectl auth can-i delete deployments --as=developer@company.com -n production
kubectl auth can-i '*' '*'         # Are you admin?
kubectl auth can-i --list          # All my permissions
kubectl auth can-i --list -n production  # In specific namespace
```

---

### Q19. ServiceAccount

```yaml
# Dedicated ServiceAccount for app (least privilege)
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  namespace: production
automountServiceAccountToken: false   # Disable unless needed
---
# Assign to pod
spec:
  serviceAccountName: my-app-sa
```

---

### Q20. Taints and Tolerations

```bash
#!/bin/bash
# Taint a node — repels pods unless they tolerate it
kubectl taint nodes gpu-node-1 nvidia.com/gpu=true:NoSchedule
kubectl taint nodes node1 maintenance=true:NoExecute    # Evict existing pods

# Remove taint
kubectl taint nodes gpu-node-1 nvidia.com/gpu-
```

```yaml
# Pod toleration — allows scheduling on tainted nodes
spec:
  tolerations:
  - key: "nvidia.com/gpu"
    operator: "Exists"
    effect: "NoSchedule"
  - key: "maintenance"
    operator: "Equal"
    value: "true"
    effect: "NoExecute"
    tolerationSeconds: 3600   # Tolerate for 1 hour then evict
```

**Effects:**

| Effect | Meaning |
|--------|---------|
| `NoSchedule` | Don't schedule new pods |
| `PreferNoSchedule` | Avoid scheduling (soft) |
| `NoExecute` | Evict existing pods + don't schedule new |

---

### Q21. Node Affinity & Pod Anti-Affinity

```yaml
spec:
  affinity:
    # Schedule on EU nodes only (compliance)
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: region
            operator: In
            values:
            - eu-west-1
            - eu-north-1

    # Spread pods across different nodes (high availability)
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: my-app
        topologyKey: kubernetes.io/hostname

    # Prefer same zone as cache pods (performance)
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app: redis-cache
          topologyKey: topology.kubernetes.io/zone
```

---

### Q22. Liveness, Readiness & Startup Probes

| Probe | Purpose | On Failure |
|-------|---------|-----------|
| **Liveness** | Is container alive? | Restart container |
| **Readiness** | Can container serve traffic? | Remove from Service endpoints |
| **Startup** | Has slow app started? | Kill + restart after timeout |

```yaml
spec:
  containers:
  - name: app
    image: my-app:latest

    # Startup — give slow apps time to initialize
    startupProbe:
      httpGet:
        path: /health
        port: 8080
      failureThreshold: 30    # 30 * 10s = 5 min max startup time
      periodSeconds: 10

    # Readiness — traffic routing
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3
      successThreshold: 1

    # Liveness — self-healing (be lenient!)
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 10
      timeoutSeconds: 5       # Give app 5s to respond
      failureThreshold: 3     # 3 failures before restart

    # TCP probe (for non-HTTP services)
    # livenessProbe:
    #   tcpSocket:
    #     port: 3306

    # Exec probe (run command)
    # readinessProbe:
    #   exec:
    #     command: ["pg_isready", "-U", "postgres"]
```

---

### Q23. PodDisruptionBudget (PDB)

```yaml
# Keep at least 2 pods running during maintenance
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 2         # OR use maxUnavailable: 1
  selector:
    matchLabels:
      app: my-app
```

**When PDB is enforced:** Node drains, cluster upgrades, voluntary disruptions. NOT for node failures.

---

## 7. Scaling & Resources

### Q24. Resource Requests & Limits

```yaml
resources:
  requests:
    memory: "128Mi"    # Scheduler uses this for pod placement
    cpu: "250m"        # 250 millicores = 0.25 CPU core
  limits:
    memory: "256Mi"    # OOMKilled if exceeded
    cpu: "500m"        # CPU throttled if exceeded (NOT killed)
```

**QoS Classes (eviction priority):**

| Class | Condition | Eviction Priority |
|-------|-----------|------------------|
| **Guaranteed** | requests == limits | Last evicted |
| **Burstable** | requests < limits | Middle |
| **BestEffort** | No requests/limits | First evicted |

---

### Q25. HPA — Horizontal Pod Autoscaler

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
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300   # Wait 5 min before scaling down
```

```bash
#!/bin/bash
# Requires metrics-server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl get hpa
kubectl describe hpa my-app-hpa    # Check conditions + events
kubectl top pods                   # Current usage
```

---

### Q26. VPA — Vertical Pod Autoscaler

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
    updateMode: "Auto"    # Off | Initial | Recreate | Auto
  resourcePolicy:
    containerPolicies:
    - containerName: app
      minAllowed:
        cpu: "100m"
        memory: "128Mi"
      maxAllowed:
        cpu: "2"
        memory: "2Gi"
```

⚠️ **HPA + VPA on same resource = conflict.** Use HPA for CPU/memory scaling, VPA only for recommendations.

---

### Q27. ResourceQuota & LimitRange

```yaml
# ResourceQuota — total namespace limit
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: team-alpha
spec:
  hard:
    pods: "20"
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
    count/deployments.apps: "10"
    count/secrets: "20"
---
# LimitRange — default per-container limits
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: team-alpha
spec:
  limits:
  - type: Container
    default:
      cpu: "500m"
      memory: "256Mi"
    defaultRequest:
      cpu: "250m"
      memory: "128Mi"
    max:
      cpu: "2"
      memory: "2Gi"
    min:
      cpu: "50m"
      memory: "64Mi"
```

---

## 8. Advanced Concepts

### Q28. Helm

```bash
#!/bin/bash
# Helm basics
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm search repo nginx
helm show values bitnami/nginx > values.yaml  # See all options

# Install
helm install my-nginx bitnami/nginx \
  --namespace production \
  --create-namespace \
  --set service.type=LoadBalancer \
  --values custom-values.yaml

# Upgrade
helm upgrade my-nginx bitnami/nginx \
  --atomic \         # Auto-rollback on failure
  --timeout 5m \
  --cleanup-on-fail

# Operations
helm list -A                         # All releases
helm status my-nginx                 # Release status
helm history my-nginx                # Release history
helm rollback my-nginx 2             # Rollback to revision 2
helm uninstall my-nginx              # Delete release
```

---

### Q29. Custom Resource Definitions (CRDs) & Operators

**CRD** — Extends K8s API with custom resource types:
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
```

**Operators** = CRD + Custom Controller. They encode operational knowledge:

| Operator | Manages |
|----------|---------|
| Prometheus Operator | Prometheus instances, alerting rules |
| Cert-Manager | TLS certificates (Let's Encrypt) |
| Strimzi | Apache Kafka clusters |
| CloudNativePG | PostgreSQL clusters |
| ArgoCD | GitOps deployments |

---

### Q30. Deployment Strategies

| Strategy | How | Downtime | Resources | Rollback |
|----------|-----|---------|-----------|---------|
| **Recreate** | Kill all → start new | Yes | 1x | Fast |
| **RollingUpdate** | Replace gradually | No | 1x+surge | Slow |
| **Blue-Green** | Switch traffic between envs | No | 2x | Instant |
| **Canary** | % traffic to new version | No | 1x+canary | Instant |

---

### Q31. GitOps — ArgoCD & Flux

```yaml
# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-production
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/k8s-manifests
    targetRevision: main
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true        # Delete resources removed from Git
      selfHeal: true     # Revert manual kubectl changes
    syncOptions:
    - CreateNamespace=true
```

```bash
#!/bin/bash
argocd app list
argocd app sync my-app-production
argocd app diff my-app-production
argocd app set my-app-production --sync-policy none    # Suspend sync
argocd app set my-app-production --sync-policy automated  # Resume
```

---

### Q32. Service Mesh (Istio)

Istio injects an **Envoy sidecar proxy** into every pod, providing:
- **Traffic management**: canary, weighted routing, circuit breaking, retries
- **Security**: mTLS between all services (zero-trust)
- **Observability**: distributed tracing, metrics, access logs

```yaml
# Canary via Istio VirtualService
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app
  http:
  - route:
    - destination:
        host: my-app-v1
        port:
          number: 8080
      weight: 90          # 90% to v1
    - destination:
        host: my-app-v2
        port:
          number: 8080
      weight: 10          # 10% to v2
```

---

## 9. Monitoring & Observability

### Q33. Production Monitoring Stack

```
Metrics:  Prometheus ──► Grafana (dashboards)
Logs:     Fluentd/Fluent Bit ──► Elasticsearch ──► Kibana  OR  Loki + Grafana
Tracing:  Jaeger / Zipkin
Alerts:   Alertmanager ──► Slack / PagerDuty
```

```bash
#!/bin/bash
# Install kube-prometheus-stack (Prometheus + Grafana + Alertmanager)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.adminPassword=admin123

kubectl get pods -n monitoring
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

```yaml
# Critical Prometheus alerts
groups:
- name: kubernetes-critical
  rules:
  - alert: PodCrashLooping
    expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Pod {{ $labels.pod }} crash looping"

  - alert: NodeMemoryPressure
    expr: kube_node_status_condition{condition="MemoryPressure",status="true"} == 1
    for: 2m
    labels:
      severity: warning

  - alert: PVCPending
    expr: kube_persistentvolumeclaim_status_phase{phase="Pending"} == 1
    for: 5m
    labels:
      severity: warning

  - alert: DeploymentReplicasMismatch
    expr: kube_deployment_spec_replicas != kube_deployment_status_available_replicas
    for: 10m
    labels:
      severity: critical
```

**Key metrics to monitor:**

| Category | Metrics |
|----------|---------|
| Node | CPU%, Memory%, Disk%, Network I/O |
| Pod | CPU throttling, OOMKills, Restart count |
| Cluster | API server latency, etcd health, scheduler queue |
| App | Request rate, Error rate (5xx), P99 latency |

---

## 10. Disaster Recovery

### Q34. etcd Backup & Restore

```bash
#!/bin/bash
# Backup
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%Y%m%d-%H%M%S).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/server.crt \
  --key=/etc/etcd/server.key

# Verify backup
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-20260524.db --write-out=table

# Restore (on ALL control plane nodes)
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-20260524.db \
  --data-dir=/var/lib/etcd-restore \
  --initial-cluster="master-1=https://10.0.0.1:2380" \
  --name master-1

# Point etcd to restored data
mv /var/lib/etcd /var/lib/etcd.old
mv /var/lib/etcd-restore /var/lib/etcd
systemctl restart etcd

# Automate daily backup
echo "0 2 * * * ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-\$(date +\%Y\%m\%d).db" | crontab -
```

---

### Q35. Velero — Cluster Backup

```bash
#!/bin/bash
# Install Velero
velero install \
  --provider aws \
  --bucket my-velero-backup \
  --backup-location-config region=us-east-1 \
  --snapshot-location-config region=us-east-1

# Create backup
velero backup create prod-backup \
  --include-namespaces production,staging

# Schedule daily backup
velero schedule create daily-backup \
  --schedule="0 2 * * *" \
  --include-namespaces production \
  --ttl 720h          # Keep 30 days

# List backups
velero backup get

# Restore
velero restore create --from-backup prod-backup
velero restore describe <restore-name>
```

---

### Q36. Multi-Environment Deployments

```bash
#!/bin/bash
# Option 1: Namespaces (simple)
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace production

# Option 2: Helm per environment
helm install my-app ./chart -f values-dev.yaml -n dev
helm install my-app ./chart -f values-staging.yaml -n staging
helm install my-app ./chart -f values-prod.yaml -n production

# Option 3: Kustomize overlays
# base/          → shared config
# overlays/dev/  → dev patches (replicas: 1, debug logs)
# overlays/prod/ → prod patches (replicas: 10, HPA)
kubectl apply -k overlays/production/
```

**RTO/RPO targets:**

| Tier | RPO | RTO | Strategy |
|------|-----|-----|---------|
| Critical | < 1 min | < 5 min | Active-active multi-region |
| Important | < 1 hr | < 30 min | Active-passive + Velero |
| Normal | < 24 hr | < 2 hr | Daily backup + restore |

---

## 11. Scenario-Based Questions

### Scenario 1: Pod stuck in CrashLoopBackOff

```bash
#!/bin/bash
# Step 1: Get crash details
kubectl describe pod <pod-name> -n <namespace>
# Key: Exit Code, Last State, Events section

# Step 2: Current logs
kubectl logs <pod-name> -n <namespace>

# Step 3: Logs from BEFORE crash (most useful!)
kubectl logs <pod-name> --previous -n <namespace>

# Step 4: All containers
kubectl logs <pod-name> --all-containers=true

# Step 5: Debug without crash
kubectl run debug --image=my-app:latest --command -- sleep 3600
kubectl exec -it debug -- bash
```

**Exit code table:**

| Exit Code | Meaning | Fix |
|-----------|---------|-----|
| `1` | App error / missing config | Check logs |
| `137` | OOMKilled | Increase memory limits |
| `139` | Segfault | App bug |
| `143` | SIGTERM (probe killed it) | Fix liveness probe timing |

---

### Scenario 2: Pod stuck in Pending

```bash
#!/bin/bash
kubectl describe pod <pod-name>
# Focus on Events section at bottom
```

| Event Message | Cause | Fix |
|---------------|-------|-----|
| `Insufficient cpu/memory` | No node has capacity | Add nodes or reduce requests |
| `node had taints that pod didn't tolerate` | Taint mismatch | Add tolerations |
| `didn't match node affinity` | Label mismatch | Fix affinity rules |
| `PVC not bound` | PVC waiting for PV | Check StorageClass |
| `ImagePullBackOff` | Can't pull image | Fix image/credentials |

---

### Scenario 3: New deployment broke production — rollback

```bash
#!/bin/bash
kubectl rollout history deployment/my-app           # Check revisions
kubectl rollout undo deployment/my-app              # Rollback to previous
kubectl rollout undo deployment/my-app --to-revision=3  # Specific revision
kubectl rollout status deployment/my-app            # Monitor rollback
kubectl describe deployment my-app | grep Image     # Verify image
```

---

### Scenario 4: Service not reachable inside cluster

```bash
#!/bin/bash
# Step 1: Check pods running
kubectl get pods -l app=my-app

# Step 2: Check endpoints (empty = selector mismatch!)
kubectl get endpoints my-service

# Step 3: Compare selector vs labels
kubectl describe svc my-service | grep Selector
kubectl get pods --show-labels

# Step 4: Test from another pod
kubectl run test --image=curlimages/curl --rm -it -- \
  curl http://my-service:80

# Step 5: Check NetworkPolicy
kubectl get networkpolicy -n <namespace>
```

---

### Scenario 5: Node is NotReady

```bash
#!/bin/bash
kubectl describe node <node-name>
# Check: MemoryPressure, DiskPressure, PIDPressure, NetworkUnavailable

# SSH into node
ssh <node-ip>
systemctl status kubelet
journalctl -u kubelet -n 100 --no-pager

# If truly broken — drain then fix
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
kubectl cordon <node>
# After repair:
kubectl uncordon <node>
```

---

### Scenario 6: App slow — high latency despite pods running

```bash
#!/bin/bash
kubectl top pods -n <namespace>
kubectl top nodes
kubectl describe hpa my-app-hpa          # Is HPA scaling?
kubectl describe pods | grep -A3 "Last State"  # OOMKilled?
kubectl get endpoints my-service          # All endpoints healthy?
kubectl describe pod <pod> | grep -A6 "Limits"  # CPU throttling?
```

---

### Scenario 7: Debug running pod without modifying it

```bash
#!/bin/bash
# Ephemeral debug container (K8s 1.23+)
kubectl debug -it <pod-name> --image=nicolaka/netshoot --target=app

# Create debug copy of pod
kubectl debug <pod-name> -it \
  --copy-to=debug-pod \
  --image=ubuntu

# Useful debug images
# busybox        — basic tools
# nicolaka/netshoot — full network toolkit (curl, dig, tcpdump, nmap)
# alpine         — lightweight with apk
```

---

### Scenario 8: Zero-downtime deployment setup

```yaml
# Complete zero-downtime configuration
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0    # Never reduce below desired count
      maxSurge: 1          # Allow 1 extra pod during update
  template:
    spec:
      containers:
      - name: app
        readinessProbe:    # Traffic only goes to ready pods
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          failureThreshold: 3
        lifecycle:
          preStop:          # Drain in-flight requests
            exec:
              command: ["/bin/sh", "-c", "sleep 5"]
      terminationGracePeriodSeconds: 30
```

---

### Scenario 9: Blue-Green deployment

```bash
#!/bin/bash
# Deploy green (new version)
kubectl apply -f deployment-green.yaml

# Verify green is healthy
kubectl get pods -l version=green

# Switch traffic (instant!)
kubectl patch service my-service \
  -p '{"spec":{"selector":{"version":"green"}}}'

# Rollback (instant!)
kubectl patch service my-service \
  -p '{"spec":{"selector":{"version":"blue"}}}'

# After validation, delete blue
kubectl delete deployment my-app-blue
```

---

### Scenario 10: Canary deployment

```bash
#!/bin/bash
# Method 1: Replica ratio (simple)
# v1: 9 replicas → 90% traffic
# v2: 1 replica  → 10% traffic
# Both have label: app: my-app
# Service selector: app: my-app

kubectl scale deployment my-app-v2 --replicas=5   # 33%
kubectl scale deployment my-app-v1 --replicas=10  # 67%

# Method 2: Nginx Ingress annotation
kubectl annotate ingress canary-ingress \
  nginx.ingress.kubernetes.io/canary="true" \
  nginx.ingress.kubernetes.io/canary-weight="10"
```

---

### Scenario 11: Migrate workloads — zero downtime

```bash
#!/bin/bash
# Step 1: Taint old nodes
kubectl taint nodes <old-node> migration=true:NoSchedule

# Step 2: Drain one by one (PDB protects availability)
kubectl drain <old-node-1> --ignore-daemonsets --delete-emptydir-data

# Step 3: Verify workloads moved
kubectl get pods -o wide

# Step 4: Repeat for all old nodes

# Step 5: Delete old nodes
kubectl delete node <old-node-1>
```

---

### Scenario 12: Rotate secret without pod restart

```bash
#!/bin/bash
# Update secret
kubectl create secret generic db-secret \
  --from-literal=DB_PASSWORD=newpassword \
  --dry-run=client -o yaml | kubectl apply -f -

# Volume-mounted secrets: auto-update in ~1 min ✅
# Env var secrets: need restart ❌
kubectl rollout restart deployment/my-app

# Best practice: External Secrets Operator + AWS Secrets Manager
# Automatic rotation without pod restart
```

---

### Scenario 13: TLS certificate — expose service to internet

```bash
#!/bin/bash
# Step 1: Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
```

```yaml
# Step 2: ClusterIssuer (Let's Encrypt)
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@company.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
---
# Step 3: Ingress with auto TLS
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - api.mycompany.com
    secretName: api-tls-cert    # cert-manager fills this automatically
```

---

### Scenario 14: Multi-team cluster isolation

```bash
#!/bin/bash
# 1. Separate namespaces
kubectl create namespace team-alpha
kubectl create namespace team-beta

# 2. ResourceQuota per team
# 3. LimitRange per namespace
# 4. RBAC — teams only access own namespace
kubectl create rolebinding alpha-admin \
  --clusterrole=admin \
  --group=team-alpha \
  --namespace=team-alpha

# 5. NetworkPolicy — deny cross-namespace
```

```yaml
# Deny all cross-namespace traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-cross-namespace
  namespace: team-alpha
spec:
  podSelector: {}
  ingress:
  - from:
    - podSelector: {}     # Only same-namespace pods
```

---

### Scenario 15: Schedule pod only on GPU nodes

```bash
#!/bin/bash
# Label GPU nodes
kubectl label nodes gpu-node-1 accelerator=nvidia-gpu
kubectl label nodes gpu-node-2 accelerator=nvidia-gpu

# Taint to ensure only GPU workloads use GPU nodes
kubectl taint nodes gpu-node-1 nvidia.com/gpu=true:NoSchedule
```

```yaml
spec:
  tolerations:
  - key: "nvidia.com/gpu"
    operator: "Exists"
    effect: "NoSchedule"
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator
            operator: In
            values:
            - nvidia-gpu
  containers:
  - name: ml-training
    image: tensorflow:latest
    resources:
      limits:
        nvidia.com/gpu: 1   # Request 1 GPU
```

---

### Scenario 16: etcd running out of disk space

```bash
#!/bin/bash
# Check etcd size
ETCDCTL_API=3 etcdctl endpoint status --write-out=table

# Compact old revisions
ETCDCTL_API=3 etcdctl compact \
  $(ETCDCTL_API=3 etcdctl endpoint status --write-out=json | \
    jq '.[0].Status.header.revision')

# Defragment to reclaim space
ETCDCTL_API=3 etcdctl defrag --endpoints=https://127.0.0.1:2379

# Increase quota in etcd config (default 2GB)
# --quota-backend-bytes=8589934592   # 8GB
```

---

### Scenario 17: Namespace ResourceQuota — 4 CPUs, 8GB RAM

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: team-alpha
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "20"
---
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: team-alpha
spec:
  limits:
  - default:
      cpu: 500m
      memory: 256Mi
    defaultRequest:
      cpu: 250m
      memory: 128Mi
    type: Container
```

---

### Scenario 18: ImagePullBackOff

```bash
#!/bin/bash
kubectl describe pod <pod-name>
# Look for: "Failed to pull image..."

# Fix 1: Wrong image name/tag
kubectl set image deployment/my-app app=myrepo/myimage:v2.1.0

# Fix 2: Private registry — create pull secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.company.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=me@company.com
```

```yaml
# Reference in deployment
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: app
    image: registry.company.com/myapp:v2
```

---

### Scenario 19: Cluster certificate expiring soon

```bash
#!/bin/bash
kubeadm certs check-expiration

kubeadm certs renew all

# Restart control plane static pods
kubectl -n kube-system delete pod -l component=kube-apiserver
kubectl -n kube-system delete pod -l component=kube-controller-manager
kubectl -n kube-system delete pod -l component=kube-scheduler

# Update kubeconfig
cp /etc/kubernetes/admin.conf ~/.kube/config
kubectl get nodes    # Verify
```

---

### Scenario 20: Force delete pod stuck in Terminating

```bash
#!/bin/bash
# Try graceful first
kubectl delete pod <pod-name> --grace-period=0

# Force delete
kubectl delete pod <pod-name> --grace-period=0 --force

# If node is down — remove finalizers
kubectl patch pod <pod-name> -p \
  '{"metadata":{"finalizers":[]}}' --type=merge

# Check if node is down
kubectl get node $(kubectl get pod <pod-name> -o jsonpath='{.spec.nodeName}')
```

---

### Scenario 21: Production pods OOMKilled every 2 hours

```bash
#!/bin/bash
kubectl describe pod <pod> | grep -A 5 "Limits"
kubectl top pod <pod>
kubectl get cronjobs    # Batch job at 14:00 consuming memory?

# Fix: Increase memory limits
kubectl patch deployment my-app --patch '
{"spec":{"template":{"spec":{"containers":[{
  "name":"app","resources":{"limits":{"memory":"1Gi"}}
}]}}}}'

# Monitor with Prometheus
# container_memory_usage_bytes{pod="my-pod"}
```

---

### Scenario 22: Secret accidentally exposed in pod logs

```bash
#!/bin/bash
# 1. IMMEDIATELY rotate the credential in the DB/service
# 2. Update K8s secret
kubectl create secret generic db-secret \
  --from-literal=password=newpassword \
  --dry-run=client -o yaml | kubectl apply -f -
kubectl rollout restart deployment/my-app

# 3. Audit access
kubectl auth can-i get secret db-secret --as=developer@company.com

# 4. Fix code — never log secrets
# 5. Prevent: OPA/Gatekeeper policy, LOG_LEVEL != DEBUG in prod
```

---

### Scenario 23: etcd down — control plane not responding

```bash
#!/bin/bash
ssh <master-node>
systemctl status etcd
journalctl -u etcd -n 100 --no-pager
ls -la /var/lib/etcd/

# Restore from backup
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-latest.db \
  --data-dir=/var/lib/etcd-restore
mv /var/lib/etcd /var/lib/etcd.old
mv /var/lib/etcd-restore /var/lib/etcd
systemctl start etcd

ETCDCTL_API=3 etcdctl member list
kubectl get nodes    # Should work now
```

---

### Scenario 24: HPA not scaling during traffic spikes

```bash
#!/bin/bash
kubectl describe hpa <hpa-name>
# Look for: "unable to get metrics" error

# Check metrics-server
kubectl get pods -n kube-system | grep metrics-server
kubectl top pods    # Should work

# Install if missing
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Ensure resource requests set on deployment
kubectl describe deploy my-app | grep -A 5 Requests
```

---

### Scenario 25: Cluster upgrade — zero downtime

```bash
#!/bin/bash
# Pre-upgrade
ETCDCTL_API=3 etcdctl snapshot save backup.db
kubeadm upgrade plan    # Check upgrade path

# Upgrade control plane
kubectl cordon <master-node>
sudo kubeadm upgrade apply v1.29.0
sudo systemctl restart kubelet
kubectl uncordon <master-node>

# Upgrade workers one by one
kubectl cordon <worker-node>
kubectl drain <worker-node> --ignore-daemonsets --delete-emptydir-data
# Upgrade OS + kubeadm + kubelet
sudo kubeadm upgrade node
sudo systemctl restart kubelet
kubectl uncordon <worker-node>
kubectl get nodes    # Verify version
```

---

### Scenario 26: ConfigMap changes not reflected

```bash
#!/bin/bash
# Check ConfigMap updated
kubectl get configmap <cm-name> -o yaml

# For env vars → need restart
kubectl rollout restart deployment/<name>

# For volume mounts → waits ~1 min, then auto-updates
kubectl exec -it <pod> -- cat /etc/config/my-key

# Auto-restart with Reloader
helm install reloader stakater/reloader
kubectl annotate deployment my-app \
  configmap.reloader.stakater.com/reload="my-configmap"
```

---

### Scenario 27: Liveness probe killing healthy pods under load

```bash
#!/bin/bash
# Problem: probe times out under load → K8s kills healthy pod
# Fix: Make probe more tolerant
kubectl patch deployment my-app --patch '
{"spec":{"template":{"spec":{"containers":[{
  "name":"app",
  "livenessProbe":{
    "httpGet":{"path":"/healthz","port":8080},
    "initialDelaySeconds":30,
    "periodSeconds":10,
    "timeoutSeconds":5,
    "failureThreshold":3
  }
}]}}}}'
```

**Rule:** Liveness = lenient (is app alive?). Readiness = strict (can app serve traffic?).

---

### Scenario 28: Pod on wrong node — GDPR compliance

```bash
#!/bin/bash
# Label EU nodes
kubectl label node eu-node-1 region=eu
kubectl label node eu-node-2 region=eu

# Add nodeSelector to deployment
kubectl patch deployment eu-app --patch '
{"spec":{"template":{"spec":{"nodeSelector":{"region":"eu"}}}}}'

# Verify
kubectl get pods -o wide | grep eu-app

# Delete old pod on US node
kubectl delete pod <pod-on-us-node>
```

---

### Scenario 29: Namespace deleted accidentally

```bash
#!/bin/bash
# Restore from Velero backup
velero restore create --from-backup daily-backup \
  --include-namespaces production

# If no backup — apply from Git repo
kubectl apply -f k8s/production/ -R

# Prevention: Add finalizers
kubectl annotate namespace production \
  "hnc.x-k8s.io/managed-by=true"
```

---

### Scenario 30: Istio sidecar breaking all pods

```bash
#!/bin/bash
kubectl describe pod <pod> | grep istio
kubectl logs <pod> -c istio-proxy

# Temporarily disable injection
kubectl label namespace production istio-injection=disabled
kubectl rollout restart deployment/my-app -n production

# Check mTLS policy
kubectl get peerauthentication -A
kubectl get destinationrule -A
```

---

### Scenario 31: Disk pressure — pods evicted

```bash
#!/bin/bash
kubectl describe node <node> | grep -A 5 "Conditions"
# DiskPressure: True

ssh <node>
df -h
docker image prune -a --force     # Remove unused images
docker system prune --force
find /var/log/pods -name "*.log" -size +100M -delete

# Long-term: configure log rotation
# /etc/docker/daemon.json:
# {"log-opts": {"max-size": "100m", "max-file": "3"}}
```

---

### Scenario 32: Teams fighting for cluster resources

```bash
#!/bin/bash
kubectl create namespace frontend-team
kubectl create namespace backend-team

# Apply ResourceQuota per namespace
# Apply LimitRange defaults per namespace

kubectl create rolebinding frontend-admin \
  --clusterrole=admin \
  --group=frontend-team \
  --namespace=frontend-team

kubectl create rolebinding backend-admin \
  --clusterrole=admin \
  --group=backend-team \
  --namespace=backend-team
```

---

### Scenario 33: StatefulSet pods — no stable names

```bash
#!/bin/bash
kubectl describe statefulset my-db

# Need headless service!
kubectl get service my-db-headless
# If missing: create with clusterIP: None
```

```yaml
# Required headless service
apiVersion: v1
kind: Service
metadata:
  name: my-db-headless
spec:
  clusterIP: None    # This makes it headless
  selector:
    app: my-db
  ports:
  - port: 5432
# Pod DNS: my-db-0.my-db-headless.namespace.svc.cluster.local
```

---

### Scenario 34: Security breach — unauthorized pod in kube-system

```bash
#!/bin/bash
# 1. Capture evidence FIRST
kubectl get pod <suspicious-pod> -o yaml > evidence.yaml
kubectl logs <suspicious-pod> > pod-logs.txt

# 2. Isolate (block network without deleting)
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: isolate-suspicious
  namespace: kube-system
spec:
  podSelector:
    matchLabels:
      <pod-label>: "true"
  policyTypes:
  - Ingress
  - Egress
EOF

# 3. Audit
kubectl get events -n kube-system --sort-by='.lastTimestamp'
kubectl get rolebindings,clusterrolebindings -A | grep suspicious
grep "suspicious-pod" /var/log/kube-apiserver.log

# 4. Remove + cleanup
kubectl delete pod <suspicious-pod> -n kube-system --force
kubectl delete clusterrolebinding <compromised-binding>
```

---

### Scenario 35: Job running forever

```bash
#!/bin/bash
kubectl describe job my-data-job
kubectl delete job my-data-job
```

```yaml
# Fix: Add timeout
apiVersion: batch/v1
kind: Job
metadata:
  name: my-data-job
spec:
  activeDeadlineSeconds: 1800   # 30 min max
  backoffLimit: 3
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
```

---

### Scenario 36–65: Additional Quick Scenarios

**36. Helm upgrade failed:**
```bash
helm status my-release -n production   # Check failed state
helm history my-release -n production
helm rollback my-release 3 -n production
# Prevention: helm upgrade --atomic --timeout 10m
```

**37. Pods in different namespaces can't communicate:**
```bash
# By default they CAN communicate. Problem = NetworkPolicy
kubectl get networkpolicies -n api
# Add: allow-from-web-namespace NetworkPolicy in api namespace
```

**38. Running out of Pod IPs:**
```bash
kubectl delete pods --field-selector status.phase=Succeeded -A
kubectl delete pods --field-selector status.phase=Failed -A
# Add nodes = new CIDR blocks = more IPs
```

**39. API server slow — 500+ nodes:**
```bash
ETCDCTL_API=3 etcdctl endpoint health
ETCDCTL_API=3 etcdctl check perf
ETCDCTL_API=3 etcdctl defrag
kubectl get flowschemas
kubectl get prioritylevelconfigurations
```

**40. Application unreachable — pods Running but READY 0/1:**
```bash
kubectl get pods    # READY column = 0/1 = readiness probe failing
kubectl describe pod <pod> | grep -A 10 "Conditions"
kubectl exec -it <pod> -- curl localhost:8080/ready
```

**41. Namespace Terminating — stuck:**
```bash
kubectl get namespace <ns> -o json | \
  jq '.spec.finalizers = []' | \
  kubectl replace --raw "/api/v1/namespaces/<ns>/finalize" -f -
```

**42. CRD broke after K8s upgrade:**
```bash
pluto detect-all-in-cluster    # Find deprecated APIs before upgrade!
kubectl api-versions | grep <group>
helm upgrade my-operator --version 2.0.0
```

**43. Multiple pods crashing at exactly same time:**
```bash
# Check CronJob running at that time
kubectl get cronjobs
kubectl get events --sort-by='.lastTimestamp'
# Check shared dependency (DB, Redis, config)
kubectl logs <pod> | grep -i "connection\|timeout"
```

**44. Deployment has no resource limits — node crashed:**
```bash
kubectl set resources deployment/runaway-app \
  --limits=cpu=500m,memory=512Mi \
  --requests=cpu=100m,memory=128Mi
kubectl uncordon <crashed-node>
# Prevention: LimitRange in namespace
```

**45. DNS resolution slow:**
```bash
kubectl run dns-test --image=busybox --rm -it -- \
  sh -c "time nslookup my-service.production.svc.cluster.local"
kubectl scale deployment coredns -n kube-system --replicas=4
# Add ndots: 2 to pod dnsConfig to reduce unnecessary lookups
```

**46. GitOps (ArgoCD) reverting manual changes:**
```bash
# Correct behavior! Fix in Git, not kubectl
git commit -m "emergency: fix config"
git push origin main
argocd app sync my-app
# For emergency: argocd app set my-app --sync-policy none
```

**47. Cluster autoscaler not adding nodes:**
```bash
kubectl get pods -n kube-system | grep cluster-autoscaler
kubectl logs -n kube-system <autoscaler-pod> | grep -i "scale\|error"
# Check cloud ASG/MIG max size
# Check pod PDB preventing scale-down (might be blocking)
```

**48. Container running as root — security audit:**
```bash
kubectl patch deployment my-app --patch '
{"spec":{"template":{"spec":{"securityContext":{
  "runAsUser":1000,"runAsNonRoot":true,"fsGroup":1000
}}}}}'
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted
```

**49. K8s upgrade broke LoadBalancer services:**
```bash
kubectl describe service my-lb-service
kubectl logs -n kube-system -l app=cloud-controller-manager
helm upgrade aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system --set clusterName=my-cluster
```

**50. After CNI change — internal services unreachable:**
```bash
kubectl get networkpolicies -A
# Check pod CIDR doesn't overlap service CIDR
kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'
# Restart CNI daemonset
kubectl rollout restart daemonset calico-node -n kube-system
```

**51. Black Friday — latency 10x spike:**
```bash
kubectl patch hpa my-app-hpa --patch '{"spec":{"maxReplicas":50}}'
kubectl scale deployment my-app --replicas=20  # Pre-scale
kubectl annotate ingress my-ingress \
  nginx.ingress.kubernetes.io/limit-rps="200"
```

**52. Sidecar crashing — main pod restarts:**
```bash
kubectl get pods -o json | \
  jq '.items[].status.containerStatuses[] | {name, restartCount}'
kubectl logs <pod> -c logging-sidecar --previous
kubectl set image deployment/my-app \
  logging-sidecar=fluent/fluent-bit:2.1
```

**53. Pods evicted during cluster upgrade:**
```yaml
# Ensure PDB before upgrade!
apiVersion: policy/v1
kind: PodDisruptionBudget
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: critical-app
```

**54. Init container taking too long:**
```bash
kubectl logs <pod> -c <init-container-name>
kubectl describe pod <pod> | grep -A 5 "Init Containers"
# Common: waiting for dependency that's not ready
# Fix: Add timeout to init container command
# 'until nc -z db-service 5432; do sleep 2; done; timeout=300'
```

**55. HPA scaling up but pods Pending — no nodes:**
```bash
kubectl describe pod <pending-pod> | grep -A 5 Events
# "0/3 nodes are available: 3 Insufficient cpu"
kubectl describe nodes | grep -A 5 "Allocated resources"
kubectl get pods -n kube-system | grep cluster-autoscaler
# Check cloud ASG max size limit!
```

**56. Deployment rollout stuck — minReadySeconds:**
```bash
kubectl describe deployment my-app | grep -E "minReadySeconds|Strategy"
kubectl patch deployment my-app --patch '{"spec":{"minReadySeconds":10}}'
kubectl rollout resume deployment/my-app
```

**57. Developer applied to wrong cluster:**
```bash
kubectl config current-context    # ALWAYS check before applying!
kubectl rollout undo deployment/my-app -n production
kubectl create rolebinding dev-view \
  --clusterrole=view \
  --user=developer@company.com \
  --namespace=production
```

**58. Pod logs show connection refused to DB:**
```bash
kubectl exec -it <pod> -- nc -zv db-service 5432  # TCP test
kubectl get endpoints db-service                   # DB pods healthy?
kubectl logs <pod> | grep -i "connection\|retry"
# Common: DB pod restarting, PVC issue, secret wrong password
```

**59. Prometheus not scraping metrics:**
```bash
kubectl get servicemonitor -A              # Check ServiceMonitors
kubectl describe servicemonitor my-app
kubectl logs -n monitoring <prometheus-pod> | grep "error\|fail"
# Check: port name matches, labels match, RBAC for Prometheus SA
```

**60. Rollback not restoring stability:**
```bash
# Problem NOT in app code — check external causes
kubectl describe configmap <app-config>  # Config changed?
kubectl get secret <app-secret> -o yaml  # Secret changed?
kubectl get networkpolicies              # New policy blocking?
kubectl describe pvc                     # Storage corrupted?
# DB migration ran → schema change is irreversible!
```

**61. StatefulSet upgrade — ordered vs parallel:**
```yaml
# Default: ordered (0→1→2, then 2→1→0 for rollout)
# For faster updates:
updateStrategy:
  type: RollingUpdate
  rollingUpdate:
    partition: 0     # Update all pods
    # partition: 2   # Only update pods 2+ (canary-style)
```

**62. Kernel OOM on node despite K8s limits:**
```bash
# K8s limits != Linux cgroups always match exactly
# Check: system reserved + kube reserved configured?
kubectl describe node <node> | grep -i "allocated\|capacity"
# Fix: Set --system-reserved and --kube-reserved in kubelet config
```

**63. CronJob creating too many failed jobs:**
```yaml
spec:
  schedule: "*/5 * * * *"
  failedJobsHistoryLimit: 3     # Keep only 3 failed jobs
  successfulJobsHistoryLimit: 1
  concurrencyPolicy: Replace    # Replace if previous still running
```

**64. Pod spec not updating — Deployment not changing:**
```bash
# Force recreation even without spec change
kubectl rollout restart deployment/my-app
# OR add annotation to trigger update:
kubectl annotate deployment my-app \
  kubectl.kubernetes.io/restartedAt="$(date -u +%Y-%m-%dT%H:%M:%SZ)"
```

**65. Production alert — pod restart count > 10/hour:**
```bash
kubectl describe pod <auth-pod> -n production
kubectl logs <auth-pod> --previous -n production | tail -50
kubectl top pod <auth-pod> -n production
kubectl exec -it <auth-pod> -- nc -zv db-service 5432
# Quick mitigation: increase liveness probe failureThreshold
# If recent deploy caused it: kubectl rollout undo deployment/auth-service
```

---

## 12. HCL — Terraform + Kubernetes

```hcl
# ============================================
# EKS cluster via Terraform
# Language: HCL
# ============================================

terraform {
  required_version = ">= 1.5.0"
  required_providers {
    aws        = { source = "hashicorp/aws",        version = "~> 5.0" }
    kubernetes = { source = "hashicorp/kubernetes", version = "~> 2.0" }
  }
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "eks/terraform.tfstate"
    region         = "eu-north-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = var.cluster_name
  cluster_version = "1.29"
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnets

  eks_managed_node_groups = {
    production = {
      instance_types = ["t3.medium"]
      min_size       = 2
      max_size       = 10
      desired_size   = 3
      labels         = { environment = "production" }
    }
    spot = {
      instance_types = ["t3.medium", "t3.large"]
      capacity_type  = "SPOT"
      min_size       = 0
      max_size       = 5
      desired_size   = 2
      labels         = { environment = "production", type = "spot" }
      taints = [{
        key    = "spot"
        value  = "true"
        effect = "NO_SCHEDULE"
      }]
    }
  }

  cluster_addons = {
    coredns    = { most_recent = true }
    kube-proxy = { most_recent = true }
    vpc-cni    = { most_recent = true }
    aws-ebs-csi-driver = { most_recent = true }
  }

  tags = { Environment = "production", Terraform = "true" }
}
```

```hcl
# ============================================
# K8s resources via Terraform
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

resource "kubernetes_resource_quota" "production" {
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
  metadata { name = "cluster-readonly" }
  rule {
    api_groups = [""]
    resources  = ["pods", "services", "configmaps", "namespaces"]
    verbs      = ["get", "list", "watch"]
  }
  rule {
    api_groups = ["apps"]
    resources  = ["deployments", "replicasets", "statefulsets"]
    verbs      = ["get", "list", "watch"]
  }
}
```

---

## 13. Quick Commands Reference

```bash
#!/bin/bash
# ============================================
# KUBERNETES COMPLETE COMMANDS CHEATSHEET
# Language: Shell (Bash)
# ============================================

# === GETTING INFO ===
kubectl get pods -A                               # All namespaces
kubectl get pods -o wide                          # With node info
kubectl get all -n production                     # All resources in ns
kubectl get events --sort-by='.lastTimestamp'     # Recent events
kubectl api-resources                             # All API resources

# === POD DEBUGGING ===
kubectl describe pod <pod>                        # Full details + events
kubectl logs <pod>                                # Current logs
kubectl logs <pod> --previous                     # Last crash logs
kubectl logs <pod> -f                             # Follow live
kubectl logs <pod> -c <container>                 # Specific container
kubectl exec -it <pod> -- /bin/sh                # Shell into pod
kubectl exec -it <pod> -c <container> -- bash    # Specific container
kubectl top pod <pod> --containers               # Per-container usage
kubectl debug -it <pod> --image=busybox          # Ephemeral debug

# === DEPLOYMENT ===
kubectl apply -f manifest.yaml                    # Create/update
kubectl delete -f manifest.yaml                   # Delete
kubectl diff -f manifest.yaml                     # Preview changes
kubectl rollout status deployment/<name>          # Watch rollout
kubectl rollout history deployment/<name>         # Revision history
kubectl rollout undo deployment/<name>            # Rollback
kubectl rollout pause deployment/<name>           # Pause
kubectl rollout resume deployment/<name>          # Resume
kubectl scale deployment/<name> --replicas=5      # Scale
kubectl set image deployment/<name> app=img:v2   # Update image

# === NODE MANAGEMENT ===
kubectl get nodes                                 # Node status
kubectl describe node <node>                      # Full details
kubectl cordon <node>                             # Stop scheduling
kubectl drain <node> --ignore-daemonsets         # Evict pods
kubectl uncordon <node>                          # Resume
kubectl top nodes                                 # Resource usage
kubectl taint nodes <node> key=val:NoSchedule    # Add taint
kubectl taint nodes <node> key-                  # Remove taint

# === NETWORKING ===
kubectl get service -A                            # All services
kubectl get endpoints <service>                   # Service targets
kubectl get networkpolicies -A                    # All policies
kubectl get ingress -A                            # All ingresses
kubectl exec <pod> -- nslookup <service>         # DNS test
kubectl exec <pod> -- curl http://<service>      # HTTP test
kubectl port-forward pod/<pod> 8080:8080         # Local port forward
kubectl port-forward svc/<service> 8080:80       # Service port forward

# === STORAGE ===
kubectl get pvc -A                                # All PVCs
kubectl describe pvc <name>                       # Why pending?
kubectl get pv                                    # All PVs
kubectl get storageclass                          # Storage classes

# === SECURITY ===
kubectl auth can-i <verb> <resource>             # Check permission
kubectl auth can-i --list                        # All my permissions
kubectl get secrets -A                           # List secrets
kubectl get serviceaccounts -A                   # Service accounts
kubectl get rolebindings -A                      # All role bindings

# === CLUSTER ===
kubectl cluster-info                              # Cluster details
kubectl version                                   # K8s version
kubectl top pods --all-namespaces                 # All pod usage
kubectl get componentstatus                       # Control plane health
kubeadm certs check-expiration                   # Check cert expiry
kubeadm certs renew all                          # Renew certs

# === HELM ===
helm repo add <name> <url>                       # Add repo
helm repo update                                  # Update repos
helm install <release> <chart> -n <ns>          # Install
helm upgrade <release> <chart> --atomic          # Upgrade safely
helm rollback <release> <revision>               # Rollback
helm list -A                                      # All releases
helm history <release>                            # Release history
helm uninstall <release>                          # Delete

# === ETCD ===
ETCDCTL_API=3 etcdctl snapshot save backup.db    # Backup
ETCDCTL_API=3 etcdctl snapshot status backup.db  # Verify
ETCDCTL_API=3 etcdctl endpoint health            # Health
ETCDCTL_API=3 etcdctl defrag                     # Defragment
ETCDCTL_API=3 etcdctl member list                # Members

# === VELERO ===
velero backup create my-backup --include-namespaces prod
velero restore create --from-backup my-backup
velero schedule create daily --schedule="0 2 * * *"
velero backup get                                 # List backups

# === ARGOCD ===
argocd app list                                   # All apps
argocd app sync <app>                             # Sync from Git
argocd app diff <app>                             # Show diff
argocd app rollback <app> <revision>             # Rollback
argocd app set <app> --sync-policy none          # Suspend sync

# === DEBUG SHORTCUTS ===
kubectl run debug --image=busybox --rm -it -- /bin/sh
kubectl run netshoot --image=nicolaka/netshoot --rm -it -- bash
kubectl get pods --field-selector status.phase=Failed -A
kubectl delete pods --field-selector status.phase=Succeeded -A
```

---

## 📊 Architecture & Troubleshooting Diagrams

### Complete K8s Architecture
```
Internet
    │
CloudFront/ALB
    │
┌───▼─────────────────────────────────────────┐
│           Ingress Controller                │  (Public Subnet)
│           (nginx / AWS ALB)                 │
└───────────────────┬─────────────────────────┘
                    │
┌───────────────────▼─────────────────────────┐
│              Service (ClusterIP)            │
│         selector: app=my-app               │
└─────┬──────────┬───────────┬───────────────┘
      │          │           │
   Pod-0      Pod-1       Pod-2              (Private Subnet)
(app + sidecar) (app) (app)
      │
      ▼
ConfigMap / Secret / PVC
      │
      ▼
RDS / Redis / External Services            (Private Subnet)
```

### Pod Debug Flow
```
kubectl get pods
       │
   STATUS?
   ┌────┴──────────────────────────────────┐
Pending                            CrashLoopBackOff
   │                                        │
   ├── Insufficient cpu/memory         kubectl logs --previous
   ├── Taint not tolerated                   │
   ├── Node affinity mismatch         Exit Code?
   ├── PVC not bound                0=exits  137=OOM
   └── ImagePullBackOff             1=error  139=segfault
```

### Networking Debug Flow
```
kubectl exec pod -- curl service:port
              │
          FAILS?
          │
   Check endpoints → Empty? → Label/selector mismatch
          │
   Check NetworkPolicy → Blocking? → Add allow rule
          │
   Check CoreDNS → Down? → kubectl rollout restart coredns -n kube-system
          │
   Check CNI plugin → Issue? → kubectl rollout restart daemonset <cni> -n kube-system
          │
   Check node firewall → Blocking? → Open overlay CNI ports
```

### Storage Debug Flow
```
PVC stuck in Pending
        │
kubectl describe pvc → Events
        │
StorageClass exists?  → No  → Create StorageClass
        │
Provisioner running?  → No  → Fix provisioner pod
        │
AccessMode correct?   → No  → Fix accessModes (RWO vs RWX)
        │
Zone/node available?  → No  → Add storage / node
        │
PVC bound → Pod still failing?
        │
Volume mounted?  kubectl exec pod -- df -h
        │
Permissions?     kubectl exec pod -- ls -la /data
```

---
s · Shell + YAML + HCL*
MDEOF
echo "Done! Lines: $(wc -l < /mnt/user-data/outputs/Kubernetes-Complete-Guide.md)"
