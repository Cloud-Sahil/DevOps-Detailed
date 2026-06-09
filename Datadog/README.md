# <img src="https://cdn.simpleicons.org/datadog" height="40" /> Datadog

> Monitoring & Observability Platform | DevOps | SRE | Cloud

---

## 📌 What is Datadog?

**Datadog** is a cloud-based **SaaS monitoring and observability platform** that gives you full visibility into your entire infrastructure — servers, containers, applications, databases, cloud services, and more — all from a single unified dashboard.

> Think of Datadog as your **all-seeing control room** — whether your app runs on one EC2 server or 10,000 Kubernetes pods, Datadog watches everything in real time and alerts you before users even notice a problem.

### 🏢 About the Company
| Detail | Info |
|--------|------|
| Founded | 2010 |
| Headquarters | New York, USA |
| Type | SaaS (Software as a Service) |
| Founded by | Olivier Pomel & Alexis Lê-Quôc |
| Nasdaq | DDOG |
| Customers | Netflix, Samsung, Airbnb, Peloton, 26,000+ companies |

---

## 🤔 Why Datadog? — The Problem it Solves

### Before Datadog (The Old Way)
```
Problem:
  EC2 server slow?  → SSH into server → run top → check logs → grep errors
  Pod crashing?     → kubectl logs pod1 → kubectl logs pod2 → no easy correlation
  API latency high? → No idea which microservice is the bottleneck

Tools needed:
  Nagios for infra monitoring   (separate)
  ELK Stack for logs            (separate)
  Jaeger for tracing            (separate)
  Grafana for dashboards        (separate)
  PagerDuty for alerting        (separate)
  = 5 different tools to manage!
```

### After Datadog (The New Way)
```
One Platform:
  ✅ Metrics     → Infrastructure monitoring
  ✅ Logs        → Log management & analytics
  ✅ Traces      → APM (distributed tracing)
  ✅ Alerts      → Built-in alerting → Slack/PagerDuty
  ✅ Dashboards  → All in one place

Result:
  MTTR (Mean Time to Resolve) drops from hours → minutes
```

---

## 🏗️ Datadog Architecture — Deep Dive

```
Your Infrastructure (EC2 / K8s / Docker / On-premise)
        │
        ▼
┌───────────────────────────────────────────────────┐
│              Datadog Agent (installed on host)    │
│   ┌──────────┐ ┌─────────┐ ┌───────────────────┐ │
│   │ Metrics  │ │  Logs   │ │ Traces (APM)      │ │
│   │ Collector│ │Forwarder│ │ (DogStatsD/OTLP)  │ │
│   └────┬─────┘ └────┬────┘ └────────┬──────────┘ │
└────────┼────────────┼───────────────┼─────────────┘
         │            │               │
         ▼            ▼               ▼
┌─────────────────────────────────────────────────────┐
│               Datadog Cloud Platform                │
│                                                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────────┐│
│  │Infrastructure│ │Log Mgmt │ │  APM / Tracing     ││
│  │Monitoring  │ │         │ │                      ││
│  └─────┬─────┘ └────┬─────┘ └──────────┬───────────┘│
│        │            │                  │             │
│        └────────────┼──────────────────┘             │
│                     ▼                                │
│  ┌──────────────────────────────────────────────┐   │
│  │           Unified Dashboard                  │   │
│  │  Metrics ←→ Logs ←→ Traces (correlated!)    │   │
│  └──────────────────────────────────────────────┘   │
│                     │                                │
│                     ▼                                │
│  ┌──────────┐  ┌──────────┐  ┌───────────────────┐  │
│  │ Alerting │  │Analytics │  │ Notebooks / SLOs  │  │
│  └──────────┘  └──────────┘  └───────────────────┘  │
└─────────────────────────────────────────────────────┘
         │
         ▼
Notifications: Email | Slack | PagerDuty | Teams | Webhook
```

---

## 🔑 Core Concepts

