# 🔧 Datadog — Troubleshooting Guide
> Common Issues | Fixes | Debug Commands | Root Cause Analysis

---

## 📌 How to Use This Guide

```
Problem happening?
        │
        ▼
Find your category below:
  ├── Agent Issues         → Section 1
  ├── Metrics Not Showing  → Section 2
  ├── Logs Not Appearing   → Section 3
  ├── APM / Trace Issues   → Section 4
  ├── Kubernetes Issues    → Section 5
  ├── Dashboard Problems   → Section 6
  ├── Alert / Monitor Issues → Section 7
  └── Integration Issues  → Section 8
```

---

## 🐾 Section 1: Agent Issues

### ❌ Problem 1: Agent Not Starting

**Symptoms:**
```bash
sudo systemctl status datadog-agent
# ● datadog-agent.service - Datadog Agent
#    Loaded: loaded
#    Active: failed (Result: exit-code)
```

**Root Causes & Fixes:**

```bash
# ── Check 1: API Key valid? ──────────────────────────
sudo datadog-agent diagnose | grep -i "api"
# "API Key check: FAIL" = wrong key

# Fix:
sudo nano /etc/datadog-agent/datadog.yaml
# Find: api_key:
# Replace with correct key from Datadog console

sudo systemctl restart datadog-agent

# ── Check 2: Config file syntax error ────────────────
sudo datadog-agent configcheck
# Shows any YAML syntax errors

# Common mistake: wrong indentation in datadog.yaml
# Fix: Use a YAML validator before saving

# ── Check 3: Port already in use ─────────────────────
sudo lsof -i :8125    # DogStatsD port
sudo lsof -i :8126    # APM port

# Kill conflicting process:
sudo kill -9 <PID>
sudo systemctl restart datadog-agent

# ── Check 4: Insufficient permissions ────────────────
ls -la /etc/datadog-agent/datadog.yaml
# Should be owned by dd-agent user

sudo chown dd-agent:dd-agent /etc/datadog-agent/datadog.yaml
sudo chmod 640 /etc/datadog-agent/datadog.yaml
sudo systemctl restart datadog-agent
```

---

### ❌ Problem 2: Agent Running But No Data in Datadog Console

**Symptoms:** Agent status shows Running, but host not appearing in Datadog UI after 5+ minutes.

```bash
# ── Check 1: Agent connectivity to Datadog ───────────
sudo datadog-agent diagnose connectivity-datadog

# Expected: "Connectivity to Datadog: OK"
# If FAIL: network or firewall issue

# ── Check 2: DNS resolution ───────────────────────────
nslookup app.datadoghq.com
ping -c 3 app.datadoghq.com

# ── Check 3: Outbound port 443 open? ─────────────────
curl -v https://app.datadoghq.com/api/v1/validate?api_key=<YOUR_KEY>
# Should return: {"valid": true}

# ── Check 4: Proxy blocking? ─────────────────────────
# Add proxy to /etc/datadog-agent/datadog.yaml:
proxy:
  https: http://proxy.company.com:8080

# ── Check 5: EC2 Security Group outbound rules ────────
# AWS Console → EC2 → Security Group → Outbound Rules
# Must allow: HTTPS (443) to 0.0.0.0/0
```

---

### ❌ Problem 3: Agent High CPU / Memory Usage

**Symptoms:** `datadog-agent` process using 20%+ CPU or 500MB+ RAM.

```bash
# ── Check 1: How many checks are running? ────────────
sudo datadog-agent status | grep "Running Checks"

# ── Check 2: Which check is slow? ────────────────────
sudo datadog-agent check <check_name> --check-rate

# ── Fix: Increase check interval ─────────────────────
# /etc/datadog-agent/conf.d/nginx.d/conf.yaml
init_config:
instances:
  - nginx_status_url: http://localhost:81/nginx_status
    min_collection_interval: 60    # was 15 (default)

# ── Fix: Disable unnecessary checks ──────────────────
sudo ls /etc/datadog-agent/conf.d/
# Comment out or remove checks you don't need

# ── Fix: Reduce log collection ────────────────────────
# /etc/datadog-agent/datadog.yaml
logs_config:
  processing_rules:
    - type: exclude_at_match
      name: exclude_debug
      pattern: DEBUG    # Skip DEBUG logs to reduce volume
```

