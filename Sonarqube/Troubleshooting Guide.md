# SonarQube Troubleshooting Guide
---

## Table of Contents

1. [General Troubleshooting Methodology](#module-1-general-troubleshooting-methodology)
2. [Installation & Startup Issues](#module-2-installation--startup-issues)
3. [Scanner & Analysis Failures](#module-3-scanner--analysis-failures)
4. [Quality Gate Issues](#module-4-quality-gate-issues)
5. [Coverage Report Issues](#module-5-coverage-report-issues)
6. [CI/CD Integration Issues](#module-6-cicd-integration-issues)
7. [Database Issues](#module-7-database-issues)
8. [Performance & Compute Engine Issues](#module-8-performance--compute-engine-issues)
9. [Authentication & Permission Issues](#module-9-authentication--permission-issues)
10. [Branch & Pull Request Decoration Issues](#module-10-branch--pull-request-decoration-issues)
11. [Plugin & Language Analyzer Issues](#module-11-plugin--language-analyzer-issues)
12. [Upgrade Issues](#module-12-upgrade-issues)

---

## MODULE 1: General Troubleshooting Methodology

### 1.1 The Layered Troubleshooting Approach

```
Symptom reported (analysis fails, results missing, gate wrong, etc.)
        │
1. Check scanner-side logs first (was the report even generated/uploaded?)
        │
2. Check SonarQube server logs (web.log, ce.log, es.log)
        │
3. Check the specific Background Task status in the UI/API
        │
4. Check configuration (sonar-project.properties, Quality Gate/Profile assignment)
        │
5. Reproduce with increased verbosity (-X on the scanner)
        │
6. Fix, re-run, confirm in the UI
```

---

### 1.2 Key Log File Locations

```bash
$SONARQUBE_HOME/logs/web.log       # Web Server process logs — UI/API issues
$SONARQUBE_HOME/logs/ce.log        # Compute Engine logs — analysis processing issues
$SONARQUBE_HOME/logs/es.log        # Elasticsearch logs — search/indexing issues
$SONARQUBE_HOME/logs/sonar.log     # Overall wrapper/startup log
```

---

### 1.3 Essential Diagnostic Commands

```bash
sonar-scanner -X                                              # verbose scanner output
curl -u <token>: "$SONAR_URL/api/system/status"                # server status
curl -u <token>: "$SONAR_URL/api/ce/activity?component=<key>"  # background task history
curl -u <token>: "$SONAR_URL/api/system/health"                # cluster/component health
```

---

## MODULE 2: Installation & Startup Issues

### 2.1 SonarQube Fails to Start — "max virtual memory areas vm.max_map_count is too low"

```bash
sudo sysctl -w vm.max_map_count=524288
echo "vm.max_map_count=524288" | sudo tee -a /etc/sysctl.conf
```
Elasticsearch (the embedded Search Server) requires this kernel setting — a very common first-install failure on freshly provisioned Linux hosts where the default value is too low.

---

### 2.2 SonarQube Refuses to Start When Run as `root`

```
ERROR: SonarQube is not permitted to run as the root user
```
This is intentional, hard-coded security behavior — create and use a dedicated non-root system user (e.g., `sonarqube`) to own and run the installation.

---

### 2.3 "Insufficient file descriptors" Error

```bash
ulimit -n 65536
# For a persistent fix, edit /etc/security/limits.conf:
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
```

---

### 2.4 Server Starts but the Web UI Never Becomes Reachable

```bash
tail -f $SONARQUBE_HOME/logs/web.log
tail -f $SONARQUBE_HOME/logs/es.log
netstat -tulnp | grep 9000
```
Check `es.log` first — if Elasticsearch fails to start (common causes: `vm.max_map_count`, insufficient heap, disk space), the Web Server process will never become fully available even though the overall wrapper process appears "running."

---

### 2.5 "Database does not exist" or Connection Refused on Startup

```bash
psql -h <db-host> -U sonar -d sonarqube -c "SELECT 1;"
```
Confirm the database and user actually exist and the credentials in `sonar.properties` match, the PostgreSQL host allows connections from the SonarQube server's IP (check `pg_hba.conf`), and that the database was created with UTF-8 encoding as required.

---

### 2.6 Docker Container Exits Immediately After Starting

```bash
docker logs sonarqube
```
Common cause on Linux Docker hosts: the host's `vm.max_map_count` setting (Module 2.1) still applies even when SonarQube runs inside a container, since it's a kernel-level setting on the host, not something configurable purely within the container.

```bash
# Set on the Docker HOST, not inside the container
sudo sysctl -w vm.max_map_count=524288
```

---

## MODULE 3: Scanner & Analysis Failures

### 3.1 "ERROR: Not authorized. Please check the properties sonar.login and sonar.password"

```bash
sonar-scanner -Dsonar.login=$SONAR_TOKEN -X
```
Confirm the token hasn't expired or been revoked, that it has "Execute Analysis" permission on the target project specifically, and (for newer SonarQube versions) whether the authentication parameter name has changed (`sonar.token` in newer versions vs the older `sonar.login`).

---

### 3.2 "Project not found" or "You're not authorized to analyze this project"

```bash
curl -u $SONAR_TOKEN: "$SONAR_URL/api/projects/search?projects=my-app"
```
Confirm the `sonar.projectKey` in your scanner config exactly matches an existing project (or that the token has permission to auto-provision new projects if this is meant to be a first-time analysis), and check for typos/case-sensitivity in the project key.

---

### 3.3 Scanner Hangs or Times Out Uploading the Report

```bash
sonar-scanner -X -Dsonar.ws.timeout=120
```
Check network connectivity/firewall rules between the CI runner and the SonarQube server (especially in CI environments with restrictive egress rules), and check the SonarQube server's own load — a heavily backlogged Compute Engine queue can cause the upload/initial acknowledgment to be slow even though the actual report content transfer itself is small.

---

### 3.4 Analysis Completes but Shows Zero Issues for a Language That Should Have Issues

```bash
sonar-scanner -X   # check which files/languages were actually detected and analyzed
```
Common causes: `sonar.sources` pointing to the wrong directory, files being unintentionally excluded via `sonar.exclusions`, or the relevant Quality Profile for that language having very few/no active rules (check Administration → Quality Profiles → confirm the assigned profile actually has rules enabled).

---

### 3.5 "SonarScanner isn't compatible with the SonarQube instance" Version Mismatch

```bash
curl "$SONAR_URL/api/server/version"
```
Check the scanner version's documented compatibility against the target server version — SonarQube periodically drops support for very old scanner versions, and using a scanner far newer than the server can also cause protocol incompatibilities; align both to compatible versions.

---

### 3.6 Analysis Fails with "OutOfMemoryError" During Scanning of a Very Large Codebase

```bash
export SONAR_SCANNER_OPTS="-Xmx4g"
sonar-scanner
```
For very large monorepos, increase the scanner's own JVM heap, and consider whether the codebase should be split into multiple smaller SonarQube projects (one per logical module/service) rather than analyzing an entire massive monorepo as a single project.

---

## MODULE 4: Quality Gate Issues

### 4.1 CI Pipeline Doesn't Actually Wait for the Quality Gate Result

The scanner step only uploads the analysis report and exits — it does **not** wait for the Compute Engine to finish processing and evaluate the Quality Gate. Without an explicit wait step (e.g., Jenkins' `waitForQualityGate`, or polling `api/ce/task` until status is `SUCCESS`, then checking `api/qualitygates/project_status`), your pipeline will always proceed regardless of the actual gate result.

```groovy
// Jenkins
timeout(time: 5, unit: 'MINUTES') {
    waitForQualityGate abortPipeline: true
}
```

---

### 4.2 Quality Gate Shows "Passed" in the UI, but You Expected It to Fail (or Vice Versa)

```bash
curl -u $SONAR_TOKEN: "$SONAR_URL/api/qualitygates/project_status?projectKey=my-app"
```
Check exactly which Quality Gate is actually assigned to this specific project (Project Settings → Quality Gate) — it may not be using the org-wide default you assumed, and check whether conditions are evaluated against "New Code" only (the default) versus overall code, since a condition failing on the whole codebase but passing on new code (or vice versa) is expected "Clean as You Code" behavior, not a bug.

---

### 4.3 "New Code" Period Isn't Behaving as Expected

```
Administration → General Settings → New Code → (check the configured definition:
  reference branch, number of days, or specific version)
```
A common source of confusion: if the "New Code" definition is set to a fixed number of days but the team rarely merges, "new code" might unexpectedly include a large amount of code merged within that window, making the Quality Gate stricter than intended — reference-branch-based new code definition is generally more predictable for teams using feature branches/PRs.

---

### 4.4 Quality Gate Fails Due to Coverage, but Coverage Data Wasn't Actually Imported

```bash
sonar-scanner -X | grep -i coverage
```
Check the scanner logs specifically for whether it found and parsed the configured coverage report path — if the report path is misconfigured or the file doesn't exist at analysis time, coverage will show as 0%, likely failing the coverage-based Quality Gate condition even though tests genuinely ran and passed elsewhere in the pipeline.

---

### 4.5 A Security Hotspot Is Blocking the Quality Gate Even Though It Was "Reviewed"

```bash
curl -u $SONAR_TOKEN: "$SONAR_URL/api/hotspots/search?projectKey=my-app&status=TO_REVIEW"
```
Confirm the hotspot was actually marked with a resolution ("Safe" or "Fixed"), not just viewed/opened in the UI — simply looking at a hotspot doesn't change its review status; it must be explicitly resolved through the review workflow for the "New Security Hotspots Reviewed" condition to reflect it.

---

## MODULE 5: Coverage Report Issues

### 5.1 Coverage Shows 0% Despite Tests Passing Locally

```bash
ls -la target/site/jacoco/jacoco.xml   # (Java/JaCoCo example — verify the file actually exists)
```
Checklist: confirm the coverage tool actually ran and generated its report **before** the SonarQube scanner step in your pipeline (ordering matters — a common CI mistake runs the scanner before or in parallel with test/coverage generation), and confirm the exact configured path (`sonar.coverage.jacoco.xmlReportPaths` or language equivalent) matches where the report was actually written.

---

### 5.2 Coverage Numbers Look Correct Locally but Differ Significantly in CI

Check for environment differences causing certain tests to be skipped in CI (feature flags, environment-specific test exclusions), and confirm the CI pipeline isn't accidentally running a subset of the test suite (e.g., only unit tests, excluding integration tests that were included in your local run's coverage).

---

### 5.3 Multi-Module Project Shows Incomplete/Missing Coverage for Some Modules

For multi-module Maven/Gradle projects, ensure coverage aggregation is properly configured (e.g., JaCoCo's report-aggregate goal for Maven multi-module builds) so a single combined coverage report reflects all modules, rather than each module's coverage being generated in isolation and only one being picked up by the scanner.

---

## MODULE 6: CI/CD Integration Issues

### 6.1 Jenkins Pipeline: "SonarQube scanner will fail because the JAVA_HOME environment is not properly set"

```groovy
tools {
    jdk 'JDK17'
}
```
Confirm a compatible JDK is configured/available on the Jenkins agent running the scanner step — recent SonarQube scanner versions require a specific minimum Java version, independent of whatever JDK your actual application build uses.

---

### 6.2 GitHub Actions: PR Decoration Not Appearing on Pull Requests

```
Administration → DevOps Platform Integrations → GitHub → (confirm configuration + token permissions)
```
Checklist: confirm the GitHub integration is properly configured server-side with a valid token/App installation having appropriate repo permissions, `sonar.pullrequest.*` properties are correctly set in the scanner invocation (key, branch, base), and that the GitHub Actions workflow actually triggers on `pull_request` events (not just `push`).

---

### 6.3 Webhook Configured but CI Pipeline Never Receives the Quality Gate Callback

```bash
curl -u $SONAR_TOKEN: "$SONAR_URL/api/webhooks/list"
tail -f $SONARQUBE_HOME/logs/web.log | grep -i webhook
```
Check the webhook URL is actually reachable from the SonarQube server (network path, not just reachable from your laptop), check for TLS certificate issues if using HTTPS, and check `web.log` for webhook delivery failure/retry logs indicating the specific error encountered.

---

### 6.4 Pipeline Fails with "Task ... failed with status FAILED" Referring to a Background Task

```bash
curl -u $SONAR_TOKEN: "$SONAR_URL/api/ce/task?id=<task-id>"
```
This means the Compute Engine itself failed to process the analysis report (not a Quality Gate failure) — check `ce.log` for the specific underlying exception, which commonly points to a corrupted/incompatible report format (often from a scanner/server version mismatch) or a database-related error during report processing.

---

### 6.5 Multiple Concurrent CI Pipelines Analyzing the Same Project Cause Conflicting/Overwritten Results

```
Administration → Configuration → General → Compute Engine (check worker count / concurrent analysis handling)
```
SonarQube processes analysis reports for a given project sequentially in the Compute Engine queue (to maintain consistent history) — if multiple pipelines for the same project submit near-simultaneously, ensure your branch/PR analysis parameters are correctly differentiating them, otherwise reports may appear to "overwrite" each other's results if they're mistakenly targeting the same branch identifier.

---

## MODULE 7: Database Issues

### 7.1 "Unsupported database" Errors During Startup or Upgrade

```bash
curl -u $SONAR_TOKEN: "$SONAR_URL/api/system/info" | grep -i database
```
Check the compatibility matrix for your specific SonarQube version — recent versions support only PostgreSQL in production; if migrating from an older, differently-supported database, a proper data migration process (not just a config change) is required.

---

### 7.2 Database Connection Pool Exhaustion Under Load

```properties
sonar.jdbc.maxActive=60
sonar.jdbc.minIdle=10
```
Symptoms include intermittent "connection timeout" errors under moderate concurrent load — increase the connection pool size (bounded by what PostgreSQL's own `max_connections` setting allows) if the Compute Engine and Web Server are both competing for a too-small shared pool.

---

### 7.3 Slow Queries / UI Sluggishness Traced to the Database

```sql
-- PostgreSQL: identify long-running queries
SELECT pid, now() - query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY duration DESC;
```
Check database server resource utilization (CPU, IO, memory) independently of SonarQube's own logs, and confirm the database host meets SonarQube's documented sizing recommendations for your project/analysis volume — an undersized database is a common root cause of general "SonarQube feels slow" complaints.

---

### 7.4 Database Migration Fails During an Upgrade

```bash
tail -f $SONARQUBE_HOME/logs/web.log | grep -i migration
```
Always back up the database before any upgrade (migrations are often irreversible without a restore), check for sufficient disk space (large migrations can require significant temporary space), and never interrupt a running migration — if a migration appears stuck, check `web.log` for actual progress/errors before assuming it's hung versus just processing a large dataset.

---

## MODULE 8: Performance & Compute Engine Issues

### 8.1 Analysis Reports Stuck in "Pending" for a Long Time

```bash
curl -u $SONAR_TOKEN: "$SONAR_URL/api/ce/activity?status=PENDING"
```
Check the Compute Engine worker count (`sonar.ce.workerCount`) — if many projects/branches are submitting analyses concurrently but only 1-2 workers are configured, a backlog queue naturally forms; increase worker count (bounded by CPU/memory available to the CE process).

---

### 8.2 Compute Engine Process Crashes or Runs Out of Memory

```bash
grep -i "OutOfMemory" $SONARQUBE_HOME/logs/ce.log
```
```properties
sonar.ce.javaOpts=-Xmx4G -Xms1G
```
Increase the Compute Engine's allocated heap — this is especially likely to occur with very large projects/monorepos or a high volume of concurrent branch analyses; also consider whether a single genuinely oversized project should be split into smaller SonarQube projects.

---

### 8.3 UI Search/Issue Filtering Is Very Slow

```bash
tail -f $SONARQUBE_HOME/logs/es.log
curl -u $SONAR_TOKEN: "$SONAR_URL/api/system/health"
```
Check Elasticsearch's allocated heap (`sonar.search.javaOpts`) and confirm `vm.max_map_count` is still correctly set (it can occasionally be reset by OS updates/reboots on hosts where the sysctl setting wasn't made persistent) — Elasticsearch under-provisioning specifically manifests as slow search/filter operations even when analysis processing itself is fine.

---

### 8.4 Overall Server Feels Slow Under Increasing Organizational Adoption

As adoption grows (more projects, more frequent CI-triggered analyses, more concurrent users browsing the UI), periodically re-evaluate sizing across all three components (Web, Compute Engine, Elasticsearch) rather than assuming the original installation sizing still fits — this is a common growth-related issue that isn't a "bug" but simply outgrown capacity planning.

---

## MODULE 9: Authentication & Permission Issues

### 9.1 "Insufficient privileges" Error During Analysis Submission

```bash
curl -u $SONAR_TOKEN: "$SONAR_URL/api/permissions/search?projectKey=my-app"
```
Confirm the token's associated user/service account actually has "Execute Analysis" permission on the specific project (project-level, not just a global role) — a very common gap when a new project is created but permissions weren't explicitly configured for the CI service account.

---

### 9.2 LDAP/SAML Users Can't Log In After Configuration

```bash
tail -f $SONARQUBE_HOME/logs/web.log | grep -i -E "ldap|saml"
```
Check the identity provider connection details (LDAP bind credentials, SAML certificate/metadata) for correctness, and confirm network connectivity from the SonarQube server to the LDAP/IdP server specifically (not just from an admin's browser, since the server itself needs to reach the identity provider for backend validation).

---

### 9.3 A Token Stops Working Unexpectedly

```bash
curl -u $SONAR_TOKEN: "$SONAR_URL/api/user_tokens/search"
```
Tokens can expire (if an expiration was set at creation), be manually revoked by an admin, or become invalid if the associated user account was deactivated/deleted — check the token's status via the API/UI rather than assuming a transient network issue.

---

### 9.4 Project-Scoped Token Can't Analyze a Newly Renamed/Recreated Project

Project analysis tokens are tied to a specific project's internal identity — if a project was deleted and recreated (even with the same project key), previously issued tokens for the old project instance are no longer valid and a new token must be generated for the recreated project.

---

## MODULE 10: Branch & Pull Request Decoration Issues

### 10.1 Branch Analysis Not Available ("Community Edition doesn't support this feature")

Branch analysis (beyond the default main branch) requires Developer Edition or above — Community Edition only supports analyzing a single branch per project; confirm your license edition if branch-specific analysis is expected to work.

---

### 10.2 PR Decoration Comment Never Appears, No Error in CI

```
Administration → DevOps Platform Integrations → (check configuration)
tail -f $SONARQUBE_HOME/logs/web.log | grep -i "pull request"
```
Check that the DevOps Platform Integration is configured correctly at the SonarQube server level (this is separate from the CI pipeline's own scanner configuration), and confirm the token/App used for the integration has permission to post comments/statuses on that specific repository.

---

### 10.3 PR Decoration Shows Outdated/Stale Results

```bash
curl -u $SONAR_TOKEN: "$SONAR_URL/api/ce/activity?component=my-app&pullRequest=42"
```
Check whether the latest push to the PR branch actually triggered a new analysis (a CI trigger misconfiguration might only run analysis on the initial PR creation, not subsequent pushes), and confirm the Compute Engine has finished processing the most recent analysis before assuming the decoration itself is stale.

---

### 10.4 Branch Shows Issues That Seem to Belong to the Base Branch, Not Just New Changes

```bash
sonar-scanner -Dsonar.pullrequest.base=main -X
```
Confirm `sonar.pullrequest.base` (or `sonar.newCode.referenceBranch` for branch analysis) is correctly pointing to the actual intended comparison branch — an incorrect or missing base branch configuration can cause SonarQube to treat far more code as "new" than actually changed in the PR/branch.

---

## MODULE 11: Plugin & Language Analyzer Issues

### 11.1 A Language Isn't Being Analyzed At All

```bash
curl -u $SONAR_TOKEN: "$SONAR_URL/api/languages/list"
```
Confirm the language plugin/analyzer is actually installed and enabled (Administration → Marketplace), and check whether the specific language requires a specific edition (some languages are Enterprise-Edition-only) rather than being universally available.

---

### 11.2 Community Plugin Breaks After a SonarQube Upgrade

```bash
tail -f $SONARQUBE_HOME/logs/web.log | grep -i plugin
```
Third-party/community plugins aren't guaranteed compatible with new SonarQube server versions — always check the plugin's documented compatibility before upgrading, and test upgrades in a staging environment first specifically because of this class of risk.

---

### 11.3 Custom Rule Plugin Fails to Load

```bash
grep -i "plugin" $SONARQUBE_HOME/logs/sonar.log
```
Check for a version mismatch between the custom rule plugin's SDK version and the running SonarQube server version, and confirm the plugin JAR is correctly placed in the `extensions/plugins` directory with correct file permissions for the SonarQube process to read it.

---

## MODULE 12: Upgrade Issues

### 12.1 Upgrade Fails Partway Through Database Migration

```bash
tail -f $SONARQUBE_HOME/logs/web.log
```
**Always** take a full database backup before any upgrade — if a migration fails partway, the safest recovery path is typically restoring the pre-upgrade database backup and retrying after diagnosing the root cause, rather than attempting to manually patch a partially-migrated schema.

---

### 12.2 Post-Upgrade, Some Projects Show Unexpectedly Different Quality Gate Results

Occasionally a major version upgrade changes default rule sets, deprecates/modifies specific rules, or adjusts how certain metrics are computed — review the release notes for "Quality Profile" or "rule changes" sections specifically, since a shift in results after an upgrade is sometimes an intentional product change rather than a bug.

---

### 12.3 Scanner Versions Across Different CI Pipelines Become Incompatible After a Server Upgrade

```bash
sonar-scanner --version
curl "$SONAR_URL/api/server/version"
```
After upgrading the server, audit all CI pipelines using various scanner versions (Maven plugin, Gradle plugin, CLI scanner, language-specific scanners) for compatibility — a server upgrade can silently break older pipelines still using outdated, now-incompatible scanner versions until they're updated in lockstep.

---

### 12.4 Elasticsearch Index Needs Rebuilding After a Major Upgrade

Some major version upgrades require an Elasticsearch reindex (SonarQube typically handles this automatically on first startup post-upgrade, but it can take significant time for large instances) — monitor `es.log` and the server status API during the post-upgrade startup window rather than assuming the server is broken if it takes longer than usual to become fully available.

---

## Quick Reference: Most Useful SonarQube Troubleshooting Commands

```bash
# Server health/status
curl -u <token>: "$SONAR_URL/api/system/status"
curl -u <token>: "$SONAR_URL/api/system/health"

# Background task / analysis processing
curl -u <token>: "$SONAR_URL/api/ce/activity?component=<projectKey>"
curl -u <token>: "$SONAR_URL/api/ce/task?id=<task-id>"

# Quality Gate status
curl -u <token>: "$SONAR_URL/api/qualitygates/project_status?projectKey=<key>"

# Permissions
curl -u <token>: "$SONAR_URL/api/permissions/search?projectKey=<key>"

# Logs
tail -f $SONARQUBE_HOME/logs/web.log
tail -f $SONARQUBE_HOME/logs/ce.log
tail -f $SONARQUBE_HOME/logs/es.log

# Verbose scanner run
sonar-scanner -X

# Kernel setting (Elasticsearch prerequisite)
sudo sysctl -w vm.max_map_count=524288
```

---
