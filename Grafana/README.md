# Grafana Complete Study Guide

---

## Table of Contents

1. [Grafana Fundamentals & Architecture](#module-1-grafana-fundamentals--architecture)
2. [Installation & Setup](#module-2-installation--setup)
3. [Data Sources](#module-3-data-sources)
4. [Dashboards & Panels](#module-4-dashboards--panels)
5. [Visualizations & Panel Types](#module-5-visualizations--panel-types)
6. [Variables & Templating](#module-6-variables--templating)
7. [Alerting](#module-7-alerting)
8. [Users, Teams & Permissions (RBAC)](#module-8-users-teams--permissions-rbac)
9. [Plugins](#module-9-plugins)
10. [Provisioning (Dashboards & Data Sources as Code)](#module-10-provisioning-dashboards--data-sources-as-code)
11. [Grafana with Prometheus](#module-11-grafana-with-prometheus)
12. [Grafana with Loki & Tempo (Logs & Traces)](#module-12-grafana-with-loki--tempo-logs--traces)
13. [Grafana Cloud vs Self-Hosted](#module-13-grafana-cloud-vs-self-hosted)
14. [Performance & Best Practices](#module-14-performance--best-practices)

---

## MODULE 1: Grafana Fundamentals & Architecture

### 1.1 What is Grafana?

Grafana is an open-source observability and data visualization platform — it doesn't store metrics/logs/traces itself (in most common setups) but connects to and queries various backend data sources (Prometheus, Loki, Elasticsearch, MySQL, CloudWatch, etc.), rendering the results into dashboards, graphs, and alerts.

---

### 1.2 Grafana Architecture

```
                     Users (Browser)
                          │
                   Grafana Server
        ┌─────────────────┼─────────────────┐
   HTTP Server        Alerting Engine     Provisioning
  (dashboards,        (evaluates rules,   (loads dashboards/
   API, auth)          sends notifications) datasources from
        │                                    config files)
   Grafana's own DB
   (SQLite/MySQL/Postgres —
    stores users, dashboards,
    orgs, alert rules — NOT
    your actual metrics data)
        │
        ▼
   Data Source Plugins
        │
  ┌─────┼─────┬──────────┬───────────┐
Prometheus  Loki      Elasticsearch  CloudWatch / MySQL / etc.
(metrics)  (logs)      (logs/search)   (many other backends)
```

**Critical distinction:** Grafana's own internal database (SQLite by default, or MySQL/PostgreSQL for production) stores Grafana's *configuration* (dashboards, users, alert rules) — it does **not** store your actual time-series metrics or log data, which lives in the connected data sources (Prometheus, Loki, etc.).

---

### 1.3 Key Terminology

| Term | Meaning |
|---|---|
| Data Source | A configured connection to a backend system (Prometheus, MySQL, etc.) |
| Dashboard | A collection of panels arranged on a page |
| Panel | A single visualization (graph, table, gauge) backed by one or more queries |
| Query | The request sent to a data source to retrieve data for a panel |
| Variable | A dynamic, user-selectable placeholder used across a dashboard's queries |
| Organization | A top-level isolation boundary within a single Grafana instance |
| Folder | A way to organize/group dashboards and control permissions on groups of them |

---

### 1.4 Grafana Editions

| Edition | Notes |
|---|---|
| Grafana OSS | Free, open-source, self-hosted |
| Grafana Enterprise | Adds RBAC, reporting, data source permissions, SAML/LDAP enhancements, support |
| Grafana Cloud | Fully managed SaaS, includes hosted Prometheus (Mimir), Loki, Tempo |

---

## MODULE 2: Installation & Setup

### 2.1 Installing Grafana (Linux, APT)

```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install grafana

sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

Default web UI: `http://localhost:3000` — default credentials `admin`/`admin` (forced to change on first login).

---

### 2.2 Installing Grafana via Docker

```bash
docker run -d --name grafana \
  -p 3000:3000 \
  -v grafana-storage:/var/lib/grafana \
  grafana/grafana-oss:latest
```

Always mount a persistent volume for `/var/lib/grafana` — this is where the SQLite database (or configuration for external DB connections) and provisioned assets live; without it, all dashboards/settings are lost when the container is recreated.

---

### 2.3 Key Configuration File

```ini
# /etc/grafana/grafana.ini (or grafana.yaml/environment variables in containerized setups)

[server]
http_port = 3000
root_url = https://grafana.mycompany.com

[database]
type = postgres
host = db.mycompany.com:5432
name = grafana
user = grafana

[security]
admin_user = admin
admin_password = changeme

[auth.anonymous]
enabled = false
```

Configuration can also be set via environment variables using the pattern `GF_<SECTION>_<KEY>` (e.g., `GF_SERVER_ROOT_URL`), which is the standard approach in containerized/Kubernetes deployments.

---

### 2.4 Production Database Recommendation

For any production deployment, configure an external MySQL or PostgreSQL database rather than relying on the default embedded SQLite — SQLite doesn't handle concurrent writes well at scale and complicates high-availability setups (multiple Grafana instances can't safely share a single SQLite file).

---

## MODULE 3: Data Sources

### 3.1 Adding a Data Source

```
Configuration → Data Sources → Add data source → (select type: Prometheus, Loki, MySQL, etc.)
        │
Configure connection URL, authentication, and any type-specific settings
        │
Save & Test (validates connectivity before saving)
```

---

### 3.2 Common Data Source Types

| Type | Use Case |
|---|---|
| Prometheus | Metrics (the most common Grafana + Prometheus pairing) |
| Loki | Logs (Grafana Labs' own log aggregation system) |
| Tempo | Distributed tracing (Grafana Labs' own tracing backend) |
| Elasticsearch | Logs/search |
| InfluxDB | Time-series metrics |
| MySQL/PostgreSQL | Relational/business data visualization |
| CloudWatch | AWS-native metrics |
| Azure Monitor / Google Cloud Monitoring | Cloud-native metrics for respective providers |

---

### 3.3 Default vs Non-Default Data Sources

Exactly one data source per organization can be marked "Default" — new panels use this data source automatically unless explicitly changed, useful for reducing repetitive configuration when most dashboards query the same primary backend (commonly Prometheus).

---

### 3.4 Data Source Permissions (Enterprise Feature)

In Grafana Enterprise, data source-level permissions restrict which teams/users can query a specific data source at all — useful in multi-tenant setups where certain sensitive data sources (e.g., a data source containing PII-adjacent business metrics) should only be queryable by specific authorized teams.

---

## MODULE 4: Dashboards & Panels

### 4.1 Creating a Dashboard

```
+ Create → Dashboard → Add new panel
        │
Select data source, write query, choose visualization type
        │
Configure panel title, axes, thresholds, legend
        │
Save dashboard (choose a Folder for organization)
```

---

### 4.2 Dashboard JSON Model

Every dashboard is fundamentally represented as a JSON document — this is what enables dashboards to be version-controlled, exported/imported, and provisioned as code.

```json
{
  "title": "My Dashboard",
  "panels": [
    {
      "title": "Request Rate",
      "type": "timeseries",
      "targets": [{ "expr": "rate(http_requests_total[5m])" }]
    }
  ],
  "templating": { "list": [] },
  "time": { "from": "now-6h", "to": "now" }
}
```

Understanding the JSON model is essential for advanced use cases: bulk editing via scripts, provisioning dashboards from Git, or building dashboards programmatically.

---

### 4.3 Rows and Panel Organization

Rows group related panels together and can be collapsed to reduce visual clutter on dashboards with many panels — useful for organizing a large dashboard into logical sections (e.g., "Compute Metrics," "Database Metrics," "Application Errors").

---

### 4.4 Dashboard Links and Linking Between Dashboards

Dashboards can link to other dashboards (or external URLs), optionally passing along the current time range and variable selections — useful for building a drill-down navigation flow (e.g., an overview dashboard linking to a detailed per-service dashboard).

---

### 4.5 Time Range and Refresh Controls

```
Time picker: "Last 6 hours", "Last 24 hours", custom range
Refresh interval: auto-refresh every 5s/30s/1m/5m, or manual only
```

The dashboard-level time range and refresh interval apply to all panels by default, though individual panels can override the time range if genuinely needed for a specific comparison (e.g., showing "this week vs last week" on the same dashboard).

---

## MODULE 5: Visualizations & Panel Types

### 5.1 Common Panel Types

| Panel Type | Best For |
|---|---|
| Time series | The default graph type — metrics over time |
| Stat | A single large number, often with a sparkline/trend |
| Gauge | A value against a min/max range, visually like a speedometer |
| Bar chart / Bar gauge | Categorical comparisons |
| Table | Tabular, sortable/filterable data |
| Heatmap | Distribution of values over time (e.g., latency histogram buckets) |
| Logs | Displaying raw log lines (typically from Loki/Elasticsearch) |
| Node graph | Visualizing service topology/relationships |
| Pie chart | Proportional breakdown of categories |

---

### 5.2 Thresholds and Value Mappings

```
Thresholds: Green < 70%, Yellow 70-90%, Red > 90%
Value Mappings: 0 → "Down", 1 → "Up" (mapping raw numeric values to human-readable labels)
```

Thresholds provide immediate visual signal (color-coding) without requiring the viewer to read exact numbers; Value Mappings translate otherwise cryptic numeric codes (like a status enum) into meaningful labels.

---

### 5.3 Transformations

Transformations manipulate query results *after* they're returned from the data source but *before* they're rendered — e.g., joining data from two separate queries, renaming fields, filtering rows, or calculating a new field from existing ones — without needing to modify the underlying query language itself.

```
Common transformations: Merge, Filter by name, Organize fields, Add field from calculation, Group by
```

---

### 5.4 Field Overrides

Field overrides let you customize the display of specific fields/series differently from the panel's general settings — e.g., making one specific server's line red and thicker than all others in a time series panel showing dozens of servers.

---

## MODULE 6: Variables & Templating

### 6.1 What are Dashboard Variables?

Variables let a single dashboard be dynamically reused across different contexts (different hosts, environments, services) by inserting a dropdown selector that changes what data queries actually retrieve, rather than needing a separate hardcoded dashboard per host/environment.

```
Dashboard Settings → Variables → Add variable
        Name: host
        Type: Query
        Query: label_values(up, instance)
```

```promql
rate(http_requests_total{instance="$host"}[5m])
```

---

### 6.2 Variable Types

| Type | Description |
|---|---|
| Query | Populated dynamically by querying the data source (e.g., all available hostnames) |
| Custom | A manually-defined, static list of options |
| Constant | A fixed, hidden value (not shown as a dropdown, used for reuse in queries) |
| Interval | A time-interval selector (e.g., 1m, 5m, 1h) commonly used for adjusting aggregation windows |
| Data source | Lets users switch which entire data source a dashboard queries against |

---

### 6.3 Multi-Value and "All" Options

```
Enable "Multi-value" → allows selecting multiple hosts simultaneously
Enable "Include All option" → adds an "All" choice representing every possible value
```

```promql
rate(http_requests_total{instance=~"$host"}[5m])
```

Note the regex match operator (`=~`) required when a variable might resolve to multiple values (via multi-value or "All") — using the exact-match operator (`=`) would break once more than one value is selected.

---

### 6.4 Chained (Dependent) Variables

```
$datacenter → $host (host's query filters based on the selected datacenter)
```

```
Query for $host: label_values(up{datacenter="$datacenter"}, instance)
```

One variable's available options can depend on another variable's current selection — enabling cascading dropdowns (e.g., select a datacenter first, and the host dropdown narrows to only hosts in that datacenter).

---

### 6.5 Global Built-in Variables

```
$__interval        # auto-calculated step interval based on the panel's time range/width
$__timeFilter       # commonly used in SQL-based data sources for the current time range
$__from / $__to      # the dashboard's current time range boundaries
```

---

## MODULE 7: Alerting

### 7.1 Grafana Unified Alerting Architecture

```
Alert Rule (query + condition, e.g., "avg CPU > 80% for 5 minutes")
        │
Evaluated periodically by Grafana's Alerting Engine
        │
State: Normal → Pending → Firing (or → Resolved)
        │
Notification sent via a configured Contact Point (Slack, PagerDuty, Email, Webhook)
```

Modern Grafana ("Unified Alerting") consolidated what used to be separate legacy dashboard-panel-based alerting into a single, centralized alerting system independent of any specific panel.

---

### 7.2 Creating an Alert Rule

```
Alerting → Alert rules → New alert rule
        Query: avg(cpu_usage) by (instance)
        Condition: IS ABOVE 80
        Evaluation: every 1m, for 5m (must stay above threshold for 5 minutes before firing)
```

---

### 7.3 Alert States

| State | Meaning |
|---|---|
| Normal | Condition is not met — everything is fine |
| Pending | Condition IS currently met, but hasn't been true continuously long enough to fire yet |
| Alerting/Firing | Condition has been true for the full required duration — actively firing |
| NoData | The query returned no data at all (often itself worth alerting on) |
| Error | The query itself failed to execute (data source unreachable, invalid query) |

---

### 7.4 Contact Points and Notification Policies

```
Contact Point: defines WHERE a notification goes (Slack channel, PagerDuty service, email address)
Notification Policy: defines WHICH alerts route to WHICH contact points, based on label matching
```

```
Notification Policy Tree:
  Default → team-platform contact point
    └── severity=critical → pagerduty-oncall contact point (overrides default for critical alerts)
```

Notification Policies form a tree with label-based matchers, similar conceptually to Prometheus Alertmanager's routing tree (and in fact, Grafana's alerting can be configured to closely mirror Alertmanager's routing model).

---

### 7.5 Silences and Mute Timings

```
Silence: temporarily suppress notifications matching specific label criteria
         (e.g., during planned maintenance)
Mute Timing: recurring time-based suppression (e.g., "don't page overnight for low-severity alerts")
```

---

## MODULE 8: Users, Teams & Permissions (RBAC)

### 8.1 Organizational Hierarchy

```
Grafana Instance
    │
Organizations (isolated tenants within one instance)
    │
Teams (groups of users within an org)
    │
Users
```

---

### 8.2 Built-in Roles

| Role | Capabilities |
|---|---|
| Viewer | View dashboards only |
| Editor | Create/edit dashboards, but limited admin capabilities |
| Admin | Full organization administration (users, data sources, settings) |

---

### 8.3 Folder Permissions

Dashboards are organized into Folders, and permissions can be set at the folder level (cascading to all dashboards within) — a common pattern is organizing dashboards by team into separate folders, then granting each team Editor access only to their own folder.

```
Folder: "Platform Team" → Team: platform-eng → Permission: Editor
Folder: "Platform Team" → Team: everyone-else → Permission: Viewer (or no access)
```

---

### 8.4 Grafana Enterprise: Fine-Grained RBAC

Enterprise Edition extends the basic Viewer/Editor/Admin roles with fine-grained, action-level permissions (e.g., a role that can create alert rules but not modify data sources) — useful for larger organizations needing more precise separation of duties than the three basic OSS roles provide.

---

### 8.5 Authentication Integration

Grafana supports LDAP, SAML, OAuth (Google, GitHub, Azure AD, Okta), and generic OAuth2 for enterprise identity federation — commonly configured so team/role assignment in Grafana automatically maps from group membership in the external identity provider.

---

## MODULE 9: Plugins

### 9.1 Plugin Types

| Type | Purpose |
|---|---|
| Data source plugins | Add support for querying a new backend system |
| Panel plugins | Add new visualization types beyond the built-in set |
| App plugins | Full mini-applications embedded within Grafana, often bundling multiple panels/pages |

---

### 9.2 Installing Plugins

```bash
grafana-cli plugins install grafana-piechart-panel
sudo systemctl restart grafana-server
```

```yaml
# Or via environment variable in Docker/Kubernetes
GF_INSTALL_PLUGINS=grafana-piechart-panel,grafana-worldmap-panel
```

---

### 9.3 Plugin Signature Verification

Grafana enforces plugin signature verification by default for security — unsigned plugins are blocked unless explicitly allowlisted, since an unsigned plugin can't be verified as coming from a trusted source and could potentially execute arbitrary code within Grafana's context.

```ini
[plugins]
allow_loading_unsigned_plugins = my-custom-plugin-id
```

---

## MODULE 10: Provisioning (Dashboards & Data Sources as Code)

### 10.1 Why Provisioning Matters

Manually clicking through the UI to configure data sources and build dashboards doesn't scale, isn't version-controlled, and isn't reproducible across environments — Provisioning lets you define data sources, dashboards, and alert rules as YAML/JSON files loaded automatically at Grafana startup, enabling GitOps-style management.

---

### 10.2 Provisioning a Data Source

```yaml
# /etc/grafana/provisioning/datasources/prometheus.yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
```

---

### 10.3 Provisioning Dashboards

```yaml
# /etc/grafana/provisioning/dashboards/dashboards.yaml
apiVersion: 1
providers:
  - name: 'default'
    orgId: 1
    folder: 'Platform'
    type: file
    options:
      path: /var/lib/grafana/dashboards
```

Place dashboard JSON files in the specified path — Grafana automatically loads (and can auto-update, if configured) them without any manual UI interaction, keeping the source of truth in version control rather than Grafana's own database.

---

### 10.4 Provisioning Alert Rules and Contact Points

```yaml
# /etc/grafana/provisioning/alerting/rules.yaml
apiVersion: 1
groups:
  - orgId: 1
    name: my_alert_group
    folder: Platform
    rules:
      - uid: high_cpu
        title: High CPU Usage
        condition: C
        data: [...]
```

---

### 10.5 Grafana as Code with Terraform

```hcl
resource "grafana_dashboard" "my_dashboard" {
  config_json = file("${path.module}/dashboards/my-dashboard.json")
}

resource "grafana_data_source" "prometheus" {
  type = "prometheus"
  name = "prometheus"
  url  = "http://prometheus:9090"
}
```

The official Grafana Terraform provider offers an alternative (or complementary) approach to file-based provisioning, especially useful when Grafana infrastructure itself is already managed via a broader Terraform-based IaC workflow.

---

## MODULE 11: Grafana with Prometheus

### 11.1 The Grafana + Prometheus Pairing

Prometheus is the most common metrics backend paired with Grafana — Prometheus handles metric collection/storage and alerting rule evaluation at the data layer, while Grafana provides the visualization layer and (increasingly, with Unified Alerting) can also handle alert routing/notification.

```
Application → exposes /metrics → Prometheus scrapes and stores → Grafana queries and visualizes
```

---

### 11.2 PromQL Basics for Grafana Panels

```promql
rate(http_requests_total[5m])                          # per-second rate over a 5-minute window
sum(rate(http_requests_total[5m])) by (service)          # aggregated by label
histogram_quantile(0.95, rate(http_request_duration_bucket[5m]))  # p95 latency
```

---

### 11.3 Using Grafana's Explore Feature for Ad-Hoc PromQL Queries

The Explore view lets you run ad-hoc queries against any data source (not tied to a saved dashboard panel) — ideal for interactive troubleshooting/investigation before committing to a specific dashboard panel design.

---

### 11.4 Grafana Alerting vs Prometheus Alertmanager

Both can technically handle alert evaluation and routing — many organizations use Prometheus's own native alerting rules + Alertmanager for infrastructure-level alerts (tightly coupled to the metrics pipeline), while using Grafana's Unified Alerting for a more unified, visually-configured alerting experience spanning multiple data source types (not just Prometheus) — the right choice depends on existing tooling investment and team preference.

---

## MODULE 12: Grafana with Loki & Tempo (Logs & Traces)

### 12.1 Loki — Log Aggregation

Loki is Grafana Labs' log aggregation system, deliberately designed to be cost-effective by indexing only metadata (labels), not the full log line content — making it cheaper to operate at scale than full-text-indexed systems like Elasticsearch, at the cost of slower ad-hoc full-text search across very large volumes.

```logql
{app="nginx"} |= "error"
{app="nginx"} | json | status_code >= 500
```

---

### 12.2 Tempo — Distributed Tracing

Tempo is Grafana Labs' tracing backend, similarly designed for cost-effective storage at scale (object-storage-backed) — commonly paired with Grafana's trace visualization and "exemplars" linking specific metric data points directly to the traces that contributed to them.

---

### 12.3 The "LGTM" Stack

```
L - Loki (logs)
G - Grafana (visualization)
T - Tempo (traces)
M - Mimir (metrics — Grafana Labs' horizontally-scalable Prometheus-compatible metrics store)
```

Grafana Labs markets this combination as a fully open-source, cohesive observability stack, with all three data backends purpose-built to integrate tightly with Grafana's visualization and cross-linking capabilities (e.g., jumping directly from a trace to related logs).

---

### 12.4 Correlating Metrics, Logs, and Traces

Grafana supports "Explore" cross-linking — e.g., clicking on a metric spike can jump directly to correlated logs from Loki for that exact time window, or clicking a trace span can jump to the exact log lines emitted during that span — a key value proposition of using the cohesive LGTM stack together rather than disconnected tools.

---

## MODULE 13: Grafana Cloud vs Self-Hosted

### 13.1 Comparison

| Grafana OSS (Self-Hosted) | Grafana Cloud |
|---|---|
| You manage infrastructure/upgrades | Fully managed SaaS |
| Free | Free tier + paid tiers based on usage |
| Full control over data residency | Data stored in Grafana Labs' infrastructure |
| Requires operating Prometheus/Loki/Tempo separately | Includes hosted Mimir/Loki/Tempo |

---

### 13.2 When to Choose Grafana Cloud

Teams wanting to avoid the operational overhead of running and scaling Prometheus/Loki/Tempo themselves, or wanting a quick start without infrastructure investment — Grafana Cloud's free tier is also commonly used for small projects/personal use.

---

### 13.3 When to Choose Self-Hosted

Organizations with strict data residency/compliance requirements, existing infrastructure investment in self-managed Prometheus/Loki, or cost considerations at very large scale where self-hosting becomes more economical than SaaS pricing.

---

## MODULE 14: Performance & Best Practices

### 14.1 Dashboard Performance Best Practices

- Avoid excessive panels on a single dashboard (each panel is a separate query load) — split into multiple focused dashboards with drill-down links instead of one giant dashboard.
- Use appropriate time ranges and avoid unnecessarily short auto-refresh intervals on dashboards viewed by many people simultaneously (each refresh re-triggers every panel's query).
- Leverage variables and templating rather than duplicating near-identical dashboards per host/service.
- Use recording rules (in Prometheus) for expensive, frequently-used queries rather than computing them fresh on every dashboard load.

---

### 14.2 Query Optimization

```promql
# Less efficient: raw metric without pre-aggregation, computed fresh every dashboard load
histogram_quantile(0.95, rate(http_request_duration_bucket[5m]))

# More efficient: pre-computed via a Prometheus recording rule
job:http_request_duration:p95
```

For expensive queries used across many dashboards/panels, define a Prometheus recording rule to pre-compute the result periodically, and have Grafana panels query the cheap, pre-aggregated recording rule metric instead.

---

### 14.3 High Availability Grafana Setup

```
Load Balancer
      │
┌─────┼─────┐
Grafana-1  Grafana-2   (stateless application instances)
      │
Shared External Database (MySQL/PostgreSQL)
```

Running multiple Grafana instances behind a load balancer requires an external shared database (not SQLite) so all instances see consistent configuration/dashboard state — Grafana application instances themselves are otherwise stateless and horizontally scalable.

---

### 14.4 Security Best Practices

- Disable anonymous access unless deliberately intended for a public dashboard use case.
- Use folder-level permissions to restrict sensitive dashboards to appropriate teams.
- Integrate with your organization's SSO/identity provider rather than managing many local Grafana accounts.
- Restrict data source query permissions (Enterprise) for genuinely sensitive backend systems.
- Keep Grafana itself patched/updated — like any web application, it has had CVEs over time.

---

### 14.5 Backup and Disaster Recovery

```bash
# Backup the Grafana database (contains dashboards, users, alert rules, orgs)
pg_dump grafana > grafana_backup.sql

# Dashboards themselves (if provisioned as code) are already backed up via Git
```

If using provisioning-as-code for dashboards/data sources, your actual "backup" for that content is simply your Git repository — the database backup is still needed for anything created purely through the UI (ad-hoc dashboards, user accounts, non-provisioned alert rules).

---

## Summary Cheat-Sheet: Most-Used Grafana Concepts & Commands

```bash
# Install / manage
grafana-cli plugins install <plugin-id>
systemctl restart grafana-server

# Docker
docker run -d -p 3000:3000 -v grafana-storage:/var/lib/grafana grafana/grafana-oss

# Key config env vars (containerized)
GF_SERVER_ROOT_URL=
GF_SECURITY_ADMIN_PASSWORD=
GF_INSTALL_PLUGINS=
GF_DATABASE_TYPE=postgres

# Provisioning paths
/etc/grafana/provisioning/datasources/
/etc/grafana/provisioning/dashboards/
/etc/grafana/provisioning/alerting/

# PromQL in a Grafana panel
rate(metric_name[5m])
sum(rate(metric_name[5m])) by (label)
histogram_quantile(0.95, rate(metric_bucket[5m]))

# LogQL in a Grafana panel
{app="myapp"} |= "error"
```

---

*This is a standalone Grafana companion guide. Pairs with: Grafana-Troubleshooting-Guide.md and Grafana-Interview-Questions-Scenario-Based.md.*