---

### ❌ Problem 4: Agent Not Sending Custom Metrics (DogStatsD)

```bash
# ── Check 1: DogStatsD enabled? ─────────────────────
# /etc/datadog-agent/datadog.yaml
use_dogstatsd: true
dogstatsd_port: 8125

# ── Check 2: Test DogStatsD manually ─────────────────
echo -n "custom.metric:1|c|#env:prod" | nc -u -w1 127.0.0.1 8125

# ── Check 3: App sending to correct host/port? ────────
# Non-containerized: use 127.0.0.1:8125
# Kubernetes: use hostIP or datadog-agent service ClusterIP

# Kubernetes: Get DogStatsD address for pods
kubectl get svc -n datadog | grep dogstatsd

# In pod env vars:
env:
  - name: DD_AGENT_HOST
    valueFrom:
      fieldRef:
        fieldPath: status.hostIP
  - name: DD_DOGSTATSD_PORT
    value: "8125"
```

---

## 📊 Section 2: Metrics Not Showing

### ❌ Problem 5: Host Metrics Missing (CPU, Memory, etc.)

```bash
# ── Check 1: Which metrics are being sent? ────────────
sudo datadog-agent check system-core
sudo datadog-agent check cpu
sudo datadog-agent check memory
sudo datadog-agent check disk

# ── Check 2: Is /proc accessible? ────────────────────
ls /proc/meminfo   # Should exist
ls /proc/stat      # Should exist

# Docker: proc must be mounted
docker run ... -v /proc/:/host/proc/:ro ...

# ── Check 3: Disk check excluding paths ───────────────
# Add in /etc/datadog-agent/conf.d/disk.d/conf.yaml:
instances:
  - use_mount: false
    excluded_filesystems:
      - tmpfs
      - devtmpfs
```

---

### ❌ Problem 6: AWS Metrics Not Appearing

```bash
# ── Check 1: IAM Role has correct permissions? ────────
# Required AWS policy for Datadog integration:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudwatch:GetMetricData",
        "cloudwatch:ListMetrics",
        "ec2:DescribeInstances",
        "ec2:DescribeInstanceStatus",
        "tag:GetResources"
      ],
      "Resource": "*"
    }
  ]
}

# ── Check 2: Integration enabled in Datadog? ─────────
# Datadog Console → Integrations → AWS → Configure
# Check: All regions selected?
# Check: CloudWatch Metric Streams or API polling?

# ── Check 3: CloudWatch metrics delay? ───────────────
# AWS CloudWatch has 5-15 min delay for some metrics
# Check the metric timestamp, not just "no data"

# ── Fix: Enable CloudWatch Metric Streams (faster) ────
# Datadog → Integrations → AWS → Metric Streams tab
# Faster than API polling (near real-time)
```

---

## 📝 Section 3: Logs Not Appearing

### ❌ Problem 7: Logs Not Showing in Log Explorer

```bash
# ── Check 1: Log collection enabled? ─────────────────
# /etc/datadog-agent/datadog.yaml
logs_enabled: true    # Must be true!

# ── Check 2: Log config file correct? ─────────────────
# Example: /etc/datadog-agent/conf.d/nginx.d/conf.yaml
logs:
  - type: file
    path: /var/log/nginx/access.log
    service: nginx
    source: nginx
    tags:
      - env:prod

# ── Check 3: File permissions ─────────────────────────
ls -la /var/log/nginx/access.log
# dd-agent user must have read permission!

sudo usermod -a -G adm dd-agent    # Add dd-agent to adm group
sudo systemctl restart datadog-agent

# ── Check 4: Test log collection ─────────────────────
sudo datadog-agent check logs
# Shows: "Found X file paths matching"

# ── Check 5: Log file exists and has content? ─────────
tail -f /var/log/nginx/access.log
# If empty → nothing to send!

# ── Check 6: Wildcard paths ───────────────────────────
# Wrong:  path: /var/log/app-*.log
# Right:  path: /var/log/app*.log   (no dash before wildcard)
```

