# 🎯 Datadog — Interview Questions
> Simple → Medium → Advanced → Scenario-Based | 50 Questions Total

---

## 📌 Table of Contents

| Level | Questions | Count |
|-------|-----------|-------|
| [🟢 Simple Level](#-simple-level-questions-1-15) | Concepts, Definitions | 15 |
| [🟡 Medium Level](#-medium-level-questions-16-30) | Architecture, Config, Features | 15 |
| [🔴 Advanced Level](#-advanced-level-questions-31-40) | Deep Dive, Internals | 10 |
| [🟣 Scenario-Based](#-scenario-based-questions-41-50) | Real-World Situations | 10 |

---

## 🟢 Simple Level Questions (1–15)

---

### Q1. What is Datadog?

**Answer:**
Datadog is a **cloud-based SaaS monitoring and observability platform** that provides real-time visibility into infrastructure, applications, logs, and traces — all from a single unified dashboard.

It helps DevOps and SRE teams to:
- Monitor servers, containers, and cloud resources
- Collect and analyze application logs
- Trace requests across microservices (APM)
- Set up alerts and notifications

> **One-liner:** "Datadog is the all-in-one observability platform for modern cloud infrastructure."

---

### Q2. What is Observability? How is it different from Monitoring?

**Answer:**

| Monitoring | Observability |
|-----------|--------------|
| Watches known things (CPU, memory) | Understands unknown issues |
| Tells you **what** is wrong | Tells you **why** it is wrong |
| Predefined dashboards | Explore any question |
| Reactive | Proactive |

**The 3 Pillars of Observability:**
```
1. Metrics  → "CPU is 90%"
2. Logs     → "ERROR: database connection refused at 14:32"
3. Traces   → "Request took 4s — DB query caused it"

Together = Full observability!
```

---

### Q3. What is the Datadog Agent?

**Answer:**
The Datadog Agent is a **lightweight software** installed on your servers, containers, or Kubernetes nodes. It is responsible for:

- Collecting system metrics (CPU, memory, disk, network)
- Forwarding application logs
- Sending APM traces
- Running integrations (Nginx, MySQL, Redis)

```
Server → [Datadog Agent] → Datadog Cloud Platform
```

It runs as a system service (`datadog-agent`) and communicates with Datadog's API using your API key.

---

### Q4. What are the 3 main pillars / data types in Datadog?

**Answer:**
```
1. Metrics  → Numerical time-series data
              Example: CPU = 85%, requests/sec = 1200

2. Logs     → Text-based event records
              Example: "2024-01-01 ERROR: Payment failed for user 123"

3. Traces   → Request path through distributed services
              Example: User request → API → Auth → DB → Response (450ms total)
```

These three are correlated in Datadog — you can jump from a metric spike → related logs → exact trace.

---

### Q5. What is APM in Datadog?

**Answer:**
APM (Application Performance Monitoring) is Datadog's **distributed tracing** feature. It tracks the full lifecycle of a single request as it travels through your microservices.

**It shows:**
- Which service is slow
- Where errors are happening
- Latency breakdown per service/operation
- Flame graphs for code-level profiling

**Example:**
```
User request (450ms total):
  API Gateway:   5ms
  Auth Service:  20ms
  Product Service: 400ms  ← bottleneck!
  Database:      25ms
```

Without APM: "App is slow" — with APM: "Product Service SQL query slow."

---

### Q6. What is a Datadog Dashboard?

**Answer:**
A Dashboard is a **visual interface** in Datadog that displays metrics, logs, traces, and monitors in customizable widgets.

**Types:**
- **Timeboard** — Time-synced, used for investigations
- **Screenboard** — Free-form layout, used for NOC/TV displays

**Common widgets:**
- Timeseries (line chart)
- Query Value (single number)
- Top List (ranking)
- Log Stream (live logs)
- Alert Status

---

### Q7. What is a Datadog Monitor / Alert?

**Answer:**
A Monitor is a configured condition that **automatically alerts** you when a metric, log, or trace exceeds a threshold.

**Example:**
```
Monitor: "CPU Alert"
Condition: CPU > 90% for 5 minutes
Action: Send Slack message to #alerts channel
```

**Monitor states:**
- **OK** → Condition not breached
- **ALERT** → Threshold exceeded
- **WARN** → Approaching threshold
- **NO DATA** → No metrics received

---

### Q8. What is a Datadog Integration?

**Answer:**
An Integration is a **pre-built connector** that allows Datadog to collect data from external services and technologies automatically.

**Examples:**
- AWS integration → pulls EC2, RDS, Lambda metrics
- Kubernetes integration → monitors pods, nodes, deployments
- Nginx integration → tracks requests, errors, connections
- Slack integration → sends alert notifications

800+ integrations available — most are free.

---

### Q9. What is a Tag in Datadog?

**Answer:**
Tags are **key:value labels** attached to metrics, logs, and hosts that allow you to filter, group, and compare data.

```
Examples:
  env:prod
  env:staging
  service:payment-api
  team:backend
  region:ap-south-1
  version:1.2.3
```

**Why tags matter:**
```
Without tags: "Average CPU across ALL servers = 65%"
With tags:    "Average CPU for env:prod, service:api = 85% — ALERT!"
```

---

### Q10. How is Datadog different from Prometheus + Grafana?

**Answer:**

| Feature | Datadog | Prometheus + Grafana |
|---------|---------|---------------------|
| Type | SaaS (cloud-hosted) | Open Source (self-hosted) |
| Setup | 15 minutes | Days to weeks |
| Logs | ✅ Built-in | Needs Loki (extra) |
| APM/Tracing | ✅ Built-in | Needs Jaeger/Tempo |
| Cost | Paid (~$15-31/host/month) | Free (infra cost only) |
| Scalability | Unlimited (AWS handles it) | You manage scaling |
| AI/ML | ✅ Watchdog, anomaly detection | ❌ |
| Support | 24/7 enterprise support | Community only |

**When to choose Datadog:** Large teams, need fast setup, want everything in one place.
**When to choose P+G:** Budget-conscious, open-source preference, full control.

---

### Q11. What is DogStatsD?

**Answer:**
DogStatsD is a **custom metrics collection service** built into the Datadog Agent. It listens on UDP port 8125 for custom metrics sent from your application code.

```bash
# Metric format:
metric_name:value|type|#tag1:val1,tag2:val2

# Examples:
page.views:1|c|#page:home            # Counter
api.latency:150|ms|#endpoint:/users  # Timer
cart.items:5|g|#user:123            # Gauge
```

**Use case:** Send business metrics from your app (orders per minute, checkout success rate) to Datadog.

---

### Q12. What ports does the Datadog Agent use?

**Answer:**

| Port | Protocol | Purpose |
|------|----------|---------|
| 443 | TCP/HTTPS | Agent → Datadog cloud (metrics/logs) |
| 8125 | UDP | DogStatsD custom metrics |
| 8126 | TCP | APM traces from app |
| 5000 | TCP | Cluster Agent (Kubernetes) |
| 5001 | TCP | Cluster Agent API (Kubernetes) |

> Most important: **Outbound 443 must be open** from your host to Datadog's endpoints.

---

### Q13. What is Datadog Log Management?

**Answer:**
Datadog Log Management is a **centralized log collection, storage, search, and analysis** service.

```
Sources:
  Files → Agent tails log files
  Docker/K8s → Container stdout/stderr
  Lambda → CloudWatch Logs
  Forwarded → Syslog, HTTP

Features:
  Live Tail       → Real-time log streaming
  Log Explorer    → Search across all logs
  Log Analytics   → Aggregate and visualize
  Log Pipelines   → Parse and structure logs
  Log Archives    → Store to S3 long-term
```

---

### Q14. What is the Datadog Cluster Agent in Kubernetes?

**Answer:**
The Cluster Agent is a **centralized component** in Kubernetes deployments that:

- Communicates with Kubernetes API server (one connection instead of one per node)
- Provides cluster-level metrics to all node agents
- Runs Kubernetes State Core check
- Powers Horizontal Pod Autoscaler (HPA) with custom metrics

```
Without Cluster Agent:
  Every node agent → Kubernetes API (N connections, redundant calls)

With Cluster Agent:
  1 Cluster Agent → Kubernetes API (1 connection)
  All node agents → Cluster Agent (efficient!)
```

---

### Q15. Name 5 AWS services that Datadog can monitor.

**Answer:**
```
1. EC2           → Instance CPU, network, status checks
2. RDS           → Database CPU, connections, read/write latency
3. Lambda        → Invocations, errors, duration, throttles
4. EKS           → Kubernetes cluster via Datadog agent
5. ALB/NLB       → Request count, target response time, 5xx errors

Others:
  S3, CloudFront, SQS, SNS, DynamoDB, ElastiCache, ECS, CloudWatch
```

---

## 🟡 Medium Level Questions (16–30)

---

### Q16. Explain Datadog's Unified Service Tagging.

**Answer:**
Unified Service Tagging (UST) is Datadog's standard for **correlating metrics, logs, and traces** using three consistent tags:

```
Required tags:
  env:       prod | staging | dev
  service:   payment-api | frontend | auth
  version:   1.2.3 | latest | git-sha

Applied at:
  Agent level → datadog.yaml
  App level   → DD_ENV, DD_SERVICE, DD_VERSION env vars
  K8s level   → Pod labels: tags.datadoghq.com/env: prod

Benefit:
  CPU spike on metric → click → see only logs for that service+version
  Log error found → click → jump to exact trace that caused it
```

---

### Q17. How do you configure Datadog to monitor Nginx?

**Answer:**
```bash
# Step 1: Enable nginx stub_status module
# /etc/nginx/conf.d/status.conf
server {
  listen 81;
  location /nginx_status {
    stub_status on;
    allow 127.0.0.1;
    deny all;
  }
}
sudo nginx -t && sudo nginx -s reload

# Step 2: Create Datadog check config
sudo mkdir -p /etc/datadog-agent/conf.d/nginx.d
sudo nano /etc/datadog-agent/conf.d/nginx.d/conf.yaml

# Content:
init_config:
instances:
  - nginx_status_url: http://localhost:81/nginx_status
    tags:
      - env:prod
      - service:nginx

# Step 3: Restart agent
sudo systemctl restart datadog-agent

# Step 4: Verify check runs
sudo datadog-agent check nginx

# Metrics you get:
nginx.net.connections
nginx.net.request_per_s
nginx.net.waiting
```

---

### Q18. What is the difference between a Timeboard and Screenboard in Datadog?

**Answer:**

| Feature | Timeboard | Screenboard |
|---------|-----------|-------------|
| Time axis | All graphs time-synced | Independent per widget |
| Layout | Grid layout | Free-form drag/drop |
| Best for | Incident investigation | NOC displays, TV screens |
| Hover | Shows data across ALL graphs at same timestamp | Each graph independent |
| Share | Snapshot with exact time | Live view |

---

### Q19. Explain how Datadog APM traces a distributed request.

**Answer:**
```
How it works:

1. HTTP request hits Service A (API)
   → dd-trace library creates: Trace ID: abc123, Span ID: 001

2. Service A calls Service B (Auth)
   → Injects trace context in HTTP headers:
     x-datadog-trace-id: abc123
     x-datadog-parent-id: 001
   → Service B creates Span ID: 002 (child of 001)

3. Service B calls Database
   → Creates Span ID: 003 (child of 002)
   → DB query tracked with duration

4. All spans sent to Datadog Agent:
   Trace: abc123
     └── Span 001 (API Gateway):  5ms
         └── Span 002 (Auth):    20ms
             └── Span 003 (DB):  15ms

Flame graph shows exactly where time is spent!
```

---

### Q20. What is a Composite Monitor? Give an example.

**Answer:**
A Composite Monitor combines **multiple monitors** using boolean logic (AND, OR, NOT) to trigger alerts only when specific combinations are true.

```
Problem: Too many false alerts (CPU spikes briefly, no real issue)

Solution: Composite Monitor
  → Alert ONLY when: CPU > 90% AND Memory > 85%

Setup:
  Monitor A: "CPU High"   → CPU > 90%
  Monitor B: "Memory High" → Memory > 85%

  Composite: Monitor A && Monitor B → Alert to PagerDuty

Real example:
  Only page on-call if:
    Pod CPU > 95% (Monitor A)
    AND Pod is not in Terminating state (Monitor B)
    AND Pod has been running > 5 min (Monitor C)
```

---

### Q21. What is an SLO in Datadog? How do you set one up?

**Answer:**
SLO (Service Level Objective) is a **reliability target** for a service, with an **error budget** showing remaining tolerance for failures.

```
Example SLO:
  Service: Payment API
  SLI (metric): success_rate = (non-5xx requests / total requests) × 100
  Target: 99.9%
  Window: 30 days
  Error Budget: 0.1% = ~43 minutes downtime/month

Setup in Datadog:
  Monitors → SLOs → New SLO
    → Type: Metric-based
    → Numerator: sum:requests.success{service:payment}.as_count()
    → Denominator: sum:requests.total{service:payment}.as_count()
    → Target: 99.9%
    → Time window: 30 days

Dashboard widget:
  Shows: Current % | Error budget remaining | Burn rate
```

---

### Q22. What is Datadog Watchdog?

**Answer:**
Watchdog is Datadog's **AI/ML-powered automatic anomaly detection** engine that continuously scans your metrics, APM data, and logs to proactively surface issues — without you setting up any monitors.

```
What Watchdog detects automatically:
  ✅ Sudden increase in error rates
  ✅ Unusual latency spikes
  ✅ Memory leaks (gradual memory increase)
  ✅ Unexpected traffic changes
  ✅ Database query performance degradation
  ✅ Anomalous process CPU usage

How it works:
  Watchdog learns your baseline over time
  → Detects deviations from normal
  → Creates "Watchdog Alerts" in Datadog
  → No threshold configuration needed!
```

---

### Q23. How does Datadog handle log parsing via Pipelines?

**Answer:**
Log Pipelines **automatically parse, enrich, and structure** raw logs as they are ingested, before storage.

```
Raw log (unstructured):
  "2024-01-01 14:30:00 ERROR Payment failed: user_id=123 amount=5000"

After Pipeline (structured):
  {
    "timestamp": "2024-01-01T14:30:00",
    "status": "error",
    "message": "Payment failed",
    "user_id": "123",
    "amount": 5000,
    "service": "payment",
    "env": "prod"
  }

Pipeline Processors:
  Grok Parser     → Extract fields with patterns
  Remapper        → Rename attributes
  Status Remapper → Map "ERROR" to log status error
  URL Parser      → Extract URL components
  User-Agent Parser → Parse browser info
  Lookup Processor → Enrich with external data
```

---

### Q24. How do you install Datadog on Kubernetes using Helm? List the key configuration options.

**Answer:**
```bash
# Step 1: Add repo
helm repo add datadog https://helm.datadoghq.com
helm repo update

# Step 2: Create secret
kubectl create secret generic datadog-secret \
  --from-literal api-key=<API_KEY> \
  --from-literal app-key=<APP_KEY> \
  -n datadog

# Step 3: values.yaml (key options)
datadog:
  apiKeyExistingSecret: datadog-secret
  appKeyExistingSecret: datadog-secret
  site: datadoghq.com

  # Logs collection
  logs:
    enabled: true
    containerCollectAll: true     # Collect ALL container logs

  # APM
  apm:
    portEnabled: true             # Enable APM

  # Live Process monitoring
  processAgent:
    enabled: true
    processCollection: true

  # Kubernetes state
  kubeStateMetricsEnabled: true

  # Tags
  tags:
    - env:prod
    - cluster:my-eks-cluster

  # Kubernetes version check
  kubelet:
    tlsVerify: false

# Cluster Agent
clusterAgent:
  enabled: true
  metricsProvider:
    enabled: true                 # HPA with custom metrics

# Step 4: Install
helm install datadog-agent datadog/datadog \
  -n datadog --create-namespace -f values.yaml
```

---

### Q25. What is Datadog Log Retention and Archive?

**Answer:**
```
Retention:
  Standard: 15 days (default)
  Extended: Up to 30 days (paid add-on)
  After retention: Logs deleted unless archived

Log Archives:
  Long-term storage by forwarding logs to S3/Azure Blob/GCS

Setup:
  Datadog → Logs → Configuration → Archives → Add Archive
  → Destination: AWS S3
  → Bucket: my-log-archive-bucket
  → Path: /datadog-logs/
  → Filter: env:prod (optional — archive only prod)

Rehydration:
  Need to analyze old logs?
  → Archives → Rehydrate from Archive
  → Select time range → loads back into Log Explorer
  Cost: Per GB rehydrated
```

---

### Q26. What is Datadog Network Performance Monitoring (NPM)?

**Answer:**
NPM gives **real-time visibility into network traffic flows** between services, hosts, containers, and cloud regions — without changing application code.

```
What it shows:
  ├── Traffic volume between services
  ├── Latency between endpoints
  ├── Retransmits and packet loss
  ├── DNS resolution issues
  └── Connection failures

Example insight:
  "Service A → Service B: 50ms latency, 2% packet loss"
  Without NPM, this is invisible at application level!

Requirements:
  Linux kernel 4.4.0+
  Agent with system-probe enabled
  Kubernetes: requires eBPF support
```

---

### Q27. Explain Datadog's pricing model.

**Answer:**
```
Datadog charges per HOST per MONTH:

Infrastructure (Agent):
  Free:       Up to 5 hosts, 1-day retention
  Pro:        ~$15/host/month (13-month retention)
  Enterprise: ~$23/host/month

APM:
  ~$31/host/month (includes Infrastructure)
  Indexed spans: extra charge per million spans

Logs:
  Ingestion: ~$0.10/GB/month
  Retention: ~$1.70 per million log events/month (15-day)

Kubernetes:
  $9/node/month (billed per K8s node)

Custom Metrics:
  $5 per 100 custom metrics/month

Tip: Use "Metrics without Limits" to reduce custom metric costs
```

---

### Q28. What is the difference between a Metric Monitor and a Log Monitor?

**Answer:**

| Feature | Metric Monitor | Log Monitor |
|---------|---------------|-------------|
| Data source | Time-series metrics (CPU, custom) | Log events |
| Trigger | When metric crosses threshold | When log count/pattern matches |
| Latency | Near real-time (seconds) | 1-2 minute delay |
| Example | CPU > 90% for 5 min | >100 ERROR logs in 5 min |
| Use case | Performance, capacity | Application errors, security |
| Alert type | Value-based | Count or facet-based |

---

### Q29. How does Datadog's anomaly detection work?

**Answer:**
Datadog's anomaly detection uses **ML algorithms** to automatically determine "normal" behavior and alert when something deviates.

```
Algorithms:
  1. Basic     → Simple statistical bounds (mean ± N std deviations)
  2. Agile     → Handles weekday/weekend patterns
  3. Robust    → Handles outliers in baseline

Example:
  Traffic is normally 1000 req/sec on weekdays, 200 on weekends
  Basic would incorrectly alert on weekend "low" traffic
  Agile learns: "200 req/sec is NORMAL on weekends" — no alert!

Setup:
  Monitor → New Monitor → Anomaly
  → Metric: web.requests
  → Algorithm: Agile
  → Deviations: 2
  → Window: 10 minutes
```

---

### Q30. How do you use Datadog with Terraform (Infrastructure as Code)?

**Answer:**
```hcl
# Provider setup
terraform {
  required_providers {
    datadog = {
      source  = "DataDog/datadog"
      version = "~> 3.0"
    }
  }
}

provider "datadog" {
  api_key = var.datadog_api_key
  app_key = var.datadog_app_key
}

# Create a monitor
resource "datadog_monitor" "cpu_alert" {
  name    = "High CPU Alert - ${var.env}"
  type    = "metric alert"
  message = "CPU is high on {{host.name}}! @slack-alerts"

  query = "avg(last_5m):avg:system.cpu.user{env:${var.env}} > 90"

  monitor_thresholds {
    critical = 90
    warning  = 75
  }

  tags = ["env:${var.env}", "team:platform"]
}

# Create a dashboard
resource "datadog_dashboard" "main" {
  title       = "Production Dashboard"
  description = "Main production monitoring"
  layout_type = "ordered"

  widget {
    timeseries_definition {
      title = "CPU Usage"
      request {
        q = "avg:system.cpu.user{env:prod} by {host}"
      }
    }
  }
}
```

---

## 🔴 Advanced Level Questions (31–40)

---

### Q31. How does the Datadog Agent collect metrics internally?

**Answer:**
```
Agent Internal Flow:

1. Scheduler
   → Runs checks every 15 seconds (default)
   → Checks: system, process, network, integrations

2. Collector
   → Each check returns metrics as (name, value, timestamp, tags)
   → Buffered in memory queue

3. Forwarder
   → Aggregates metrics (avg, max, min over flush interval)
   → Compresses and sends to api.datadoghq.com:443
   → Retry with backoff if network fails
   → Local disk buffer if agent restarted (no metric loss)

4. DogStatsD Server (parallel)
   → UDP server on :8125
   → Receives custom metrics from apps
   → Aggregates counters/gauges over flush interval (10s)

Flush interval: 10 seconds for DogStatsD, 15s for checks
```

---

### Q32. What is distributed tracing and how does Datadog implement it?

**Answer:**
```
Distributed Tracing tracks a SINGLE request across multiple services:

Implementation (W3C Trace Context / Datadog format):

1. When request enters Service A:
   dd-trace creates:
     trace_id: random 64-bit ID (unique per request)
     span_id:  random 64-bit ID (unique per operation)

2. When Service A calls Service B:
   Injects headers into outbound HTTP/gRPC:
     x-datadog-trace-id: <trace_id>
     x-datadog-parent-id: <span_id>
     x-datadog-sampling-priority: 1

3. Service B extracts headers:
   Creates new span with same trace_id, parent = A's span_id

4. Each service's dd-trace sends spans to local Agent:
   Agent → Datadog backend

5. Datadog stitches spans by trace_id into flame graph

Supported auto-instrumented libraries:
  Python: Flask, Django, FastAPI, SQLAlchemy, Redis, Celery
  Node.js: Express, HTTP, MongoDB, Redis, PostgreSQL
  Java: Spring, Tomcat, JDBC, gRPC
```

---

### Q33. What is Datadog's eBPF-based monitoring?

**Answer:**
```
eBPF (extended Berkeley Packet Filter) allows Datadog to run
programs INSIDE the Linux kernel without modifying it.

Used for:
  1. Network Performance Monitoring (NPM)
     → Capture all TCP/UDP flows at kernel level
     → No code changes in applications
     → See: source IP, dest IP, bytes, retransmits, latency

  2. Universal Service Monitoring (USM)
     → Automatically detect services and HTTP traffic
     → Zero instrumentation required
     → Works for: HTTP, Kafka, gRPC, Redis, MySQL, Postgres

  3. Cloud Workload Security
     → Detect suspicious system calls in real-time
     → Example: unexpected file write to /etc/passwd

Requirements:
  Linux kernel 4.4+  (NPM)
  Linux kernel 4.14+ (USM with full HTTPS decryption)
```

---

### Q34. How do you implement custom metrics in Datadog from a Python application?

**Answer:**
```python
# Method 1: DogStatsD (UDP, low overhead)
from datadog import initialize, statsd

initialize()   # Uses DD_AGENT_HOST, port 8125 by default

# Counter (increment on each event)
statsd.increment('orders.placed', tags=['env:prod', 'region:us'])

# Gauge (current value)
statsd.gauge('queue.depth', 500, tags=['service:worker'])

# Histogram (distribution)
statsd.histogram('api.request.duration_ms', 250, tags=['endpoint:/checkout'])

# Timer (auto-calculates distribution)
@statsd.timed('db.query.time', tags=['table:orders'])
def get_orders():
    return db.query("SELECT * FROM orders")

# Method 2: API (batch, less real-time)
from datadog import api

api.Metric.send(
    metric='custom.business.revenue',
    points=[(time.time(), 15000)],
    tags=['env:prod']
)

# Method 3: Lambda Extension (serverless)
# Add layer: arn:aws:lambda:...:datadog:Datadog-Extension:XX
# Set env: DD_SERVERLESS_LOGS_ENABLED=true
```

---

### Q35. Explain Datadog's Log Pipeline Grok Parser with an example.

**Answer:**
```
Grok is a pattern matching language used to parse unstructured logs.

Raw log:
  "2024-01-15 14:32:01 ERROR [payment-service] Transaction failed: user_id=42 amount=9999"

Grok rule:
  %{date("yyyy-MM-dd HH:mm:ss"):timestamp} %{word:level} \[%{word:service}\] %{data:message}: user_id=%{integer:user_id} amount=%{integer:amount}

Parsed result:
  {
    "timestamp": "2024-01-15 14:32:01",
    "level": "ERROR",
    "service": "payment-service",
    "message": "Transaction failed",
    "user_id": 42,
    "amount": 9999
  }

Benefits after parsing:
  ✅ Filter by: level:ERROR
  ✅ Filter by: user_id:42
  ✅ Analytics: avg(amount) where level:ERROR
  ✅ Alert: when count of ERROR > 100/min

Pipeline Setup:
  Logs → Configuration → Pipelines → Add Pipeline
  → Filter: source:payment
  → Add Processor → Grok Parser → paste rule
```

---

### Q36. How would you design a Datadog monitoring strategy for a microservices application?

**Answer:**
```
Step 1: Unified Service Tagging (Foundation)
  Apply: env, service, version tags everywhere

Step 2: Golden Signals per service (USE + RED method)
  USE: Utilization, Saturation, Errors (infra)
  RED: Rate, Errors, Duration (services)

  Monitors:
    ├── Request rate (p/s)    → Anomaly monitor
    ├── Error rate (%)        → Threshold > 1%
    ├── Latency p99 (ms)      → Threshold > 500ms
    ├── Pod restart count     → Threshold > 3/hour
    └── DB connection pool    → Threshold > 80%

Step 3: SLOs per critical service
  Payment API: 99.99% success rate
  Auth Service: 99.9% availability
  Search API: p99 < 200ms

Step 4: Correlation
  Enable APM → trace slow requests
  Enable USM → auto-detect services
  Enable NPM → see inter-service latency

Step 5: Alerting hierarchy
  P1 (critical):  → PagerDuty (wake someone up)
  P2 (high):      → Slack #incidents (immediate response)
  P3 (medium):    → Slack #monitoring (business hours)
  P4/P5 (low):    → Ticket / weekly review

Step 6: Dashboards
  Executive:  SLO burn rates, error budgets
  On-call:    Service health, active incidents
  Service:    Detailed per-service metrics, traces
```

---

### Q37. What is Datadog Continuous Profiler?

**Answer:**
```
Continuous Profiler captures CPU and memory profiles of your
RUNNING application code continuously in production.

What it shows:
  ├── Which functions use most CPU
  ├── Which code paths allocate most memory
  ├── Lock contention (threads waiting)
  └── I/O wait time per code path

Example finding:
  Without profiler: "API is slow"
  With profiler:
    hottest_function: calculate_recommendations()
    line 234: nested loop O(n²) on 10k items → 85% of CPU!
    Fix: Add caching → CPU drops from 90% to 15%

Languages: Java, Python, Go, Node.js, Ruby, .NET, PHP

Enable (Python):
  pip install ddtrace
  DD_PROFILING_ENABLED=true ddtrace-run python app.py

Zero overhead design:
  Samples code every 10ms
  <5% performance overhead
```

---

### Q38. What is Datadog's incident management workflow?

**Answer:**
```
Full Incident Lifecycle in Datadog:

1. Detection
   Monitor fires → Alert created
   OR Watchdog surfaces anomaly automatically

2. Declaration
   Datadog Incidents → Declare Incident
   → Severity: SEV1 (critical) | SEV2 | SEV3
   → Commander: who is leading
   → Slack channel auto-created: #incident-2024-001

3. Investigation
   Incident linked to:
     → Triggered monitors
     → Related dashboard
     → Correlated logs and traces
   Team adds timeline entries (what they tried)

4. Resolution
   Root cause identified
   Fix deployed
   Monitors return to OK state
   Incident marked Resolved

5. Post-mortem
   Datadog auto-generates timeline of all changes
   Team fills: What happened | Why | How to prevent
   Action items tracked to completion

Key benefit:
  All context (metrics, logs, traces, conversation) in ONE place
  No tab-switching during high-stress incidents
```

---

### Q39. Explain Datadog's role in a CI/CD pipeline.

**Answer:**
```
Datadog CI Visibility tracks your CI/CD pipeline performance:

What it monitors:
  ├── Pipeline run duration and trends
  ├── Test results and flaky tests
  ├── Failure rates per stage
  ├── Slow/bottleneck pipeline stages
  └── Deployment tracking

Deployment tracking:
  # Send deploy event to Datadog
  curl -X POST "https://api.datadoghq.com/api/v1/events" \
    -H "DD-API-KEY: ${DD_API_KEY}" \
    -d '{
      "title": "Deployed payment-service v1.2.3",
      "text": "Deploy to production",
      "tags": ["service:payment", "env:prod", "version:1.2.3"],
      "alert_type": "info"
    }'

Dashboard shows:
  Metric spike correlates with deploy marker!
  "Error rate spiked at 14:32 — same time as v1.2.3 deploy"
  → Instant correlation → rollback decision

GitHub Actions integration:
  - uses: DataDog/synthetics-ci-github-action@v1
    with:
      api_key: ${{ secrets.DD_API_KEY }}
      app_key: ${{ secrets.DD_APP_KEY }}
```

---

### Q40. What is Datadog Synthetic Monitoring?

**Answer:**
```
Synthetic Monitoring simulates user actions to proactively
test your application availability and performance.

Types:
1. API Tests
   → HTTP request to your endpoint every N minutes
   → From: multiple global locations
   → Checks: status code, response body, latency
   → Alert if: response > 2s or status != 200

2. Browser Tests
   → Headless Chrome simulates real user journey
   → "Open homepage → Login → Add to cart → Checkout"
   → Alert if any step fails
   → Records screenshots on failure

3. Multistep API Tests
   → Chain of API calls
   → Extract token from login → Use in next request

Setup:
  Synthetic Monitoring → New Test → API Test
  → URL: https://api.myapp.com/health
  → Locations: Singapore, Mumbai, Frankfurt
  → Frequency: Every 5 minutes
  → Alert if: Latency > 1000ms OR Status != 200

Value:
  "Find issues BEFORE users do"
  SLA monitoring from user perspective
```

---

## 🟣 Scenario-Based Questions (41–50)

---

### Q41. Scenario: Your production Kubernetes pod is crashing every hour. How do you use Datadog to diagnose it?

**Answer:**
```
Investigation Steps:

Step 1: Check Kubernetes Dashboard
  Datadog → Containers → Kubernetes
  → Filter: namespace:production, pod:my-app
  → Look at: Restart count graph → spikes every hour?

Step 2: Check container metrics at crash time
  → CPU → Did it spike to limit? (OOMKilled)
  → Memory → Did it hit memory limit? → OOMKilled!

Step 3: Check logs around crash time
  Log Explorer → service:my-app env:prod
  → Time: zoom to 5 minutes before restart
  → Look for: ERROR, FATAL, OOM, panic

Step 4: Check APM traces
  APM → Services → my-app
  → Look for: error traces, timeout errors before crash

Step 5: Kubernetes Events
  Log Explorer → source:kubernetes
  → filter: @evt.reason:OOMKilling
  → Confirms: "OOMKilling: Container my-app exceeded memory limit"

Root Cause: Memory limit too low
Fix:
  Increase memory limit in pod spec:
    resources:
      limits:
        memory: "512Mi"  # was 256Mi
  Add Datadog monitor: kubernetes.memory.working_set > 80% of limit
```

---

### Q42. Scenario: Your API response time suddenly increased from 200ms to 2 seconds. How do you debug this with Datadog?

**Answer:**
```
Step 1: APM Service Overview
  APM → Services → my-api
  → Latency graph: confirm p99 spike at exact time
  → Error rate: any correlated increase?

Step 2: Trace Investigation
  APM → Traces → Filter: latency > 1000ms
  → Open a slow trace → Flame graph
  → Identify: which span is taking longest?

  Example flame graph:
    Total: 2000ms
    ├── API Handler: 10ms  ← normal
    ├── Auth check: 15ms   ← normal
    └── DB Query: 1970ms   ← PROBLEM!

Step 3: Database Investigation
  Dashboard → Database (RDS/Postgres)
  → Read latency: spiked
  → Active connections: near maximum!
  → Cause: connection pool exhausted

Step 4: Correlate with deployment
  APM Service page → Deployments overlay
  → Spike started at 14:32 → matches deploy of v1.2.5

Root Cause: New code in v1.2.5 added N+1 query
  (1 query per item instead of 1 query for all items)

Fix: Fix query → deploy → latency back to 200ms
     Add Datadog monitor: db.query.duration p99 > 500ms
```

---

### Q43. Scenario: Your team needs to set up alerting but you're getting too many false alerts. How would you fix this?

**Answer:**
```
Problem Analysis:
  → Too many alerts → alert fatigue → team ignores alerts
  → Need: high signal, low noise

Solutions:

1. Increase evaluation windows
   BAD:  Alert if CPU > 90% for 1 minute (too sensitive)
   GOOD: Alert if CPU > 90% for 10 minutes (sustained issue)

2. Use rollup to smooth metrics
   avg:system.cpu.user{*}.rollup(avg, 300)
   Averages over 5-minute windows → removes spikes

3. Composite monitors
   CPU > 90% AND memory > 85% → real problem
   CPU > 90% alone → might be normal burst

4. Multi-alert on groups
   Instead of: one alert for all hosts
   Use: alert per host with dynamic thresholds

5. Anomaly detection instead of fixed thresholds
   Handles: day/night patterns, weekly cycles
   No manual threshold tuning needed

6. Use warning vs critical
   Warning: CPU > 75% → Slack notification only
   Critical: CPU > 90% → PagerDuty (wake up)

7. Downtime scheduling
   Create downtime for: maintenance windows, deployments
   → Suppress alerts during known events

8. Alert priority tagging
   P1: page → requires immediate response
   P4: notify → look at it tomorrow

Result: 80% fewer alerts, but 0 missed critical issues
```

---

### Q44. Scenario: A developer asks "Can you show me all errors for my payment service in the last hour?" How do you help them in Datadog?

**Answer:**
```
Approach:

1. Log Explorer (fastest)
   Logs → Explorer
   → Search: service:payment status:error
   → Time: Last 1 hour
   → Group by: @error.message (see top errors)
   → Results: "DB connection refused" × 234, "timeout" × 45

2. APM Error Traces
   APM → Traces
   → Filter: service:payment, status:error
   → Last 1 hour
   → Shows: exact code lines causing errors + full stack traces

3. APM Service Page
   APM → Services → payment
   → Error rate graph for last 1 hour
   → "Error Tracking" tab → unique error groups

4. Combined View (most powerful)
   Open any APM error trace
   → Click "Logs from trace" tab
   → See logs that happened DURING that exact request
   → Switch to metrics at exact timestamp

Bonus: Create a Saved View
   Log Explorer → current filters → Save View: "Payment Errors"
   → Share URL with developer
   → One click next time!
```

---

### Q45. Scenario: Management wants a dashboard showing application health for a business review. What would you build?

**Answer:**
```
Executive-level Dashboard Design:

Section 1: Business KPIs (top row)
  Widget 1: Orders per minute (gauge)
  Widget 2: Revenue processed (query value)
  Widget 3: Payment success rate (query value, color: green/red)
  Widget 4: Active users (gauge)

Section 2: Service Health (middle)
  Widget 5: SLO burn rate widget for each critical service
  Widget 6: Overall error rate timeseries (30 days)
  Widget 7: Avg response time timeseries

Section 3: Infrastructure (bottom)
  Widget 8: EC2/pod count
  Widget 9: Cloud cost trend
  Widget 10: Availability by region (heatmap)

Data sources:
  Custom metrics → DogStatsD from app
  SLOs → Datadog Monitors
  Infra → Datadog Agent

Make it exec-friendly:
  → Large fonts, simple colors (green/red/yellow)
  → Auto-refresh every 5 minutes
  → Share as link (no login needed — use public dashboard)
  → Export to PDF for weekly reports
```

---

### Q46. Scenario: Your Datadog costs are increasing rapidly. How would you reduce them?

**Answer:**
```
Audit first:
  Datadog → Plan & Usage → view top cost drivers

Reduction strategies:

1. Custom Metrics (biggest cost driver)
   → Metrics without Limits: only pay for queried metrics
   → Delete unused custom metrics from your code
   → Combine similar metrics (use tags, not separate metrics)

2. Logs
   → Increase sampling: don't log EVERY request (log 1 in 10 for 200s)
   → Exclusion filters: don't ingest DEBUG logs
     Logs → Indexes → Add Exclusion Filter: status:debug
   → Move old logs to Archives (S3 is much cheaper)
   → Reduce retention: 15 days → 7 days for dev

3. APM
   → Reduce sampling rate for high-traffic services
     # 10% sampling for healthy traffic
     DD_TRACE_SAMPLE_RATE=0.1
   → Keep 100% for errors
     DD_TRACE_ANALYTICS_ENABLED=true (only errors/slow traces)

4. Infrastructure
   → Remove agent from idle/dev servers
   → Use containerized agent (billed per node, not per container)

5. Right-size plan
   → Some hosts need Pro, some only Free
   → Separate by host tag using Datadog metrics billing

Target: 30-50% cost reduction without losing visibility
```

---

### Q47. Scenario: A new microservice is being deployed. What Datadog setup would you do before go-live?

**Answer:**
```
Pre-deployment Checklist:

1. Instrumentation (in code — developer task)
   ├── Add dd-trace library (auto-instrumentation)
   ├── Set DD_SERVICE, DD_ENV, DD_VERSION env vars
   └── Add custom business metrics (DogStatsD)

2. Log configuration
   ├── Ensure logs go to stdout/stderr (not files only)
   ├── Log format: JSON structured (easier to parse)
   └── Include trace_id in logs (auto-correlation with APM)

3. Datadog Integration setup
   ├── Create Log Pipeline for this service's log format
   ├── Enable APM for the service
   └── Set up Unified Service Tagging

4. Monitors (create BEFORE launch)
   ├── Error rate > 1% → Slack alert
   ├── p99 latency > 500ms → Slack alert
   ├── Pod restarts > 3/hour → PagerDuty
   └── No data (service down) → PagerDuty

5. SLO (define success criteria)
   ├── Availability SLO: 99.9%
   └── Latency SLO: p99 < 500ms (95% of time)

6. Dashboard
   └── Create service dashboard before launch
       → Show in deployment PR review

7. Synthetic Test
   └── Browser test for critical user journey
       → Runs every 5 min from 5 locations
       → Alert if any step fails

Result: Day-1 observability instead of debugging in production!
```

---

### Q48. Scenario: You receive an alert at 3 AM — "Payment service down." Walk through your investigation using Datadog.

**Answer:**
```
3 AM On-call Response (minute by minute):

00:00 — Alert received on PagerDuty
  → Open Datadog mobile app

00:02 — Check Incidents dashboard
  → Is there an active incident? Join if yes

00:03 — Check Service Health
  APM → Services → payment-service
  → Error rate: 100% (ALL requests failing!)
  → Request rate: 0 (no traffic reaching service!)

00:04 — Check Kubernetes
  Containers → K8s → namespace:prod
  → payment pods: 0/3 Running!  (all crashed)
  → kubectl get pods shows: CrashLoopBackOff

00:05 — Check pod logs
  Logs → service:payment → last 30 min
  → "FATAL: Cannot connect to postgres on :5432"

00:06 — Check database
  Dashboard → RDS → payment-db
  → CPU: 0% | Connections: 0 | Status: STOPPED

00:07 — Root cause found!
  RDS database stopped (scheduled maintenance window?)
  OR ran out of disk space (check FreeStorageSpace → 0!)

00:08 — Fix
  Option A: Start RDS instance
  Option B: Free disk space / increase storage

00:15 — Pods restart, connect to DB
  Monitors → payment monitors → returning to OK
  Payment service recovery confirmed ✅

00:20 — Write incident timeline in Datadog Incidents
  Postmortem: Add CloudWatch storage alarm next time
```

---

### Q49. Scenario: Your team is migrating from Prometheus + Grafana to Datadog. What is your migration plan?

**Answer:**
```
Phase 1: Parallel Run (Week 1-2)
  ├── Install Datadog Agent on all hosts (alongside Prometheus)
  ├── Enable Prometheus metrics scraping in Datadog:
  │     DD_PROMETHEUS_SCRAPE_ENABLED=true
  │     # Datadog automatically discovers /metrics endpoints!
  ├── Verify same metrics appear in both systems
  └── Don't disable Prometheus yet

Phase 2: Dashboard Migration (Week 2-3)
  ├── Identify top 10 most-used Grafana dashboards
  ├── Recreate in Datadog (metric names may differ)
  │     Prometheus: container_cpu_usage_seconds_total
  │     Datadog:    kubernetes.cpu.usage.total
  ├── Share with team for validation
  └── Grafana dashboards: mark as "deprecated"

Phase 3: Alert Migration (Week 3-4)
  ├── Export Prometheus alert rules (rules.yaml)
  ├── Recreate as Datadog Monitors
  ├── Test each alert (use force-trigger or CPU stress)
  └── Disable Prometheus Alertmanager for migrated alerts

Phase 4: Log Migration (Week 4-5)
  ├── Currently using ELK? Configure Datadog Agent log collection
  ├── Validate logs appearing in Datadog Log Explorer
  ├── Migrate Kibana saved searches → Datadog Saved Views
  └── Disable Filebeat/Logstash after validation

Phase 5: Cutover (Week 6)
  ├── Final parallel validation (2-week data comparison)
  ├── Decommission Prometheus, Grafana, ELK
  ├── Update runbooks to reference Datadog
  └── Training session for team on Datadog features

Risk mitigation:
  → Keep Prometheus for 2 weeks after cutover (safety net)
  → Have rollback plan (just restart Prometheus)
```

---

### Q50. Scenario: You need to monitor a serverless application (AWS Lambda). How would you set up Datadog for it?

**Answer:**
```
Lambda Monitoring Setup:

Option 1: Datadog Lambda Extension (recommended)
  → Add Lambda Layer: Datadog-Extension
  → Zero infrastructure needed, runs in Lambda process

# In serverless.yml or Terraform:
functions:
  my-function:
    handler: handler.main
    layers:
      - arn:aws:lambda:ap-south-1:464622532012:layer:Datadog-Extension:XX
    environment:
      DD_API_KEY: ${env:DD_API_KEY}
      DD_SITE: datadoghq.com
      DD_SERVERLESS_LOGS_ENABLED: true    # Forward Lambda logs
      DD_TRACE_ENABLED: true              # Enable APM
      DD_SERVICE: my-lambda-function
      DD_ENV: prod

Option 2: CloudWatch Logs Forwarder
  → Deploy Datadog Forwarder Lambda
  → Subscribe to CloudWatch Logs streams
  → Parses and forwards to Datadog

What you get:
  ├── Invocations, errors, duration (from CloudWatch)
  ├── Cold start tracking
  ├── Memory usage vs limit
  ├── APM traces through Lambda function
  ├── Enhanced metrics: 15-second granularity (vs 1-min CloudWatch)
  └── Log correlation with traces

Key Lambda monitors to create:
  ├── Error rate > 1% → Alert
  ├── Duration p99 > 5000ms → Alert  (approaching 15min limit?)
  ├── Throttles > 0 → Alert (concurrency issue)
  └── ConcurrentExecutions near limit → Warning

Serverless dashboard includes:
  → Cold start %, duration distribution, error breakdown by function
```

---

## 📋 Quick Reference — Interview Cheat Sheet

| Topic | Key Points |
|-------|-----------|
| What is Datadog? | SaaS observability platform — metrics, logs, traces |
| 3 pillars | Metrics → Logs → Traces (correlated) |
| Agent | Lightweight software collecting data from host |
| APM | Distributed tracing — shows slow service in microservices |
| DogStatsD | UDP:8125 — send custom metrics from app code |
| Tags | key:value labels — env:prod, service:api |
| Monitor | Alert condition — CPU > 90% for 5 min → Slack |
| Cluster Agent | K8s — one API connection for all node agents |
| UST | Unified Service Tagging — env+service+version everywhere |
| SLO | Reliability target + error budget |
| Watchdog | AI anomaly detection — no thresholds needed |
| vs Prometheus | Datadog: SaaS, easy, paid; Prometheus: OSS, free, complex |

---

