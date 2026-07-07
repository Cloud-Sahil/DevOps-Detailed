# Grafana Troubleshooting Guide
---

## Table of Contents

1. [General Troubleshooting Methodology](#module-1-general-troubleshooting-methodology)
2. [Installation & Startup Issues](#module-2-installation--startup-issues)
3. [Data Source Connection Issues](#module-3-data-source-connection-issues)
4. [Dashboard & Panel Rendering Issues](#module-4-dashboard--panel-rendering-issues)
5. [Query Issues](#module-5-query-issues)
6. [Variable & Templating Issues](#module-6-variable--templating-issues)
7. [Alerting Issues](#module-7-alerting-issues)
8. [Authentication & Permission Issues](#module-8-authentication--permission-issues)
9. [Plugin Issues](#module-9-plugin-issues)
10. [Performance Issues](#module-10-performance-issues)
11. [Provisioning Issues](#module-11-provisioning-issues)
12. [Database & Storage Issues](#module-12-database--storage-issues)

---

## MODULE 1: General Troubleshooting Methodology

### 1.1 The Layered Troubleshooting Approach

```
Symptom reported (panel broken, alert not firing, login fails, etc.)
        │
1. Check Grafana server logs first
        │
2. Reproduce in Explore mode (isolate: is it the query, the panel, or the data source?)
        │
3. Test the underlying data source independently (query it directly, bypassing Grafana)
        │
4. Check browser console (for frontend/rendering issues specifically)
        │
5. Check configuration (grafana.ini / environment variables / provisioning files)
        │
6. Fix, verify, document
```

---

### 1.2 Key Log Locations

```bash
# Package install (Linux)
/var/log/grafana/grafana.log

# Docker
docker logs grafana

# Increase log verbosity for deeper debugging
# grafana.ini:
[log]
level = debug
```

---

### 1.3 Essential Diagnostic Commands

```bash
curl http://localhost:3000/api/health              # basic server health check
curl -u admin:password http://localhost:3000/api/datasources   # list configured data sources
systemctl status grafana-server                       # service status (systemd installs)
grafana-cli admin reset-admin-password newpassword       # reset admin password if locked out
```

---

## MODULE 2: Installation & Startup Issues

### 2.1 Grafana Service Fails to Start

```bash
sudo journalctl -u grafana-server -n 100 --no-pager
tail -f /var/log/grafana/grafana.log
```
Common causes: port 3000 already in use by another process, a malformed `grafana.ini` (syntax error), or insufficient permissions on the data directory (`/var/lib/grafana`).

```bash
sudo lsof -i :3000    # check what's using the port
```

---

### 2.2 "database is locked" Errors (SQLite)

```
Error: database is locked
```
This is a classic symptom of using the default SQLite database under concurrent load or with multiple Grafana instances pointed at the same file — migrate to an external MySQL/PostgreSQL database for any production or multi-instance deployment, since SQLite fundamentally isn't designed for this concurrency pattern.

---

### 2.3 Docker Container Exits Immediately

```bash
docker logs grafana
```
Common causes: a mounted volume with incorrect ownership/permissions (Grafana's container runs as a specific non-root UID by default and needs write access to `/var/lib/grafana`), or an invalid environment variable value causing a config parsing failure at startup.

```bash
docker run -d -p 3000:3000 -v grafana-storage:/var/lib/grafana --user "$(id -u)" grafana/grafana-oss
```

---

### 2.4 Web UI Loads but Shows a Blank/White Page

```bash
# Check browser developer console for JavaScript errors
```
Common causes: `root_url` misconfigured in `grafana.ini` not matching how users actually access Grafana (especially common behind a reverse proxy with a subpath), or a browser cache issue after an upgrade — try a hard refresh/incognito window to rule out stale cached assets.

```ini
[server]
root_url = %(protocol)s://%(domain)s:%(http_port)s/grafana/
serve_from_sub_path = true
```

---

### 2.5 Locked Out of Admin Account

```bash
grafana-cli admin reset-admin-password newpassword
```
Resets the built-in admin account's password directly via the CLI without needing to already be logged in — requires shell access to the Grafana host/container.

---

## MODULE 3: Data Source Connection Issues

### 3.1 "Data source connection failed" When Testing a New Data Source

```bash
curl http://prometheus:9090/api/v1/status/config    # test connectivity directly, bypassing Grafana
```
Checklist: confirm the URL is reachable from the Grafana server itself (not just your own browser — Grafana's backend, not your browser, makes the actual query in "Server" access mode), check for a typo in the URL/port, and confirm any required authentication (basic auth, API key, TLS client cert) is correctly configured.

---

### 3.2 Data Source Works in "Save & Test" but Panels Still Show No Data

```bash
# Test the exact query directly against the data source outside Grafana
curl -G http://prometheus:9090/api/v1/query --data-urlencode 'query=up'
```
A successful connectivity test only confirms Grafana can reach the data source at all — it doesn't validate that a *specific query* returns data; check the panel's actual query for typos, incorrect label matchers, or a time range that simply doesn't contain any matching data.

---

### 3.3 "Access" Mode Confusion — Browser vs Server

```
Server (default): Grafana's backend queries the data source directly — data source must be reachable FROM the Grafana server
Browser (legacy, deprecated for most sources): the USER'S browser queries the data source directly
```
If a data source works when tested by an admin but fails for regular users (or vice versa), confirm which access mode is configured — "Browser" mode requires the data source to be reachable directly from every end user's browser (including through any CORS restrictions), a fundamentally different network path than "Server" mode.

---

### 3.4 TLS/Certificate Errors Connecting to a Data Source

```
Error: x509: certificate signed by unknown authority
```
Common in internal/self-signed-certificate environments — configure the data source's TLS settings to include the appropriate CA certificate, or (only for non-production/testing) temporarily enable "Skip TLS Verify" while investigating the proper certificate chain configuration.

---

### 3.5 CloudWatch/Azure Monitor/GCP Data Source Authentication Failures

```bash
# For CloudWatch specifically, verify the IAM role/credentials Grafana is using have the required permissions
aws sts get-caller-identity --profile grafana-role
```
Cloud-provider data sources require appropriately scoped IAM/service credentials (e.g., `cloudwatch:GetMetricData`, `cloudwatch:ListMetrics` for AWS) — a "test" failure often points to either missing credentials entirely or credentials lacking the specific required read permissions for the metrics APIs being queried.

---

## MODULE 4: Dashboard & Panel Rendering Issues

### 4.1 Panel Shows "No Data"

```
Check: 1) Is the time range appropriate? 2) Does the query actually match any series?
       3) Is the data source correct for this specific panel?
```
Use the panel's "Query Inspector" (available in the panel edit view) to see the exact raw request sent and raw response received — this immediately clarifies whether the issue is an empty result set from the data source versus a Grafana-side rendering problem.

---

### 2.2 Panel Shows an Error Icon / Red Exclamation Mark

```
Click the panel → Inspect → Query Inspector → check the exact error message
```
Common causes: invalid query syntax for that data source (a PromQL syntax error, a malformed SQL query), a data source that's become unreachable since the panel was last successfully loaded, or exceeding a data source's query complexity/result-size limits.

---

### 4.3 Graph Renders but Values Look Wrong / Unexpected

```
Check the panel's unit configuration, and check for unintended aggregation (sum vs avg) in the query itself
```
A very common "wrong values" cause is a unit mismatch (e.g., displaying bytes as if they were already in megabytes) or accidentally summing a metric across all label combinations when an average or a filtered subset was intended — verify both the query's actual aggregation logic and the panel's display unit configuration separately.

---

### 4.4 Dashboard Loads Extremely Slowly

```
Query Inspector → check individual panel query response times
```
Identify whether one specific expensive panel is the bottleneck (common with unoptimized queries or panels containing very high-cardinality data) versus a genuinely widespread issue (e.g., the underlying data source itself being slow/overloaded) — see Module 10 for deeper performance troubleshooting.

---

### 4.5 Panel Displays Correctly for Admin but Broken for a Regular Viewer User

```
Check: does the viewer's role have query permissions on the underlying data source? (Enterprise data source permissions)
Check: is the panel using a template variable the viewer doesn't have appropriate values for?
```
Confirm the viewing user's role/data-source permissions aren't more restrictive in a way that silently changes query results (Enterprise fine-grained permissions), and check whether the dashboard relies on a variable whose default/available values differ unexpectedly for that specific user's context.

---

## MODULE 5: Query Issues

### 5.1 PromQL Query Returns Data in Explore but Not in a Dashboard Panel

```
Compare: the panel's configured time range, and whether template variables are correctly interpolated
```
Check the panel's specific query for a variable reference (`$host`) that might not be resolving as expected in the dashboard context — use the panel's "Query Inspector" to see the *actual final query* sent to Prometheus after variable interpolation, comparing it against what you tested directly in Explore.

---

### 5.2 Query Times Out on Large Time Ranges

```
Prometheus/data-source-specific query timeout settings, and Grafana's own dataproxy timeout
```
```ini
[dataproxy]
timeout = 60
```
Increase the relevant timeout settings (both Grafana's proxy timeout and the underlying data source's own query timeout configuration) for legitimately long-running queries over large time ranges, but also consider whether the query itself should be optimized (e.g., via a Prometheus recording rule) rather than simply extending timeouts indefinitely.

---

### 5.3 LogQL Query in Loki Panel Returns No Results Despite Logs Existing

```logql
{app="myapp"}    # confirm the label selector itself matches — check available labels first
```
Use Explore to first run a broad label-only query (`{app="myapp"}`) to confirm logs are being ingested and matched at all, before adding more specific filter expressions (`|= "error"`) — narrowing down whether the issue is the label selector itself or a subsequent line-filter expression not matching as expected.

---

### 5.4 SQL Data Source Query Fails with a Macro-Related Error

```sql
SELECT * FROM metrics WHERE $__timeFilter(time_column)
```
Grafana's SQL data sources (MySQL, PostgreSQL) support special macros like `$__timeFilter()` that expand into the appropriate time-range WHERE clause — a common error source is referencing a column name in the macro that doesn't actually exist in the table, or forgetting the macro entirely and instead hardcoding a time range that doesn't respond to the dashboard's time picker.

---

### 5.5 Query Works for One Time Range but Not Another

```
Check: step/interval calculation ($__interval) producing too many or too few data points
```
For very wide time ranges, `$__interval` (auto-calculated based on panel width/time range) can become very large, potentially causing unexpected downsampling/aggregation behavior in certain data sources — verify the actual interpolated interval value via Query Inspector rather than assuming it's a fixed value across different zoom levels.

---

## MODULE 6: Variable & Templating Issues

### 6.1 Variable Dropdown Shows No Options

```
Check the variable's query directly in Explore, and confirm the data source is correctly selected for that variable
```
A variable's underlying query (e.g., `label_values(up, instance)`) can fail silently in the dropdown UI — test the exact same query manually against the data source to confirm it actually returns values.

---

### 6.2 Multi-Value Variable Breaks the Query (Returns No Data When Multiple Values Selected)

```promql
# WRONG when $host can be multi-valued
rate(http_requests_total{instance="$host"}[5m])

# CORRECT — use regex match for multi-value variables
rate(http_requests_total{instance=~"$host"}[5m])
```
Using the exact-match operator (`=`) with a variable that can resolve to multiple comma-separated values breaks the query syntax entirely — always use the regex match operator (`=~`) for any variable configured as multi-value or with "Include All" enabled.

---

### 6.3 Chained Variable Doesn't Update Based on Parent Variable's Selection

```
Check: does the child variable's query actually reference the parent variable? ($datacenter used inside $host's query?)
```
Confirm the dependent variable's own query definition explicitly references the parent variable (e.g., `label_values(up{datacenter="$datacenter"}, instance)`) — if it doesn't reference the parent at all, it will never actually filter/update based on the parent's selection, appearing "chained" in name only.

---

### 6.4 "All" Option Doesn't Actually Return All Values (Returns Empty or Errors)

```
Check the variable's "Custom all value" setting
```
By default, "All" expands to a regex matching every currently-available option value — but if a "Custom all value" has been manually configured (e.g., `.*`) that doesn't correctly match the actual label values' format, the resulting query can behave unexpectedly; verify the actual interpolated "All" value via Query Inspector.

---

## MODULE 7: Alerting Issues

### 7.1 Alert Rule Never Fires Despite the Condition Clearly Being True

```
Alerting → Alert rules → check the rule's current STATE (Normal/Pending/Firing) directly
```
Check the "for" duration setting — if the condition must be true continuously for 5 minutes before firing but only briefly spiked and recovered, it would correctly remain in "Pending" and never reach "Firing"; also confirm the evaluation interval itself is running (check for evaluation errors in the rule's state history).

---

### 7.2 Alert Fires But No Notification Is Received

```
Check: Notification Policy label matching, and the Contact Point's own delivery status/logs
```
Verify the alert's labels actually match the notification policy's routing matchers (a common gap when an alert rule's labels don't align with what the notification policy tree expects), and check the specific Contact Point's delivery history/logs (Alerting → Contact points → check for delivery failures, e.g., an expired Slack webhook URL).

---

### 7.3 "NoData" State Triggering Unwanted Alerts

```
Alert rule configuration → "Configure no data and error handling" → set explicit behavior
```
By default, a query returning no data can be configured to trigger "NoData" as its own alertable state (useful for catching "my exporter died" scenarios) or to be treated as OK/no-op — review this setting specifically if you're getting unexpected alert noise whenever a metric briefly has no data (e.g., during a deploy causing a brief scrape gap).

---

### 7.4 Alert Rule Query Works in Explore but Errors When Used as an Alert Condition

```
Check: alert conditions require a query returning a SINGLE value per series (not a full time-series range in the final expression step)
```
Alert rule "Condition" expressions specifically expect reduced/instant values (not raw ranges) — ensure the query pipeline includes an appropriate reduction step (e.g., using Grafana's built-in Math/Reduce expressions, or ensuring the underlying query itself, like a PromQL instant query, already returns single values) rather than a raw time-series range expression that works fine for visualization but isn't valid as an alert condition.

---

### 7.5 Duplicate/Flapping Alert Notifications

```
Check: evaluation interval vs "for" duration relationship, and whether the underlying metric itself is noisy/flapping
```
A metric hovering right at the threshold can cause repeated Pending→Firing→Normal transitions, each potentially generating a notification — consider a longer "for" duration, or adding hysteresis (different thresholds for triggering vs. resolving) if the underlying tooling/pattern supports it, to reduce genuine flapping rather than just noise from an overly sensitive threshold.

---

## MODULE 8: Authentication & Permission Issues

### 8.1 SAML/OAuth Login Redirects Fail or Loop

```bash
tail -f /var/log/grafana/grafana.log | grep -i -E "saml|oauth"
```
Check the configured redirect/callback URL in both Grafana's config AND the identity provider's application configuration for an exact match (including protocol http vs https, and any subpath), and confirm `root_url` in `grafana.ini` correctly reflects the externally-accessible URL if Grafana is behind a reverse proxy.

---

### 8.2 User Logs In Successfully but Has No Dashboards Visible

```
Check: which Organization the user was assigned to on login, and folder-level permissions for that org/team
```
A common cause with multi-org setups or auto-provisioning via SSO group mapping is the user being placed into an unexpected (often a newly auto-created, empty) Organization rather than the intended one — verify org assignment logic in the auth configuration.

---

### 8.3 "Access Denied" for an Action a User Should Be Permitted to Perform

```bash
curl -u admin:password http://localhost:3000/api/access-control/user/permissions
```
(Enterprise) Check the user's actual effective fine-grained RBAC permissions via the API, since a custom role might be missing a specific granular permission even if it appears to grant broadly similar capabilities in the UI's role summary.

---

### 8.4 LDAP Group Mapping Not Assigning the Expected Grafana Role

```ini
[[servers.group_mappings]]
group_dn = "cn=admins,ou=groups,dc=company,dc=com"
org_role = "Admin"
```
Verify the `group_dn` exactly matches the LDAP group's actual distinguished name (a very common source of silent mismatch), and enable LDAP debug logging to see exactly which groups Grafana believes the authenticating user belongs to during the actual login attempt.

---

## MODULE 9: Plugin Issues

### 9.1 Plugin Installation Fails

```bash
grafana-cli plugins install <plugin-id> -v
```
Check network connectivity to Grafana's plugin catalog (`grafana.com`), and confirm the plugin ID is spelled correctly and actually compatible with your installed Grafana version.

---

### 9.2 Plugin Installed but Not Appearing / "Plugin Not Signed" Error

```ini
[plugins]
allow_loading_unsigned_plugins = my-plugin-id
```
Grafana blocks unsigned plugins by default as a security measure — for internally-developed or unsigned community plugins you've deliberately chosen to trust, explicitly allowlist the plugin ID in `grafana.ini`, understanding the security tradeoff of doing so.

---

### 9.3 Plugin Panel Renders Incorrectly After a Grafana Upgrade

```bash
grafana-cli plugins list-remote | grep <plugin-name>
```
Check the plugin's own compatibility/changelog against the new Grafana version — plugins (especially community-maintained ones) aren't always immediately updated for the newest Grafana releases; test upgrades in a staging environment specifically because of this class of risk before rolling out to production.

---

## MODULE 10: Performance Issues

### 10.1 Grafana UI Feels Sluggish Overall

```bash
# Check Grafana server resource utilization
top -p $(pgrep grafana)
```
Check for a high volume of concurrent dashboard auto-refreshes across many simultaneously open browser tabs/users (each triggering fresh queries), and consider whether the underlying data sources themselves (not Grafana) are actually the bottleneck — Grafana is often just relaying an underlying data source's own slowness.

---

### 10.2 Specific Dashboard Loads Very Slowly

```
Query Inspector on each panel → identify which specific panel/query is the slow outlier
```
Isolate whether one specific expensive panel (e.g., an unbounded high-cardinality query, or an inefficient join in a SQL panel) is responsible, versus the dashboard simply having too many panels overall each contributing a small load that adds up — the fix differs (optimize one query vs. reduce total panel count/split dashboard).

---

### 10.3 High Cardinality Queries Causing Data Source Overload

```promql
# Potentially very expensive if "instance" has thousands of unique values
sum(rate(http_requests_total[5m])) by (instance)
```
Reduce cardinality where genuinely not needed for the visualization's purpose (aggregate by a coarser label, or pre-aggregate via a recording rule), since Grafana repeatedly re-running a high-cardinality query on every dashboard load/refresh directly stresses the underlying data source (particularly relevant for Prometheus, where cardinality is a well-known operational concern).

---

### 10.4 Alerting Evaluation Falling Behind (Alerts Delayed)

```
Check the Alerting engine's own evaluation metrics/logs for evaluation duration vs configured interval
```
If there are many alert rules with short evaluation intervals (e.g., hundreds of rules evaluating every 10 seconds), the alerting engine itself can become a bottleneck — review whether all rules genuinely need such frequent evaluation, and check Grafana server resource allocation if evaluation is measurably falling behind its configured schedule.

---

## MODULE 11: Provisioning Issues

### 11.1 Provisioned Dashboard Doesn't Appear After Restart

```bash
tail -f /var/log/grafana/grafana.log | grep -i provisioning
```
Check the provisioning YAML file's syntax (a malformed YAML file fails silently from the UI's perspective, only visible in logs), confirm the specified dashboard JSON `path` actually exists and is readable by the Grafana process user, and verify the JSON file itself is valid (a broken dashboard JSON export can prevent loading).

---

### 11.2 Changes Made in the UI to a Provisioned Dashboard Are Lost After Restart

This is expected, default behavior — provisioned dashboards are typically read-only from the UI's perspective (or, depending on configuration, UI edits are overwritten by the provisioned file on next load) specifically to enforce that the file (version-controlled) remains the single source of truth; if UI-based editing needs to persist, the dashboard shouldn't be provisioned, or the provisioning workflow needs an explicit "export changes back to the file" step.

---

### 11.3 Provisioned Data Source Connection Fails Only in Provisioning, Works Fine When Added Manually via UI

```yaml
datasources:
  - name: Prometheus
    url: http://prometheus:9090   # confirm this is resolvable from WITHIN the Grafana container/host
```
A common cause in containerized setups: the provisioned URL uses a hostname (like `localhost`) that resolves differently inside a container's network namespace than what a human manually configuring it via a browser might assume — ensure the URL is correct from Grafana's own network perspective (e.g., a Docker Compose service name, not `localhost`).

---

### 11.4 Provisioning Alert Rules Fails with a Validation Error

```bash
tail -f /var/log/grafana/grafana.log | grep -i alert
```
Check the provisioned alert rule YAML against the exact schema expected by your specific Grafana version (this schema has evolved across versions as Unified Alerting matured) — a rule definition exported from a different/newer Grafana version may not be directly compatible with an older instance's provisioning schema.

---

## MODULE 12: Database & Storage Issues

### 12.1 Grafana's Internal Database Growing Very Large

```sql
-- Check for excessive alert rule state history / annotation accumulation
SELECT COUNT(*) FROM annotation;
```
Annotations and alert state history can accumulate significantly over time — review and configure appropriate retention/cleanup settings (`[unified_alerting] / [annotations]` cleanup settings in `grafana.ini`) rather than letting the database grow unbounded indefinitely.

---

### 12.2 Migrating from SQLite to an External Database

```ini
[database]
type = mysql
host = db.mycompany.com:3306
name = grafana
user = grafana
password = ${GF_DATABASE_PASSWORD}
```
Grafana doesn't provide a fully automated built-in migration tool for this specific transition in all versions — plan carefully (back up first), test in a non-production environment, and be aware this is a common enough need that community tools/documented procedures exist, but it requires deliberate planning rather than a simple config flip on a live production instance.

---

### 12.3 Database Connection Pool Exhaustion Under Load

```ini
[database]
max_open_conn = 300
max_idle_conn = 100
```
If Grafana logs show database connection errors under load (many concurrent users/dashboards), tune the connection pool settings — bounded by what the actual database server's own `max_connections` configuration allows.

---

## Quick Reference: Most Useful Grafana Troubleshooting Commands

```bash
# Health & Status
curl http://localhost:3000/api/health
systemctl status grafana-server

# Logs
tail -f /var/log/grafana/grafana.log
docker logs grafana

# Admin Recovery
grafana-cli admin reset-admin-password newpassword

# Data Sources
curl -u admin:password http://localhost:3000/api/datasources

# Plugins
grafana-cli plugins list-remote
grafana-cli plugins install <plugin-id>

# Query debugging (in-UI)
Panel → Inspect → Query Inspector

# Config check
grafana-server -config /etc/grafana/grafana.ini cfg:default.log.mode=console
```

---

*This troubleshooting guide pairs with Grafana-Complete-Study-Guide.md and Grafana-Interview-Questions-Scenario-Based.md.*