---

### ❌ Problem 8: Kubernetes Pod Logs Not Collected

```bash
# ── Check 1: logs containerCollectAll enabled? ────────
# datadog-values.yaml
datadog:
  logs:
    enabled: true
    containerCollectAll: true   # Must be true!

# ── Check 2: Agent has access to /var/log/pods ────────
kubectl exec -it <datadog-agent-pod> -n datadog -- ls /var/log/pods
# Should show pod log directories

# ── Check 3: Check agent logs for errors ─────────────
kubectl logs <datadog-agent-pod> -n datadog | grep -i "log"
kubectl logs <datadog-agent-pod> -n datadog | grep -i "error"

# ── Check 4: Add annotations to specific pod ─────────
# If not using containerCollectAll, add to pod spec:
annotations:
  ad.datadoghq.com/my-container.logs: |
    [{"source": "my-app", "service": "my-service"}]

# ── Check 5: Verify agent is DaemonSet (on every node) ─
kubectl get daemonset -n datadog
# DESIRED == READY count?
# If not: some nodes not getting agent!
```

---

## 🔍 Section 4: APM / Trace Issues

### ❌ Problem 9: Traces Not Appearing in APM

```bash
# ── Check 1: APM enabled in agent? ───────────────────
# datadog.yaml
apm_config:
  enabled: true
  apm_non_local_traffic: true   # Required for containers!

# ── Check 2: App sending traces to correct endpoint? ──
# Default: localhost:8126 (same host)
# Docker/K8s: Use DD_AGENT_HOST env variable

# In application code (Python example):
from ddtrace import tracer
tracer.configure(hostname='<DD_AGENT_HOST>', port=8126)

# ── Check 3: Environment variables set? ───────────────
echo $DD_AGENT_HOST    # Should be set
echo $DD_TRACE_AGENT_PORT   # Should be 8126

# ── Check 4: Test APM port connectivity ───────────────
curl http://localhost:8126/info
# Should return JSON with agent info

# ── Check 5: Library instrumented? ───────────────────
# Python:
pip install ddtrace
ddtrace-run python app.py    # Auto-instrumentation

# Node.js:
npm install dd-trace
require('dd-trace').init()   # Must be FIRST line!

# ── Check 6: Service name set? ────────────────────────
# Without this, traces appear as "unnamed-service"
DD_SERVICE=my-api DD_ENV=prod DD_VERSION=1.0 ddtrace-run python app.py
```

---

## ☸️ Section 5: Kubernetes Issues

### ❌ Problem 10: Datadog Agent Pod in CrashLoopBackOff

```bash
# ── Check 1: View agent logs ─────────────────────────
kubectl logs <datadog-agent-pod> -n datadog
kubectl logs <datadog-agent-pod> -n datadog --previous  # Last crash

# ── Check 2: Secret exists? ───────────────────────────
kubectl get secret datadog-secret -n datadog
kubectl describe secret datadog-secret -n datadog
# Keys: api-key, app-key should exist

# Fix: Recreate secret
kubectl delete secret datadog-secret -n datadog
kubectl create secret generic datadog-secret \
  --from-literal api-key=<YOUR_API_KEY> \
  --from-literal app-key=<YOUR_APP_KEY> \
  -n datadog

# Restart agent
kubectl rollout restart daemonset datadog-agent -n datadog

# ── Check 3: Resource limits too low? ────────────────
kubectl describe pod <datadog-agent-pod> -n datadog | grep -A5 "Limits"
# OOMKilled? Increase memory limits in values.yaml:

agents:
  containers:
    agent:
      resources:
        limits:
          memory: 512Mi    # Increase from default 256Mi
          cpu: 400m
```

---

### ❌ Problem 11: Pod Metrics Missing in Datadog