| Concept | Description |
|---------|-------------|
| **Agent** | Lightweight software installed on your server/node to collect data |
| **Metric** | Numerical time-series data (CPU%, request count, latency) |
| **Log** | Text-based event records from your app/system |
| **Trace** | End-to-end path of a single request through microservices |
| **Span** | A single operation within a trace (one service's work) |
| **Dashboard** | Visual panels showing metrics, logs, and traces |
| **Monitor** | A configured alert condition (trigger when CPU > 90%) |
| **Integration** | Connector to external service (AWS, Kubernetes, Nginx) |
| **Tag** | Key:value label for filtering (`env:prod`, `service:api`) |
| **DogStatsD** | UDP protocol for sending custom metrics to agent |
| **SLO** | Service Level Objective — reliability target (99.9% uptime) |
| **Notebook** | Collaborative investigation document in Datadog |

---

## 📦 Datadog Product Suite

### 1. 📊 Infrastructure Monitoring
Monitors physical/virtual servers and cloud resources:

```
What it tracks:
  ├── CPU utilization
  ├── Memory usage (RAM)
  ├── Disk I/O and space
  ├── Network in/out
  ├── Process health
  └── System load average

Data Sources:
  ├── Datadog Agent (on each server)
  ├── AWS CloudWatch integration
  ├── Azure Monitor integration
  └── GCP Stackdriver integration

Key Views:
  ├── Infrastructure List  → All hosts at a glance
  ├── Host Map            → Color-coded health grid
  └── Container Map       → All containers visualized
```

---

### 2. 📈 Application Performance Monitoring (APM)
Distributed tracing for microservices:

```
Request Flow Example:
  User → API Gateway → Auth Service → Product Service → DB

Trace shows:
  Total: 450ms
  ├── API Gateway:      5ms
  ├── Auth Service:    20ms   ← slow? check here
  ├── Product Service: 400ms  ← BOTTLENECK FOUND!
  └── Database query:  25ms

Without APM: "The app is slow" (no idea why)
With APM:    "Product Service line 234 SQL query is 400ms" ✅
```

APM tracks:
- Request latency (p50, p95, p99)
- Error rates per service
- Throughput (requests/second)
- Dependency maps between services
- Flame graphs for code-level profiling

---

### 3. 📝 Log Management
Centralized log collection and analysis:

```
Sources → Datadog Agent/Forwarder → Datadog Logs
  ├── Application logs (stdout/stderr)
  ├── System logs (/var/log/syslog)
  ├── Nginx/Apache access logs
  ├── Kubernetes pod logs
  └── AWS CloudWatch Logs

Features:
  ├── Live Tail          → Real-time log stream
  ├── Log Explorer       → Search and filter
  ├── Log Analytics      → Count, group, visualize
  ├── Log Pipelines      → Parse and enrich logs
  └── Log Archives       → Store to S3 long-term
```

---

### 4. 🐳 Container & Kubernetes Monitoring
Deep visibility into containerized workloads:

```
Kubernetes Monitoring:
  Cluster Level:
    ├── Node health & resources
    ├── Namespace resource usage
    └── Cluster capacity planning

  Workload Level:
    ├── Deployment health
    ├── Pod restarts & crashes
    ├── HPA scaling events
    └── Job/CronJob status

  Container Level:
    ├── Container CPU/Memory
    ├── OOMKilled events
    └── Container restarts

Key Kubernetes Metrics:
  kubernetes.cpu.usage.total
  kubernetes.memory.working_set
  kubernetes.pods.running
  kubernetes.containers.restarts
```

---

### 5. ☁️ Cloud Monitoring (AWS / Azure / GCP)

```
AWS Integration covers:
  ├── EC2         → Instance metrics
  ├── RDS         → DB performance
  ├── Lambda      → Invocations, errors, duration
  ├── ALB/NLB     → Request counts, latency
  ├── EKS         → Cluster metrics
  ├── S3          → Bucket metrics
  ├── SQS         → Queue depth
  └── CloudFront  → CDN metrics

Setup: IAM Role → Datadog AWS Integration → Done (no agent needed!)
```

---

### 6. 🔔 Alerting & Monitors

Monitor types:
```
1. Threshold Monitor    → Alert when metric > threshold
   Example: CPU > 90% for 5 minutes → alert!

2. Anomaly Monitor      → ML-based, alert on unusual behavior
   Example: Traffic drops 40% vs same time last week

3. Forecast Monitor     → Predict future issues
   Example: Disk will be full in 4 hours

4. Composite Monitor    → Combine multiple monitors
   Example: Alert when CPU > 90% AND memory > 85%

5. Log Monitor          → Alert on log patterns
   Example: >100 ERROR logs in 5 minutes

6. APM Monitor          → Alert on trace metrics
   Example: p99 latency > 2 seconds

7. SLO Monitor          → Alert when SLO at risk
   Example: Error budget < 10% remaining
```

Notification channels:
```
├── Email
├── Slack (#incidents channel)
├── PagerDuty (on-call rotation)
├── Microsoft Teams
├── OpsGenie
├── Webhook (custom)
└── AWS SNS
```

---

## 🛠️ Installation & Setup

### Install Datadog Agent on Ubuntu EC2

```bash
# Step 1: Get your API key from Datadog
# Datadog Console → Organization Settings → API Keys → New Key

# Step 2: Install Agent (one-liner)
DD_API_KEY=<YOUR_API_KEY> \
DD_SITE="datadoghq.com" \
bash -c "$(curl -L https://s3.amazonaws.com/dd-agent-release/scripts/install_script_agent7.sh)"

# Step 3: Verify agent is running
sudo datadog-agent status

# Step 4: Check agent version
sudo datadog-agent version
```

### Install Datadog Agent on Kubernetes (Helm)

```bash
# Step 1: Add Datadog Helm repo
helm repo add datadog https://helm.datadoghq.com
helm repo update

# Step 2: Create Kubernetes secret with API key
kubectl create secret generic datadog-secret \
  --from-literal api-key=<YOUR_API_KEY> \
  --from-literal app-key=<YOUR_APP_KEY>

# Step 3: Create values.yaml
cat > datadog-values.yaml << 'EOF'
datadog:
  apiKeyExistingSecret: datadog-secret
  appKeyExistingSecret: datadog-secret
  site: datadoghq.com
  logs:
    enabled: true
    containerCollectAll: true
  apm:
    portEnabled: true
  processAgent:
    enabled: true
    processCollection: true
  kubelet:
    tlsVerify: false
clusterAgent:
  enabled: true
  metricsProvider:
    enabled: true
EOF

# Step 4: Install
helm install datadog-agent datadog/datadog \
  -n datadog \
  --create-namespace \
  -f datadog-values.yaml

# Step 5: Verify
kubectl get pods -n datadog
# NAME                                    READY   STATUS
# datadog-agent-xxxxx                     3/3     Running  ← one per node
# datadog-cluster-agent-xxxxxxx-xxxxx     1/1     Running  ← cluster agent
```

### Install Agent via Docker

```bash
docker run -d \
  --name datadog-agent \
  --cgroupns host \
  --pid host \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v /proc/:/host/proc/:ro \
  -v /sys/fs/cgroup/:/host/sys/fs/cgroup:ro \
  -e DD_API_KEY=<YOUR_API_KEY> \
  -e DD_SITE="datadoghq.com" \
  -e DD_LOGS_ENABLED=true \
  -e DD_LOGS_CONFIG_CONTAINER_COLLECT_ALL=true \
  gcr.io/datadoghq/agent:7

# Verify
docker exec -it datadog-agent agent status
```

---

## 🏷️ Tagging Strategy

Tags are key to filtering and grouping in Datadog:

```yaml
# Recommended tag structure
env: prod | staging | dev
service: api | frontend | payment | auth
team: platform | backend | frontend
version: 1.2.3
region: ap-south-1
cluster: prod-eks-cluster
```

### Apply tags in Agent config
```yaml
# /etc/datadog-agent/datadog.yaml
tags:
  - env:prod
  - service:my-api
  - team:backend
  - region:ap-south-1
```

### Apply tags in Kubernetes (via labels)
```yaml
# pod spec
metadata:
  labels:
    tags.datadoghq.com/env: prod
    tags.datadoghq.com/service: my-app
    tags.datadoghq.com/version: "1.2.3"
```

---

## 📊 Building Dashboards

```
Datadog → Dashboards → New Dashboard → Choose type:
  ├── Timeboard    → Time-synced widgets (incident investigation)
  └── Screenboard  → Free-form layout (TV displays, NOC screens)

Widget types:
  ├── Timeseries  → Line chart over time
  ├── Query Value → Single number (current CPU%)
  ├── Top List    → Ranked items (slowest services)
  ├── Heatmap     → Distribution (latency spread)
  ├── Log Stream  → Live log feed
  ├── Alert Graph → Monitor status
  └── SLO Widget  → SLO burn rate
```

---

## 🔗 Key Integrations

| Integration | What You Get |
|------------|-------------|
| **AWS** | EC2, RDS, Lambda, ALB, EKS metrics via IAM role |
| **Kubernetes** | Full cluster/pod/container visibility |
| **Nginx** | Request rates, error rates, active connections |
| **PostgreSQL / MySQL** | Query performance, connections, replication lag |
| **Redis** | Hit rate, memory, connected clients |
| **Jenkins** | Build times, failures, CI/CD pipeline tracking |
| **GitHub** | Deployment tracking, commit correlation |
| **Slack** | Alert notifications |
| **PagerDuty** | On-call escalation |
| **Terraform** | Manage Datadog resources as code |

---

## 📐 SLOs (Service Level Objectives)

```
SLO = Reliability target for your service

Example SLO:
  Service: Payment API
  SLI:     Request success rate (non-5xx / total)
  Target:  99.9% over 30 days
  Error Budget: 0.1% = ~43 minutes downtime allowed

Datadog SLO Setup:
  Monitors → SLOs → New SLO
    → Type: Metric-based
    → Good events: !error requests
    → Total events: all requests
    → Target: 99.9%
    → Time window: 30 days
```

---

## 🔄 Datadog vs Competitors

| Feature | Datadog | Prometheus + Grafana | New Relic | Dynatrace |
|---------|---------|---------------------|-----------|-----------|
| Type | SaaS | Open Source | SaaS | SaaS |
| Setup | Easy (15 min) | Complex (days) | Easy | Medium |
| Cost | Expensive | Free (infra cost) | Expensive | Expensive |
| Logs | ✅ Built-in | ❌ Need Loki | ✅ | ✅ |
| APM | ✅ Excellent | ❌ Need Jaeger | ✅ | ✅ AI-powered |
| K8s | ✅ Native | ✅ Good | ✅ | ✅ |
| ML Anomaly | ✅ | ❌ | Limited | ✅ |
| Self-hosted | ❌ | ✅ | ❌ | ❌ |
| Best for | Mid-large teams | Cost-conscious | App focus | AI/auto |

---

## 💰 Pricing Overview

```
Datadog pricing is per host per month:

Infrastructure:
  Free:     5 hosts, 1-day retention
  Pro:      ~$15/host/month (1-year retention)
  Enterprise: ~$23/host/month (custom retention)

APM:
  ~$31/host/month (includes infra)

Logs:
  Ingestion: ~$0.10/GB
  Retention: ~$1.70/million log events/month

Kubernetes:
  $9/node/month (agents billed per node)
```

---

## ✅ Advantages of Datadog

- **One platform** — metrics, logs, traces all correlated
- **Easy setup** — agent installed in minutes
- **800+ integrations** — almost everything supported
- **AI/ML features** — anomaly detection, forecasting, watchdog
- **Kubernetes-native** — built for cloud-native workloads
- **Collaboration** — notebooks, shared dashboards, team features
- **Mobile app** — monitor on the go

## ❌ Limitations of Datadog

- **Expensive** — costs add up quickly at scale
- **Vendor lock-in** — data stored in Datadog cloud
- **Internet required** — no air-gapped / fully offline support
- **Complex pricing** — hard to predict bills
- **Data privacy** — sending metrics/logs to third party

---

## 🏆 Best Practices

- Use **tags consistently** — `env`, `service`, `team`, `version` on everything
- Enable **Unified Service Tagging** — correlate metrics, logs, traces automatically
- Set **composite monitors** — reduce alert noise
- Use **SLOs** — define reliability targets, not just thresholds
- Enable **Log Pipelines** — parse logs on ingest for better search
- Use **Watchdog** — Datadog's AI proactively surfaces anomalies
- Set **alert priorities** — P1 (page on-call) vs P5 (just notify)
- Review **dashboards regularly** — stale dashboards mislead teams

---

## 📚 Related Technologies

| Technology | Relationship |
|-----------|-------------|
| **Prometheus** | Open-source alternative for metrics |
| **Grafana** | Open-source alternative for dashboards |
| **Jaeger / Zipkin** | Open-source distributed tracing |
| **ELK Stack** | Open-source log management |
| **PagerDuty** | Incident management (integrates with Datadog) |
| **Terraform** | Manage Datadog monitors/dashboards as code |
| **OpenTelemetry** | Open standard for instrumentation (Datadog accepts OTLP) |
