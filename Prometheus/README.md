# Prometheus Complete Study Guide

---

## Table of Contents

1. [Prometheus Fundamentals & Architecture](#module-1-prometheus-fundamentals--architecture)
2. [Installation & Configuration](#module-2-installation--configuration)
3. [Data Model & Metric Types](#module-3-data-model--metric-types)
4. [PromQL Fundamentals](#module-4-promql-fundamentals)
5. [PromQL Advanced: Aggregation, Functions, Joins](#module-5-promql-advanced-aggregation-functions-joins)
6. [Service Discovery & Scraping](#module-6-service-discovery--scraping)
7. [Exporters](#module-7-exporters)
8. [Recording Rules](#module-8-recording-rules)
9. [Alerting Rules & Alertmanager](#module-9-alerting-rules--alertmanager)
10. [Federation & Remote Write](#module-10-federation--remote-write)
11. [Storage & TSDB Internals](#module-11-storage--tsdb-internals)
12. [High Availability & Scaling: Thanos, Mimir, Cortex](#module-12-high-availability--scaling-thanos-mimir-cortex)
13. [Security](#module-13-security)
14. [Performance & Best Practices](#module-14-performance--best-practices)

---

## MODULE 1: Prometheus Fundamentals & Architecture

### 1.1 What is Prometheus?

Prometheus is an open-source systems monitoring and alerting toolkit, built around a multi-dimensional time-series data model, a pull-based metrics collection model, and its own powerful query language (PromQL) — originally built at SoundCloud, now a CNCF graduated project and the de facto standard for Kubernetes/cloud-native metrics monitoring.

---

### 1.2 Prometheus Architecture

```
        Targets (applications, exporters exposing /metrics)
                          │
                   Prometheus Server
        ┌─────────────────┼─────────────────┐
   Service Discovery   Scraper (HTTP pull)   TSDB (local storage)
   (finds targets to     every scrape_interval    (time-series database)
    scrape)                    │
                          Rule Evaluation
                     (recording rules + alerting rules)
                               │
                          Alertmanager
                     (dedup, group, route, notify)
                               │
                     Slack / PagerDuty / Email / Webhook

        Grafana (or Prometheus's own built-in Expression Browser)
                    queries Prometheus via PromQL
```

---

### 1.3 Pull vs Push Model — Why Prometheus Chose Pull

Prometheus actively "pulls" (scrapes) metrics from targets via HTTP at regular intervals, rather than targets pushing metrics to Prometheus.

**Advantages of pull:**
- Prometheus can immediately tell if a target is unreachable (a failed scrape is itself a meaningful signal — "the target is down").
- Simpler target-side implementation (just expose an HTTP endpoint, no need to know where to push).
- Easier to run ad-hoc/locally against a target for debugging (just curl the `/metrics` endpoint).

**When push is still needed (Pushgateway):** For short-lived batch jobs that complete before a scrape could occur, an intermediary Pushgateway allows a one-time push that Prometheus then scrapes from — a deliberate, narrow exception to the pull model, not a general-purpose replacement for it.

---

### 1.4 Core Components

| Component | Role |
|---|---|
| Prometheus Server | Scrapes and stores time-series data, evaluates rules |
| Alertmanager | Handles alert deduplication, grouping, routing, silencing |
| Pushgateway | Allows short-lived jobs to push metrics for later scraping |
| Exporters | Translate third-party system metrics into Prometheus format |
| Client Libraries | Instrument application code directly (Go, Java, Python, etc.) |

---

## MODULE 2: Installation & Configuration

### 2.1 Installing Prometheus

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.53.0/prometheus-2.53.0.linux-amd64.tar.gz
tar xvf prometheus-2.53.0.linux-amd64.tar.gz
cd prometheus-2.53.0.linux-amd64
./prometheus --config.file=prometheus.yml
```

Default web UI: `http://localhost:9090`.

---

### 2.2 Basic `prometheus.yml` Configuration

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node1:9100', 'node2:9100']

rule_files:
  - "rules/*.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
```

```bash
promtool check config prometheus.yml    # validate config before applying/restarting
```

---

### 2.3 Running Prometheus via Docker

```bash
docker run -d --name prometheus \
  -p 9090:9090 \
  -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

---

### 2.4 Reloading Configuration Without Restarting

```bash
curl -X POST http://localhost:9090/-/reload
# or send SIGHUP to the process
kill -HUP $(pgrep prometheus)
```

Requires the `--web.enable-lifecycle` flag to be set at startup for the HTTP reload endpoint to be available — allows applying most configuration changes (scrape configs, rule files) without a full restart/downtime.

---

## MODULE 3: Data Model & Metric Types

### 3.1 The Prometheus Data Model

```
metric_name{label1="value1", label2="value2"} value timestamp

Example:
http_requests_total{method="GET", status="200", instance="web-1:8080"} 1027 1609459200
```

Every time series is uniquely identified by its metric name plus its complete set of label key-value pairs — this multi-dimensional model is what enables flexible aggregation/filtering across any combination of labels.

---

### 3.2 The Four Core Metric Types

| Type | Description | Example |
|---|---|---|
| Counter | Monotonically increasing value (only goes up, resets to 0 on restart) | `http_requests_total` |
| Gauge | A value that can go up or down | `memory_usage_bytes` |
| Histogram | Samples observations into configurable buckets, plus sum and count | `http_request_duration_seconds` |
| Summary | Similar to histogram, but calculates quantiles client-side | `http_request_duration_seconds` (summary variant) |

---

### 3.3 Counters — Correct Usage

```promql
# WRONG — raw counter value is meaningless alone (just an ever-increasing number)
http_requests_total

# CORRECT — use rate() to get a meaningful per-second rate
rate(http_requests_total[5m])
```

**Interview point:** Never graph a raw counter directly — always wrap it in `rate()` or `increase()` — a raw counter's absolute value is rarely meaningful on its own, and `rate()` correctly handles counter resets (e.g., after an application restart) that would otherwise produce a nonsensical negative-looking spike if computed naively.

---

### 3.4 Histograms vs Summaries

```
Histogram: le="0.1", le="0.5", le="1", le="+Inf" (cumulative buckets)
           → server-side quantile calculation via histogram_quantile()
           → aggregatable across instances

Summary:   quantile="0.5", quantile="0.9", quantile="0.99"
           → client-side pre-calculated quantiles
           → NOT aggregatable across instances (can't average percentiles meaningfully)
```

**Key interview distinction:** Histograms are aggregatable across multiple instances/dimensions (you can sum buckets from many servers and then compute an overall quantile), while Summary quantiles are pre-calculated per-instance and mathematically cannot be correctly combined/averaged across instances — this is why Histograms are generally preferred for anything you might need to aggregate later (e.g., cluster-wide p99 latency).

---

### 3.5 Labels and Cardinality

```promql
http_requests_total{method="GET", path="/api/users/12345", status="200"}
```

**Cardinality warning:** Never use unbounded/high-cardinality values as label values (like a raw user ID, full request path with dynamic segments, or a timestamp) — each unique label combination creates a new, separate time series, and excessive cardinality is the single most common cause of Prometheus performance/memory problems in production.

---

## MODULE 4: PromQL Fundamentals

### 4.1 Instant Vectors vs Range Vectors

```promql
http_requests_total                    # instant vector — one value per series, at a single point in time
http_requests_total[5m]                 # range vector — a range of values per series, over the last 5 minutes
```

Range vectors can't be graphed directly — they're typically fed into a function like `rate()` or `increase()` that reduces them back into an instant vector.

---

### 4.2 `rate()` vs `irate()` vs `increase()`

```promql
rate(http_requests_total[5m])       # average per-second rate over the 5m window, smoothed
irate(http_requests_total[5m])       # instantaneous rate, using only the last two data points in the window
increase(http_requests_total[5m])     # total increase over the 5m window (rate * window duration)
```

**Interview point:** `rate()` is generally preferred for alerting and dashboards (smoother, less noisy); `irate()` is better suited for fast-moving, volatile counters where you want to see rapid changes, but is much noisier and not recommended for alerting due to its sensitivity to brief spikes.

---

### 4.3 Basic Selectors and Matching

```promql
http_requests_total{status="500"}                # exact match
http_requests_total{status!="500"}                 # negative match
http_requests_total{status=~"5.."}                  # regex match
http_requests_total{status!~"2..|3.."}               # negative regex match
```

---

### 4.4 Arithmetic and Comparison Operators

```promql
node_memory_MemFree_bytes / node_memory_MemTotal_bytes * 100   # arithmetic between two vectors
http_requests_total > 1000                                       # filter: keep only series with value > 1000
```

---

### 4.5 Offset and `@` Modifiers

```promql
http_requests_total offset 1h            # value as it was 1 hour ago
rate(http_requests_total[5m] offset 1d)   # compare current rate to same time yesterday
```

---

## MODULE 5: PromQL Advanced: Aggregation, Functions, Joins

### 5.1 Aggregation Operators

```promql
sum(rate(http_requests_total[5m]))                       # total across everything
sum(rate(http_requests_total[5m])) by (service)            # grouped by a specific label
sum(rate(http_requests_total[5m])) without (instance)        # grouped by everything EXCEPT instance
avg(node_cpu_usage) by (instance)
max(node_memory_usage) by (instance)
count(up == 1)                                                 # count of currently "up" targets
topk(5, rate(http_requests_total[5m]))                          # top 5 series by value
```

**Interview point on `by` vs `without`:** `by (label)` keeps only the specified label(s) in the result, discarding all others; `without (label)` keeps all labels EXCEPT the specified one(s) — choose based on which is more concise for your specific grouping intent.

---

### 5.2 `histogram_quantile()`

```promql
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
```

Computes an approximate quantile (p95 here) from histogram bucket data — note the `sum(...) by (le)` pattern, which is required when aggregating histogram buckets across multiple instances/series before computing the quantile (you must preserve the `le` label through the aggregation, or `histogram_quantile()` can't do its calculation correctly).

---

### 5.3 `label_replace()` and `label_join()`

```promql
label_replace(up, "host", "$1", "instance", "(.+):.*")
```

Used for manipulating labels within a query — e.g., extracting just the hostname portion from an `instance` label that includes a port, useful for joining with data from a different metric that only has the hostname without a port.

---

### 5.4 Vector Matching and Joins

```promql
# One-to-one matching (default) — both sides must have matching label sets
node_memory_MemFree_bytes / node_memory_MemTotal_bytes

# many-to-one matching — explicit group_left/group_right needed when cardinality differs
kube_pod_info * on(pod) group_left(node) kube_pod_status_phase
```

`group_left`/`group_right` are required whenever you're joining two vectors where one side has additional labels not present in the other, or where the cardinality relationship isn't strictly one-to-one — a very commonly tested PromQL interview topic.

---

### 5.5 Useful Functions

```promql
abs(), ceil(), floor(), round()
predict_linear(node_filesystem_free_bytes[6h], 4 * 3600)   # predict value 4 hours in the future based on 6h trend
delta(cpu_temp_celsius[1h])                                    # difference between first and last value in range (for gauges)
deriv(cpu_temp_celsius[1h])                                     # per-second derivative (rate of change) for gauges
clamp_max(value, 100)                                            # cap a value at a maximum
```

---

## MODULE 6: Service Discovery & Scraping

### 6.1 Static vs Dynamic Service Discovery

```yaml
# Static — manually maintained list, doesn't scale for dynamic/cloud environments
scrape_configs:
  - job_name: 'static'
    static_configs:
      - targets: ['host1:9100', 'host2:9100']

# Dynamic — Kubernetes service discovery example
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
```

Prometheus supports numerous service discovery mechanisms (Kubernetes, Consul, EC2, Azure, GCE, file-based, DNS) — dynamic discovery is essential in any environment where targets are frequently created/destroyed (autoscaling groups, Kubernetes pods).

---

### 6.2 Relabeling

```yaml
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: (.+)
        replacement: ${1}
```

Relabeling is one of Prometheus's most powerful (and initially confusing) features — it lets you filter which discovered targets are actually scraped, and rewrite/derive labels from Kubernetes metadata (like using a pod annotation to control whether it's scraped at all) before the scrape even happens.

---

### 6.3 `metric_relabel_configs` vs `relabel_configs`

```
relabel_configs         → applied to TARGETS before scraping (can filter which targets are scraped at all)
metric_relabel_configs  → applied to METRICS/samples AFTER scraping, before storage (can drop specific unwanted metrics)
```

A common interview trap: confusing these two — `relabel_configs` operates at the target-selection stage; `metric_relabel_configs` operates on the actual scraped sample data afterward, commonly used to drop unwanted, expensive, high-cardinality metrics before they're ever stored.

---

### 6.4 Scrape Configuration Options

```yaml
scrape_configs:
  - job_name: 'myapp'
    scrape_interval: 30s
    scrape_timeout: 10s
    metrics_path: /custom-metrics
    scheme: https
    tls_config:
      insecure_skip_verify: false
    static_configs:
      - targets: ['myapp:8443']
```

---

## MODULE 7: Exporters

### 7.1 What is an Exporter?

An exporter translates metrics from a third-party system (that doesn't natively expose Prometheus-format metrics) into the Prometheus exposition format, exposed via an HTTP `/metrics` endpoint that Prometheus can scrape.

```
Third-party system (MySQL, Redis, hardware)
        │
Exporter (mysqld_exporter, redis_exporter, node_exporter)
        │
Exposes /metrics in Prometheus format
        │
Prometheus scrapes it like any other target
```

---

### 7.2 Common Exporters

| Exporter | Purpose |
|---|---|
| node_exporter | Host-level metrics (CPU, memory, disk, network) for Linux/Unix machines |
| blackbox_exporter | Probes endpoints over HTTP/TCP/ICMP/DNS to measure availability/latency |
| mysqld_exporter | MySQL database metrics |
| redis_exporter | Redis metrics |
| kube-state-metrics | Kubernetes object state (not node-level — cluster object states like deployments, pods) |
| cAdvisor | Container-level resource usage metrics |

---

### 7.3 `node_exporter` Basics

```bash
./node_exporter --web.listen-address=":9100"
curl http://localhost:9100/metrics | grep node_memory
```

```yaml
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

---

### 7.4 `blackbox_exporter` for Endpoint Probing

```yaml
scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
```

This is a very common, frequently-tested pattern: Prometheus scrapes the `blackbox_exporter` itself, but instructs it (via the `__param_target` relabel) which external target to actually probe, and rewrites `__address__` to point to the blackbox_exporter's own address rather than the probed target's address.

---

### 7.5 kube-state-metrics vs cAdvisor vs node_exporter — A Common Kubernetes Monitoring Confusion

```
node_exporter        → host-level metrics (the underlying VM/node's CPU, memory, disk)
cAdvisor (built into kubelet)  → container-level resource usage (per-container CPU/memory)
kube-state-metrics    → Kubernetes API object state (deployment replica counts, pod phase, etc.) — NOT resource usage at all
```

A very commonly tested distinction: `kube-state-metrics` doesn't report resource usage (CPU/memory) at all — it reports the *state* of Kubernetes objects as recorded in the API server (e.g., "this deployment wants 3 replicas but only has 2 ready") — genuinely different information from cAdvisor's actual resource consumption metrics.

---

## MODULE 8: Recording Rules

### 8.1 What are Recording Rules?

Recording rules pre-compute frequently-needed or expensive PromQL expressions on a schedule, saving the result as a new time series — improving query performance for dashboards/alerts that would otherwise repeatedly recompute the same expensive expression.

```yaml
groups:
  - name: example
    interval: 30s
    rules:
      - record: job:http_requests:rate5m
        expr: sum(rate(http_requests_total[5m])) by (job)
```

Naming convention: `level:metric:operations` (e.g., `job:http_requests:rate5m`) is a widely-adopted community convention indicating the aggregation level and applied operations.

---

### 8.2 When to Use Recording Rules

- A dashboard panel's query is expensive and used across many dashboards/viewers.
- An alerting rule's underlying expression is complex and needs fast, reliable re-evaluation.
- You want a stable, well-known metric name abstracting away a more complex underlying calculation.

```bash
promtool check rules recording_rules.yml    # validate rule file syntax before deploying
```

---

## MODULE 9: Alerting Rules & Alertmanager

### 9.1 Defining an Alerting Rule

```yaml
groups:
  - name: example
    rules:
      - alert: HighErrorRate
        expr: sum(rate(http_requests_total{status="500"}[5m])) / sum(rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate on {{ $labels.job }}"
          description: "Error rate is {{ $value | humanizePercentage }} for the last 5 minutes."
```

`for: 5m` requires the condition to remain true continuously for 5 minutes before the alert actually fires (transitions to "firing") — preventing brief, transient spikes from immediately triggering a page.

---

### 9.2 Alert States

```
Inactive → Pending (condition true, but "for" duration not yet elapsed) → Firing (condition sustained, notification sent)
```

---

### 9.3 Alertmanager Architecture

```
Prometheus (alert rules fire)
        │
Alertmanager
        │
   ┌────┼────┬─────────┐
Grouping  Inhibition  Silencing
   │
Routing Tree (label-based routing)
   │
Receivers (Slack, PagerDuty, Email, Webhook)
```

---

### 9.4 Alertmanager Routing Configuration

```yaml
route:
  receiver: default
  group_by: ['alertname', 'cluster']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  routes:
    - match:
        severity: critical
      receiver: pagerduty

receivers:
  - name: default
    slack_configs:
      - channel: '#alerts'
  - name: pagerduty
    pagerduty_configs:
      - service_key: '<key>'
```

- **`group_wait`** — how long to wait for additional alerts to arrive before sending the first notification for a new group (reducing notification storms from many alerts firing near-simultaneously).
- **`group_interval`** — how long to wait before sending a notification about new alerts added to an already-notified group.
- **`repeat_interval`** — how long to wait before re-sending a notification for an alert that's still firing.

---

### 9.5 Inhibition Rules

```yaml
inhibit_rules:
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal: ['alertname', 'cluster']
```

Inhibition suppresses lower-severity alerts if a corresponding higher-severity alert (matching on specified labels) is already firing — e.g., don't bother notifying about "high latency" warnings if a "service down" critical alert for the same service is already active, since the warning is likely just a symptom of the same underlying outage.

---

### 9.6 Silences

```bash
amtool silence add alertname="HighErrorRate" --duration="2h" --comment="Planned maintenance"
```

Temporarily suppresses matching alerts — commonly used during planned maintenance windows.

---

## MODULE 10: Federation & Remote Write

### 10.1 Federation

```yaml
scrape_configs:
  - job_name: 'federate'
    honor_labels: true
    metrics_path: '/federate'
    params:
      'match[]':
        - '{job="node_exporter"}'
    static_configs:
      - targets:
          - 'prometheus-datacenter-1:9090'
```

Federation lets a "global" Prometheus scrape a subset of already-aggregated metrics from multiple "local" Prometheus servers — useful for hierarchical setups (per-datacenter Prometheus instances feeding into a global aggregation view), but **not** intended as a general-purpose way to centralize all raw metrics data (better solved by remote write/Thanos/Mimir).

---

### 10.2 Remote Write

```yaml
remote_write:
  - url: "https://mimir.mycompany.com/api/v1/push"
    basic_auth:
      username: prometheus
      password: <password>
```

Remote Write continuously streams every scraped sample to an external, long-term storage system (Thanos, Mimir, Cortex, or various cloud-managed Prometheus-compatible backends) — the standard modern approach for long-term retention and cross-cluster/global querying, generally preferred over federation for these use cases.

---

### 10.3 Remote Read

```yaml
remote_read:
  - url: "https://mimir.mycompany.com/api/v1/read"
```

Allows Prometheus to query historical data from a remote long-term storage backend as if it were local data — often used in combination with remote write to give Prometheus a unified query experience spanning both its own local retention window and older data now living only in the remote store.

---

## MODULE 11: Storage & TSDB Internals

### 11.1 Prometheus's Local TSDB (Time-Series Database)

```
Prometheus Storage
        │
   ┌────┼────┐
 Head Block   Persistent Blocks (2-hour chunks, compacted over time)
 (in-memory,  (on-disk, immutable once written)
  recent data)
        │
Write-Ahead Log (WAL) — ensures durability against crashes before data is compacted to disk
```

- Data is initially written to an in-memory "head block" plus a Write-Ahead Log (for crash recovery).
- Every 2 hours, the head block is compacted into an immutable, on-disk block.
- Background compaction later merges smaller blocks into larger ones over time, improving query efficiency and reducing storage overhead.

---

### 11.2 Retention Configuration

```bash
./prometheus --storage.tsdb.retention.time=15d
./prometheus --storage.tsdb.retention.size=50GB
```

By default, Prometheus retains data for 15 days locally — for longer retention, either increase local retention (bounded by available disk) or (the generally recommended approach at scale) use remote write to a long-term storage system like Thanos/Mimir rather than growing local Prometheus retention indefinitely.

---

### 11.3 Why Prometheus's Local Storage Isn't Designed for "Infinite" Retention

Local single-node storage doesn't provide replication/durability guarantees beyond the single disk it's on, and scaling storage/query performance indefinitely on a single node has practical limits — this is precisely the gap that Thanos/Mimir/Cortex address by providing durable, horizontally-scalable, long-term storage built on top of (or compatible with) Prometheus's data model.

---

### 11.4 Checking Storage Usage and TSDB Stats

```bash
curl http://localhost:9090/api/v1/status/tsdb
```

Returns statistics like the number of active time series, top metrics by cardinality, and label cardinality — invaluable for diagnosing cardinality-related performance issues.

---

## MODULE 12: High Availability & Scaling: Thanos, Mimir, Cortex

### 12.1 Basic Prometheus HA Pattern

```
Two identical Prometheus instances, both scraping the same targets independently
        │
Alertmanager deduplicates alerts from both (since both would fire identically)
```

The simplest Prometheus HA approach is simply running two (or more) fully independent, identically-configured Prometheus instances scraping the same targets — if one fails, the other continues; Alertmanager's clustering/gossip protocol handles deduplicating alerts that both instances would otherwise fire redundantly.

---

### 12.2 Why This Basic HA Pattern Has Limits

Each independent Prometheus instance still only has its own local storage/retention — there's no unified long-term storage, no global query view across instances, and no automatic downsampling for older data — this is exactly the gap that Thanos/Mimir/Cortex solve.

---

### 12.3 Thanos Architecture (High-Level)

```
Prometheus + Thanos Sidecar (uploads blocks to object storage: S3/GCS)
        │
Thanos Store Gateway (serves historical data from object storage)
        │
Thanos Query (global query layer, deduplicates across multiple Prometheus/Store instances)
        │
Thanos Compactor (downsampling, compaction of historical data in object storage)
```

Thanos extends Prometheus with long-term, durable, object-storage-backed retention and a global query view across many Prometheus instances/clusters, while remaining fully compatible with existing Prometheus deployments (via the sidecar pattern).

---

### 12.4 Mimir and Cortex (High-Level)

Mimir (Grafana Labs) and Cortex are both horizontally-scalable, multi-tenant, Prometheus-compatible long-term metrics storage systems — architecturally different from Thanos (built as a distributed system from the ground up rather than extending individual Prometheus instances via a sidecar), commonly used specifically via remote write rather than the sidecar/object-storage-upload pattern Thanos uses.

---

### 12.5 Choosing Between Thanos, Mimir, and Cortex

| Consideration | Guidance |
|---|---|
| Already running standalone Prometheus, want to extend it | Thanos (sidecar pattern integrates naturally) |
| Building a centralized, multi-tenant metrics platform from scratch | Mimir or Cortex (designed for this from the ground up) |
| Using Grafana Cloud or want tight Grafana Labs ecosystem alignment | Mimir |
| Established Cortex user base/operational expertise already exists | Cortex (though many Cortex users have migrated to Mimir over time) |

---

## MODULE 13: Security

### 13.1 Prometheus Has No Built-in Authentication by Default

Prometheus itself doesn't provide native user authentication/authorization out of the box — production deployments typically place a reverse proxy (nginx, oauth2-proxy) in front of Prometheus to handle authentication, or rely on network-level access control (private VPC, firewall rules) to restrict access.

---

### 13.2 TLS for Scraping

```yaml
scrape_configs:
  - job_name: 'secure-target'
    scheme: https
    tls_config:
      ca_file: /etc/prometheus/ca.crt
      cert_file: /etc/prometheus/client.crt
      key_file: /etc/prometheus/client.key
```

---

### 13.3 Restricting Access to the Admin API

```bash
./prometheus --web.enable-admin-api=false    # disabled by default in recent versions — enable only if genuinely needed
```

The Admin API (allowing operations like deleting time series data) is a powerful, potentially destructive capability — it should only be enabled where genuinely needed and appropriately access-controlled, since Prometheus itself has no native authorization layer to restrict who can call it once enabled.

---

### 13.4 Securing Alertmanager Webhooks/Receivers

Ensure webhook URLs and API keys/tokens configured in Alertmanager receivers (Slack webhooks, PagerDuty integration keys) are treated as sensitive secrets — stored via appropriate secrets management rather than committed in plaintext to version control, since anyone with these values could potentially send spoofed notifications or, depending on the specific integration, take other unintended actions.

---

## MODULE 14: Performance & Best Practices

### 14.1 Cardinality Management

```promql
# Check total number of active time series
prometheus_tsdb_head_series

# Find highest-cardinality metrics
topk(10, count by (__name__)({__name__=~".+"}))
```

Cardinality (the number of unique label combinations) is the single most important factor in Prometheus's memory/performance characteristics — avoid label values with unbounded/high-cardinality dimensions (user IDs, full URLs with dynamic path segments, timestamps as labels).

---

### 14.2 Scrape Interval Tuning

```yaml
global:
  scrape_interval: 15s     # default — a reasonable balance for most use cases
```

Shorter scrape intervals increase resolution but also increase storage/CPU load proportionally — most organizations don't need sub-15-second resolution for general infrastructure monitoring; reserve shorter intervals for genuinely latency-sensitive, high-value specific targets rather than applying them globally.

---

### 14.3 Using Recording Rules to Reduce Query Load

As covered in Module 8 — pre-computing expensive, frequently-used aggregations via recording rules significantly reduces the real-time query burden on Prometheus, especially for dashboards viewed by many users or alerting rules evaluated frequently.

---

### 14.4 Resource Sizing Guidance

Prometheus's memory usage scales primarily with the number of active time series (cardinality) and the configured retention — as a very rough rule of thumb, more active series requires proportionally more memory for the in-memory head block; monitor `prometheus_tsdb_head_series` and actual process memory usage over time to inform capacity planning rather than guessing.

---

### 14.5 Best Practices Checklist

- Always wrap counters in `rate()`/`increase()` — never graph raw counter values.
- Use Histograms (not Summaries) for anything you might need to aggregate across instances later.
- Keep label cardinality bounded — never use raw user IDs, full dynamic URLs, or timestamps as label values.
- Use recording rules for expensive, frequently-reused expressions.
- Validate configuration with `promtool check config`/`promtool check rules` before applying changes.
- For durable, long-term, multi-cluster metrics storage, use remote write to Thanos/Mimir/Cortex rather than growing local retention indefinitely or relying on federation as a general-purpose data-centralization mechanism.
- Tune `for:` durations on alert rules deliberately to balance fast detection against avoiding noise from brief, self-resolving spikes.

---

## Summary Cheat-Sheet: Most-Used Prometheus Concepts & Commands

```bash
# Validate configuration
promtool check config prometheus.yml
promtool check rules rules.yml

# Reload config without restart
curl -X POST http://localhost:9090/-/reload

# Check TSDB stats
curl http://localhost:9090/api/v1/status/tsdb

# Common PromQL patterns
rate(http_requests_total[5m])
sum(rate(http_requests_total[5m])) by (job)
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
predict_linear(node_filesystem_free_bytes[6h], 4*3600)

# Alertmanager silence
amtool silence add alertname="HighErrorRate" --duration="2h" --comment="Maintenance"

# API queries
curl -G http://localhost:9090/api/v1/query --data-urlencode 'query=up'
```

---