```bash
# ── Check 1: Cluster Agent running? ──────────────────
kubectl get pods -n datadog | grep cluster-agent
# Should be Running

# ── Check 2: RBAC permissions correct? ────────────────
kubectl auth can-i list pods --as=system:serviceaccount:datadog:datadog-agent
# Should return: yes

# Fix: Apply ClusterRole binding
kubectl apply -f https://raw.githubusercontent.com/DataDog/datadog-agent/main/Dockerfiles/manifests/rbac/clusterrole.yaml

# ── Check 3: kubelet TLS issue? ──────────────────────
# In datadog-values.yaml:
datadog:
  kubelet:
    tlsVerify: false    # Disable TLS verification if self-signed cert

# ── Check 4: Check kube-state-metrics ────────────────
kubectl get pods -n kube-system | grep kube-state-metrics
# Missing? Install it:
helm install kube-state-metrics prometheus-community/kube-state-metrics \
  -n kube-system
```

---

## 📊 Section 6: Dashboard Problems

### ❌ Problem 12: Dashboard Shows "No Data"

```bash
# ── Check 1: Time range mismatch? ────────────────────
# Click the time selector in top right
# Try: "Past 1 hour" → "Past 1 day"
# Agent might have just been installed — no historical data yet

# ── Check 2: Wrong metric name? ───────────────────────
# Click on widget → Edit → Check metric name
# Common typo: system.cpu.utilization vs system.cpu.user

# List available metrics:
# Metrics Explorer → Search for metric name
# OR
curl -X GET "https://api.datadoghq.com/api/v1/metrics" \
  -H "DD-API-KEY: <API_KEY>" \
  -H "DD-APPLICATION-KEY: <APP_KEY>"

# ── Check 3: Wrong scope/tag filter? ─────────────────
# Widget might filter by: env:prod
# But your host has tag: env:production (mismatch!)
# Fix: Normalize tags across all hosts

# ── Check 4: Template variable not set? ───────────────
# If dashboard uses $env template variable
# Make sure a value is selected in the dropdown at top
```

---

## 🔔 Section 7: Alert / Monitor Issues

### ❌ Problem 13: Monitor Always in Alert State (Flapping)

```bash
# Symptoms: Alert fires → recovers → fires again (repeatedly)

# ── Fix 1: Increase evaluation window ────────────────
# Monitor Edit → Evaluation → Window: 5 min → 15 min
# Prevents alerting on single spike

# ── Fix 2: Add smoothing with rollup ─────────────────
# Query: avg:system.cpu.user{*}.rollup(avg, 300)
# Averages over 5-minute windows

# ── Fix 3: Enable renotification wisely ───────────────
# Monitor → No Data / Recovery settings
# Renotify: Never (avoid repeated pages)

# ── Fix 4: Composite monitor ─────────────────────────
# Only alert when BOTH CPU > 90% AND Memory > 85%
# Reduces false positives
```

---

### ❌ Problem 14: No Alert Notifications Being Received

```bash
# ── Check 1: Notification handle correct? ────────────
# In monitor message:
@slack-alerts-channel    # Slack
@pagerduty-oncall        # PagerDuty
@email@company.com       # Email

# ── Check 2: Integration connected? ──────────────────
# Datadog → Integrations → Slack
# Test the integration!

# ── Check 3: Monitor in muted state? ─────────────────
# Monitors → Select monitor → Check if "Muted" badge visible
# Unmute: Actions → Unmute

# ── Check 4: Downtime scheduled? ─────────────────────
# Monitors → Manage Downtimes
# Is there an active downtime covering this monitor?
```

---

## 🔌 Section 8: Integration Issues

### ❌ Problem 15: Nginx Integration Not Working

