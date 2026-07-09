# Prometheus Troubleshooting Guide

---

## Table of Contents

1. [General Troubleshooting Methodology](#module-1-general-troubleshooting-methodology)
2. [Installation & Startup Issues](#module-2-installation--startup-issues)
3. [Scrape & Target Issues](#module-3-scrape--target-issues)
4. [PromQL Query Issues](#module-4-promql-query-issues)
5. [Storage & TSDB Issues](#module-5-storage--tsdb-issues)
6. [Alerting Rule Issues](#module-6-alerting-rule-issues)
7. [Alertmanager Issues](#module-7-alertmanager-issues)
8. [Federation & Remote Write Issues](#module-8-federation--remote-write-issues)
9. [Cardinality & Performance Issues](#module-9-cardinality--performance-issues)
10. [Service Discovery & Relabeling Issues](#module-10-service-discovery--relabeling-issues)
11. [Exporter Issues](#module-11-exporter-issues)
12. [High Availability / Thanos Issues](#module-12-high-availability--thanos-issues)

---

## MODULE 1: General Troubleshooting Methodology

### 1.1 The Layered Troubleshooting Approach

```
Symptom reported (missing data, alert not firing, slow queries, etc.)
        │
1. Check Prometheus's own /targets page (is the target even being scraped successfully?)
        │
2. Check the raw metric directly (query the metric name alone in the Expression Browser)
        │
3. Check the specific query logic (aggregations, label matchers, functions)
        │
4. Check Prometheus server logs
        │
5. Check resource utilization (memory, disk, cardinality stats)
        │
6. Fix, validate with promtool, reload/restart if needed
```

---

### 1.2 Essential Diagnostic Endpoints and Commands

```bash
curl http://localhost:9090/-/healthy               # basic liveness check
curl http://localhost:9090/-/ready                   # readiness check (TSDB loaded, etc.)
curl http://localhost:9090/api/v1/targets             # target scrape status
curl http://localhost:9090/api/v1/status/tsdb          # TSDB stats (series count, cardinality)
curl http://localhost:9090/api/v1/rules                 # rule evaluation status
curl http://localhost:9090/api/v1/alerts                 # currently active alerts
promtool check config prometheus.yml
promtool check rules rules.yml
```

---

## MODULE 2: Installation & Startup Issues

### 2.1 Prometheus Fails to Start — "error loading config"

```bash
promtool check config prometheus.yml
```
Always validate the config file with `promtool` before attempting to start/restart — YAML indentation errors or invalid field names are the most common cause, and `promtool` pinpoints the exact issue far more clearly than Prometheus's own startup error output.

---

### 2.2 "opening storage failed: ... lock DB directory"

```bash
ps aux | grep prometheus
```
Indicates another Prometheus process is already running and holding a lock on the same data directory — check for a duplicate/zombie process, or confirm you're not accidentally pointing two separate Prometheus instances at the same `--storage.tsdb.path`.

---

### 2.3 Prometheus Starts but Immediately Uses Very High Memory/CPU

```bash
curl http://localhost:9090/api/v1/status/tsdb
```
Common cause: replaying a large Write-Ahead Log (WAL) after an unclean shutdown — this is expected, one-time startup behavior and should stabilize once WAL replay completes; if it persists indefinitely, check for a genuinely oversized WAL (very high cardinality, or retention/compaction issues) rather than assuming a bug.

---

### 2.4 Docker Container Exits Immediately

```bash
docker logs prometheus
```
Common causes: an invalid mounted `prometheus.yml` (check with `promtool` first), incorrect volume permissions on the mounted data directory, or a missing/incorrectly-referenced rule file path specified in the config.

---

### 2.5 Web UI Loads but Shows No Targets at All

```bash
curl http://localhost:9090/api/v1/targets
```
Confirm the `scrape_configs` section in `prometheus.yml` actually has entries, and check whether a `--config.file` flag was accidentally omitted at startup (causing Prometheus to fall back to a default, empty configuration rather than your intended file).

---

## MODULE 3: Scrape & Target Issues

### 3.1 Target Shows as "DOWN" on the /targets Page

```bash
curl http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | select(.health=="down")'
```
Check the specific error message shown for that target (visible in the UI or API response) — common causes: the target's `/metrics` endpoint is unreachable (network/firewall), the target process itself isn't running, or a TLS/authentication misconfiguration if the target requires HTTPS/credentials.

---

### 3.2 Target Is "UP" but Metrics Are Missing/Stale

```bash
curl http://target-host:9100/metrics | grep specific_metric_name
```
Query the target's `/metrics` endpoint directly (bypassing Prometheus entirely) to confirm whether the metric actually exists at the source — if it's missing there too, the issue is with the exporter/application itself, not Prometheus's scraping.

---

### 3.3 Scrape Timeout Errors ("context deadline exceeded")

```yaml
scrape_configs:
  - job_name: 'slow-target'
    scrape_timeout: 30s    # increase from default 10s if the target genuinely needs more time
```
Check whether the target's `/metrics` endpoint is simply slow to respond (perhaps due to computing many metrics on-demand per scrape) — increase `scrape_timeout` for that specific job if needed, but also consider whether the target's own metrics-generation logic needs optimization rather than just extending the timeout indefinitely.

---

### 3.4 "duplicate sample for timestamp" Errors

```
Two scrapes producing the same label set with different values at effectively the same timestamp
```
Common causes: two Prometheus instances (misconfigured to be independent) accidentally scraping and both trying to write via the same remote_write path, or a target application inadvertently exposing the same metric/label combination twice within a single scrape response (a bug in the target's own metrics exposition, not a Prometheus issue).

---

### 3.5 Scrape Successful but "up" Metric Shows 0 Intermittently

```bash
curl http://localhost:9090/api/v1/targets
```
Check `lastError` and `lastScrapeDuration` for that target in the API response — intermittent failures often point to target-side resource exhaustion (the target briefly can't respond under its own load), network flakiness between Prometheus and the target, or the target exceeding `scrape_timeout` only under specific load conditions.

---

### 3.6 A Newly Added Kubernetes Pod Doesn't Appear as a Scrape Target

```bash
curl http://localhost:9090/api/v1/targets | jq '.data.droppedTargets'
```
Check `droppedTargets` specifically (not just `activeTargets`) — pods are commonly dropped by a `relabel_configs` filter (e.g., requiring a specific `prometheus.io/scrape: "true"` annotation) that the new pod simply doesn't have configured; confirm the pod's annotations match what the relabeling rules expect.

---

## MODULE 4: PromQL Query Issues

### 4.1 Query Returns "No Data"

```promql
http_requests_total    # first check the raw metric alone, with no filters/functions
```
Start with the simplest possible version of the query (just the bare metric name) and incrementally add back filters/functions one at a time — this quickly isolates whether the issue is a label matcher that doesn't actually match anything, a time range with no data, or a function/aggregation causing the empty result.

---

### 4.2 Graphing a Counter Shows a Strange, Sawtooth/Negative-Looking Pattern

```promql
# WRONG — raw counter, will show resets as sharp drops
http_requests_total

# CORRECT — rate() correctly handles counter resets
rate(http_requests_total[5m])
```
This is expected behavior when graphing a raw counter directly instead of using `rate()`/`increase()` — a counter resets to 0 whenever its process restarts, and `rate()` is specifically designed to detect and correctly handle these resets, producing a sensible rate rather than an apparent negative dip.

---

### 4.3 `histogram_quantile()` Returns Unexpected/Wrong Values

```promql
# Missing the `by (le)` grouping when aggregating across instances — WRONG
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# CORRECT — must preserve `le` label through the aggregation
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
```
Forgetting to group by `le` (or accidentally dropping it) when aggregating histogram buckets across multiple series is one of the most common `histogram_quantile()` mistakes — without preserving `le`, the function can't correctly reconstruct the cumulative bucket structure needed for the quantile calculation.

---

### 4.4 "vector cannot contain metrics with the same labelset" Error

```promql
# Two series that become identical after a by()/without() aggregation collapses distinguishing labels
sum(metric_a) by (job) + sum(metric_b) by (job)
```
This typically occurs during vector matching (binary operations between two vectors) when the label sets aren't unique after the aggregation as expected — review exactly which labels remain after your `by`/`without` clause, and confirm both sides of the operation have genuinely matching, unique label sets for the intended one-to-one (or explicitly declared many-to-one via `group_left`/`group_right`) matching.

---

### 4.5 Query Works in the Expression Browser but Fails/Errors as an Alert Rule Condition

```
promtool check rules rules.yml
```
Alert rule expressions are evaluated the same way as any instant query — if it works in the Expression Browser but the rule file itself has an error, it's typically a YAML syntax issue in how the expression is embedded in the rule file (e.g., missing quotes around a colon-containing expression), not a fundamental PromQL difference; validate with `promtool check rules` to catch this class of issue.

---

### 4.6 Slow Queries Timing Out in the UI or via Grafana

```promql
# Expensive: raw high-cardinality aggregation computed fresh on every query
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, path, method, instance))

# Consider: reduce the grouping labels to only what's genuinely needed, or use a recording rule
```
Check whether the query is aggregating across an unnecessarily high number of label dimensions (each additional `by()` label multiplies the amount of data Prometheus must process) — reduce to only genuinely needed grouping labels, and for frequently-run expensive queries, define a recording rule to pre-compute the result instead.

---

## MODULE 5: Storage & TSDB Issues

### 5.1 Disk Filling Up Rapidly

```bash
curl http://localhost:9090/api/v1/status/tsdb
du -sh /prometheus-data/*
```
Check `--storage.tsdb.retention.time`/`.size` settings (defaults may be more generous than intended for your available disk), and check for a cardinality explosion (a sudden spike in unique time series) causing disproportionate storage growth — the TSDB stats endpoint's top-cardinality-metrics output quickly identifies a runaway metric/label combination.

---

### 5.2 "out of order sample" Errors

```
Rejected sample with a timestamp older than the most recent sample already stored for that series
```
Common causes: a target's clock is out of sync (NTP drift) relative to Prometheus's own clock, or (in remote-write scenarios) samples arriving out of their original order due to network retries/buffering — verify clock synchronization across all involved systems, and check for genuinely duplicate/conflicting scrape configurations targeting the same series.

---

### 5.3 Compaction Falling Behind / High Disk I/O from Compaction

```bash
curl http://localhost:9090/api/v1/status/tsdb
```
Check for a high number of small, uncompacted blocks accumulating — often correlates with high cardinality (more unique series means more chunks to compact) or genuinely insufficient disk I/O throughput for the ingest rate; consider whether the underlying storage (especially in cloud environments with provisioned IOPS limits) is adequately sized for your actual scrape volume.

---

### 5.4 Prometheus Restart Takes a Very Long Time (WAL Replay)

```bash
tail -f prometheus.log    # watch for WAL replay progress messages
```
A large WAL (from a high ingestion rate, or having accumulated during a period the compactor couldn't keep up) takes proportionally longer to replay on startup — this is generally expected recovery behavior after an unclean shutdown, though persistently very slow restarts may indicate an underlying cardinality or disk performance issue worth investigating separately.

---

### 5.5 "context deadline exceeded" or Query Errors Specifically During Compaction

Compaction is I/O and CPU intensive and can transiently compete with query serving for resources — if this is a recurring, noticeable problem, it often points to genuinely undersized infrastructure (disk IOPS, CPU) for the current ingestion/cardinality volume, warranting either resource scaling or a move toward externalizing long-term storage (Thanos/Mimir) to reduce the burden on a single Prometheus instance.

---

## MODULE 6: Alerting Rule Issues

### 6.1 Alert Rule Never Fires Despite the Condition Appearing True

```bash
curl http://localhost:9090/api/v1/rules | jq '.data.groups[].rules[] | select(.name=="MyAlert")'
```
Check the rule's actual current state and any evaluation error directly via the API — confirm the `for:` duration hasn't simply not yet elapsed (the rule may correctly be in "pending" rather than "firing"), and independently verify the alert expression itself returns the expected result when run directly in the Expression Browser.

---

### 6.2 Alert Rule Shows a Persistent Evaluation Error

```bash
curl http://localhost:9090/api/v1/rules | jq '.data.groups[].rules[] | select(.health=="err")'
```
Common causes: a PromQL syntax error in the rule expression, a referenced metric that doesn't actually exist (typo in the metric name), or a query that's timing out due to being too expensive — test the exact expression manually in the Expression Browser to isolate the specific failure.

---

### 6.3 Alert Fires But the Annotation/Label Templating Shows Raw `{{ }}` Text Instead of Actual Values

```yaml
annotations:
  summary: "High error rate on {{ $labels.job }}"    # confirm correct Go template syntax
```
Check for a templating syntax error (missing/mismatched braces, incorrect variable reference like `$labels.jobs` instead of `$labels.job`) — Prometheus's alert templating uses Go's `text/template` syntax, and a malformed template typically fails silently by rendering the raw unprocessed text rather than throwing an obvious error.

---

### 6.4 Recording Rule Not Producing the Expected Metric

```bash
curl -G http://localhost:9090/api/v1/query --data-urlencode 'query=job:http_requests:rate5m'
promtool check rules recording_rules.yml
```
Confirm the rule file is actually being loaded (check `rule_files:` in `prometheus.yml` and that the file path/glob pattern matches), and validate the rule's syntax with `promtool` — a rule file with a YAML error may fail to load entirely without an obviously visible startup failure depending on Prometheus version/configuration.

---

## MODULE 7: Alertmanager Issues

### 7.1 Alert Reaches Alertmanager but No Notification Is Sent

```bash
curl http://localhost:9093/api/v2/alerts
amtool config show
```
Check whether the alert's labels actually match a route in the routing tree (an alert with labels not matching any specific route falls through to the `route.receiver` default — confirm that's actually configured with valid receiver details), and check the specific receiver's configuration (correct webhook URL, valid credentials) for the destination it's meant to reach.

---

### 7.2 Duplicate Notifications for the Same Alert

```yaml
route:
  group_by: ['alertname', 'cluster']    # ensure grouping is configured sensibly
```
Check whether multiple independent Prometheus instances are both firing the same alert without Alertmanager clustering configured to deduplicate between them (each Alertmanager instance needs to be part of the same cluster/mesh, not running in isolation), and verify `group_by` settings are correctly grouping related alerts rather than treating each individually.

---

### 7.3 Silences Not Suppressing the Intended Alerts

```bash
amtool silence query
```
Confirm the silence's label matchers actually align with the firing alert's exact labels (an overly narrow or slightly mismatched matcher won't suppress anything), and check the silence hasn't already expired if using a duration-based silence rather than an explicit end time.

---

### 7.4 Inhibition Rule Not Suppressing Lower-Severity Alerts as Expected

```yaml
inhibit_rules:
  - source_match: { severity: critical }
    target_match: { severity: warning }
    equal: ['alertname', 'cluster']   # BOTH alerts must match on these labels EXACTLY
```
Verify the `equal` list's specified labels genuinely match between the source (critical) and target (warning) alerts — if the two alerts have even slightly different values for any label listed in `equal` (e.g., a different `instance` label), the inhibition won't apply, since it requires an exact match on all `equal`-listed labels between the two.

---

### 7.5 Alertmanager Cluster Members Not Seeing Each Other's Alerts

```bash
curl http://localhost:9093/api/v2/status
```
Check the cluster peer status via the API — common causes include network connectivity issues between Alertmanager instances on the gossip port (default 9094), or incorrect `--cluster.peer` flags at startup not correctly listing all intended peer addresses.

---

## MODULE 8: Federation & Remote Write Issues

### 8.1 Federated Metrics Missing Expected Labels

```yaml
scrape_configs:
  - job_name: 'federate'
    honor_labels: true    # critical — preserves original labels from the federated source
```
Forgetting `honor_labels: true` causes the federating (global) Prometheus to potentially overwrite the source's own labels with its own scrape-derived labels — almost always undesired in a federation setup, where you want to preserve the original instance/job labels from the source Prometheus.

---

### 8.2 Remote Write Falling Behind / Queue Growing

```bash
curl http://localhost:9090/metrics | grep prometheus_remote_storage_
```
Check `prometheus_remote_storage_samples_pending` and related queue metrics — a growing backlog indicates the remote endpoint (Thanos/Mimir/Cortex receiver) can't keep up with the ingestion rate, or network issues between Prometheus and the remote endpoint; consider tuning `queue_config` (batch size, parallelism) or investigating the remote endpoint's own capacity.

---

### 8.3 Remote Write Authentication Failures

```yaml
remote_write:
  - url: "https://mimir.mycompany.com/api/v1/push"
    basic_auth:
      username: prometheus
      password_file: /etc/prometheus/remote_write_password
```
Check for expired/rotated credentials, and confirm the remote endpoint's expected authentication method (basic auth, bearer token, mTLS) matches exactly what's configured — check Prometheus's own logs for the specific HTTP error code returned by the remote endpoint (401 vs 403 vs connection refused each point to different root causes).

---

### 8.4 Data Visible Locally but Missing from the Remote Long-Term Storage

```bash
curl http://localhost:9090/metrics | grep prometheus_remote_storage_succeeded_samples_total
```
Confirm remote write is actually succeeding (check the success/failure counters), and separately confirm the remote system's own retention/query configuration isn't excluding the data for a different reason (e.g., a tenant/multi-tenancy misconfiguration in Mimir/Cortex causing writes to land under an unexpected tenant ID).

---

## MODULE 9: Cardinality & Performance Issues

### 9.1 Prometheus Memory Usage Growing Unbounded

```bash
curl http://localhost:9090/api/v1/status/tsdb
topk(10, count by (__name__)({__name__=~".+"}))
```
Check `prometheus_tsdb_head_series` trend over time and the top-cardinality-metrics query — a single misbehaving application exposing a high-cardinality label (like a raw user ID or unbounded URL path) is the most common root cause; identify and fix the source, or use `metric_relabel_configs` to drop the problematic metric/label if you can't immediately fix the source application.

---

### 9.2 A Specific Metric Is Identified as High-Cardinality — How to Mitigate Without Losing All Visibility

```yaml
metric_relabel_configs:
  - source_labels: [__name__]
    regex: 'problematic_metric_name'
    action: drop
```
Options include: working with the application team to fix the label design (best long-term fix), dropping the metric entirely via `metric_relabel_configs` if it's not actually valuable, or aggregating away the problematic label via a recording rule at scrape time isn't directly possible (relabeling operates on raw samples, not aggregations) — the real fix is almost always at the source (application instrumentation), not a Prometheus-side workaround.

---

### 9.3 Query Performance Degrading Over Time as the Instance Ages

```bash
curl http://localhost:9090/api/v1/status/tsdb
```
Check whether cardinality has grown significantly since initial deployment (a common, gradual "cardinality creep" as more applications/labels get added over time without active governance) — periodically audit cardinality growth trends rather than only reacting once performance becomes visibly problematic.

---

### 9.4 High CPU Usage Specifically During Rule Evaluation

```bash
curl http://localhost:9090/api/v1/rules
```
Check for expensive alerting/recording rule expressions (high cardinality aggregations, frequent evaluation intervals) — consider whether `evaluation_interval` is set unnecessarily aggressively for rules that don't genuinely need sub-minute freshness, and whether specific expensive rules could be optimized or their evaluation interval relaxed.

---

## MODULE 10: Service Discovery & Relabeling Issues

### 10.1 Kubernetes Service Discovery Finding No Targets at All

```bash
curl http://localhost:9090/api/v1/targets | jq '.data.droppedTargets | length'
```
Check whether the Prometheus service account has the correct RBAC permissions to list/watch pods/services/endpoints in the Kubernetes API (a common gap in a freshly deployed Prometheus in a new cluster), and confirm the `role:` specified in `kubernetes_sd_configs` (pod, service, endpoints, etc.) matches what you actually intend to discover.

---

### 10.2 Relabeling Rule Not Producing the Expected Result

```bash
curl http://localhost:9090/api/v1/targets   # inspect discoveredLabels vs actual labels post-relabeling
```
Step through the relabel_configs sequence manually against the actual `__meta_*` labels Kubernetes SD provides (visible in the target's "Discovered Labels" in the UI) — a very common mistake is an incorrect regex pattern that doesn't actually match the real label value format, silently causing the relabel action to have no effect.

---

### 10.3 Confusing `relabel_configs` (Pre-Scrape) with `metric_relabel_configs` (Post-Scrape)

If you're trying to drop a specific unwanted metric/sample and it's not working, confirm you're using `metric_relabel_configs` (operates on already-scraped sample data) rather than `relabel_configs` (operates only on target selection before scraping ever happens) — using the wrong one for the intended purpose is a very common source of "my relabeling isn't working" confusion.

---

### 10.4 File-Based Service Discovery Not Picking Up Changes

```yaml
scrape_configs:
  - job_name: 'file_sd'
    file_sd_configs:
      - files: ['/etc/prometheus/targets/*.json']
        refresh_interval: 30s
```
Confirm the target file is valid JSON matching the expected schema (an array of objects with `targets` and optional `labels`), and check `refresh_interval` — file-based SD only re-reads the file periodically, not instantly on every file system change.

---

## MODULE 11: Exporter Issues

### 11.1 node_exporter Shows No Data for a Specific Metric

```bash
curl http://localhost:9100/metrics | grep node_filesystem
```
Some node_exporter collectors are disabled by default or require specific flags to enable (e.g., certain filesystem/mount-point collectors) — check the exporter's actual startup flags/enabled collectors against what metric you expect to see.

---

### 11.2 blackbox_exporter Probe Always Fails

```bash
curl "http://blackbox-exporter:9115/probe?target=https://example.com&module=http_2xx"
```
Test the probe directly against the blackbox_exporter (bypassing Prometheus) to isolate whether the issue is the probe/module configuration itself versus the Prometheus scrape configuration's relabeling setup (a very common source of confusion, since blackbox_exporter's Prometheus integration relies on a somewhat unusual relabeling pattern to pass the actual target URL as a parameter).

---

### 11.3 kube-state-metrics Not Showing Expected Deployment/Pod State

```bash
curl http://kube-state-metrics:8080/metrics | grep kube_deployment
```
Confirm kube-state-metrics has the necessary RBAC permissions to read the relevant Kubernetes API objects, and remember it only reports API-observed state (not resource usage) — if you're looking for actual CPU/memory usage and seeing nothing in kube-state-metrics, that's expected, since that data comes from cAdvisor/metrics-server instead, not kube-state-metrics.

---

### 11.4 Custom Application Exporter (Client Library) Not Exposing Metrics

```bash
curl http://myapp:8080/metrics
```
Confirm the application actually registered the `/metrics` HTTP handler correctly (a common oversight when first instrumenting an application), and check the client library's default metrics port/path matches what Prometheus's scrape config expects.

---

## MODULE 12: High Availability / Thanos Issues

### 12.1 Thanos Sidecar Not Uploading Blocks to Object Storage

```bash
docker logs thanos-sidecar
```
Check the sidecar's object storage credentials/configuration (S3/GCS bucket permissions), and confirm the associated Prometheus instance actually has `--storage.tsdb.min-block-duration` and `--storage.tsdb.max-block-duration` configured appropriately (Thanos sidecar typically expects these to match, e.g., both set to 2h, to align with when blocks become eligible for upload).

---

### 12.2 Thanos Query Showing Duplicate/Doubled Data

```
Check: are multiple Prometheus+sidecar pairs (from an HA pair) both being queried without deduplication configured?
```
Thanos Query needs explicit deduplication configuration (`--query.replica-label`) when querying across HA pairs of identical Prometheus instances — without it, identical data from both replicas appears as duplicated/doubled series in query results.

---

### 12.3 Alertmanager Not Receiving Alerts When Using Multiple Prometheus HA Replicas

Confirm both Prometheus HA replicas are configured to send alerts to the SAME set of Alertmanager instances (and that those Alertmanager instances are properly clustered with each other) — Alertmanager's own deduplication relies on its cluster gossip protocol correctly recognizing that alerts from both Prometheus replicas represent the same underlying condition.

---

### 12.4 Thanos Compactor Falling Behind on Downsampling

```bash
docker logs thanos-compactor
```
Check compactor resource allocation (CPU/memory/disk I/O) relative to the volume of blocks being uploaded from all sidecars — a single compactor instance can become a bottleneck at large scale, and Thanos's compaction is explicitly designed to run as a single instance at a time per storage prefix (not horizontally scaled trivially), so vertical scaling or reducing overall cardinality/block volume may be needed if it's persistently falling behind.

---

## Quick Reference: Most Useful Prometheus Troubleshooting Commands

```bash
# Health & Status
curl http://localhost:9090/-/healthy
curl http://localhost:9090/-/ready
curl http://localhost:9090/api/v1/targets
curl http://localhost:9090/api/v1/status/tsdb

# Config & Rules Validation
promtool check config prometheus.yml
promtool check rules rules.yml

# Alerting
curl http://localhost:9090/api/v1/rules
curl http://localhost:9090/api/v1/alerts
curl http://localhost:9093/api/v2/alerts
amtool silence query

# Cardinality Investigation
topk(10, count by (__name__)({__name__=~".+"}))

# Reload config
curl -X POST http://localhost:9090/-/reload

# Remote Write Health
curl http://localhost:9090/metrics | grep prometheus_remote_storage_
```

---