```bash
# ── Step 1: Enable nginx stub_status ─────────────────
# /etc/nginx/nginx.conf
server {
  listen 81;
  location /nginx_status {
    stub_status on;
    allow 127.0.0.1;
    deny all;
  }
}
sudo nginx -t    # Test config
sudo nginx -s reload

# ── Step 2: Verify stub_status works ─────────────────
curl http://localhost:81/nginx_status
# Output should show:
# Active connections: 5
# server accepts handled requests
#  10 10 15

# ── Step 3: Configure Datadog integration ─────────────
sudo nano /etc/datadog-agent/conf.d/nginx.d/conf.yaml
# Content:
init_config:
instances:
  - nginx_status_url: http://localhost:81/nginx_status
    tags:
      - env:prod
      - service:nginx

# ── Step 4: Restart and test ─────────────────────────
sudo systemctl restart datadog-agent
sudo datadog-agent check nginx
# Should show metrics like: nginx.net.connections
```

---

## 🛠️ Master Debug Commands Reference

```bash
# ─── Agent Health ──────────────────────────────────────
sudo datadog-agent status                    # Full agent status
sudo datadog-agent health                    # Quick health check
sudo datadog-agent version                   # Agent version
sudo datadog-agent diagnose                  # Run all diagnostics

# ─── Connectivity ──────────────────────────────────────
sudo datadog-agent diagnose connectivity-datadog   # Can reach Datadog?
sudo datadog-agent diagnose port-binding           # Ports in use?

# ─── Configuration ─────────────────────────────────────
sudo datadog-agent configcheck               # Config syntax valid?
sudo datadog-agent config                    # Show full resolved config
sudo datadog-agent flare                     # Generate support bundle

# ─── Checks ────────────────────────────────────────────
sudo datadog-agent check <check_name>        # Run specific check
sudo datadog-agent check nginx               # Test nginx check
sudo datadog-agent check cpu                 # Test CPU check
sudo datadog-agent check docker              # Test Docker check

# ─── Logs ──────────────────────────────────────────────
sudo tail -f /var/log/datadog/agent.log      # Agent log
sudo tail -f /var/log/datadog/dogstatsd.log  # DogStatsD log
sudo tail -f /var/log/datadog/trace-agent.log # APM agent log

# ─── Kubernetes ────────────────────────────────────────
kubectl get pods -n datadog                          # All pods
kubectl logs <agent-pod> -n datadog                  # Agent logs
kubectl logs <agent-pod> -n datadog -c agent         # Specific container
kubectl exec -it <agent-pod> -n datadog -- agent status  # Status in pod
kubectl describe pod <agent-pod> -n datadog          # Pod events
kubectl get events -n datadog --sort-by='.lastTimestamp'  # K8s events

# ─── Docker ────────────────────────────────────────────
docker exec -it datadog-agent agent status          # Agent in container
docker exec -it datadog-agent agent check docker    # Docker check
docker logs datadog-agent                            # Container logs
docker logs datadog-agent --tail 100                 # Last 100 lines
```

---

## 📋 Quick Troubleshooting Decision Tree

```
Agent installed but no data in Datadog?
    │
    ├─ Agent running?
    │   ├── NO  → Check: systemctl status datadog-agent
    │   │         Fix: systemctl restart datadog-agent
    │   │         Check logs: /var/log/datadog/agent.log
    │   │
    │   └── YES → Check connectivity:
    │             datadog-agent diagnose connectivity-datadog
    │             ├── FAIL → Check firewall/SG (outbound 443)
    │             └── OK   → Check API key in datadog.yaml
    │
Logs not appearing?
    │
    ├─ logs_enabled: true in datadog.yaml?
    │   └── NO → Add it, restart agent
    │
    ├─ conf.yaml file exists in conf.d/app.d/?
    │   └── NO → Create it with type/path/service/source
    │
    └─ File permissions?
        └── dd-agent can't read log file?
            Fix: sudo usermod -a -G adm dd-agent

K8s pods not monitored?
    │
    ├─ DaemonSet all nodes running?
    │   └── NO → Check: kubectl get daemonset -n datadog
    │
    ├─ Cluster Agent running?
    │   └── NO → Check: kubectl get deploy -n datadog
    │
    └─ RBAC correct?
        └── Test: kubectl auth can-i list pods \
                    --as=system:serviceaccount:datadog:datadog-agent
```

---
