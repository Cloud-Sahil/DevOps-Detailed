# 🔧 Trivy — Troubleshooting Guide

> A complete reference for diagnosing and fixing every common Trivy error, warning, and unexpected behavior.

---

## 📋 Table of Contents

| # | Section |
|---|---------|
| 1 | [Installation Issues](#1-installation-issues) |
| 2 | [Database Issues](#2-database-issues) |
| 3 | [Image Scanning Issues](#3-image-scanning-issues) |
| 4 | [Filesystem & Repo Scan Issues](#4-filesystem--repo-scan-issues) |
| 5 | [Kubernetes Scan Issues](#5-kubernetes-scan-issues) |
| 6 | [Config / IaC Scan Issues](#6-config--iac-scan-issues) |
| 7 | [CI/CD Pipeline Issues](#7-cicd-pipeline-issues) |
| 8 | [Output & Reporting Issues](#8-output--reporting-issues) |
| 9 | [Performance Issues](#9-performance-issues) |
| 10 | [False Positives & Ignore Issues](#10-false-positives--ignore-issues) |
| 11 | [Network & Proxy Issues](#11-network--proxy-issues) |
| 12 | [Air-Gapped / Offline Issues](#12-air-gapped--offline-issues) |
| 13 | [Diagnostic Commands](#13-diagnostic-commands) |

---

## 1. Installation Issues

---

### ❌ Problem: `trivy: command not found`

**Cause:** Trivy binary not in your `$PATH`.

**Fix:**
```bash
# Check where Trivy was installed
which trivy
ls /usr/local/bin/trivy

# If not found, reinstall
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
  | sh -s -- -b /usr/local/bin

# Verify PATH includes /usr/local/bin
echo $PATH

# Add to PATH if missing
export PATH=$PATH:/usr/local/bin
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc
```

---

### ❌ Problem: `Permission denied` when installing

**Cause:** Not enough permissions to write to `/usr/local/bin`.

**Fix:**
```bash
# Use sudo for the install
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
  | sudo sh -s -- -b /usr/local/bin

# Or install to user's local bin (no sudo needed)
mkdir -p ~/.local/bin
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
  | sh -s -- -b ~/.local/bin
export PATH=$PATH:~/.local/bin
```

---

### ❌ Problem: `no such file or directory` after install (macOS)

**Cause:** Homebrew installed Trivy but shell hasn't refreshed.

**Fix:**
```bash
brew install trivy
hash -r                  # refresh shell command cache
trivy --version
```

---

### ❌ Problem: Old version of Trivy running despite update

**Cause:** Multiple Trivy binaries in different PATH locations.

**Fix:**
```bash
# Find all trivy binaries
which -a trivy
# or
find / -name trivy -type f 2>/dev/null

# Remove the old one
sudo rm /usr/bin/trivy    # or wherever the old one is

# Keep only the new one at /usr/local/bin/trivy
trivy --version
```

---

## 2. Database Issues

---

### ❌ Problem: `ERROR: unable to initialize an image scanner`

**Cause:** Trivy database not downloaded yet or corrupted.

**Fix:**
```bash
# Force re-download of the database
trivy image --download-db-only nginx:latest

# Clear the cache and retry
trivy clean --all
trivy image nginx:latest
```

---

### ❌ Problem: `FATAL: DB error: eof`

**Cause:** The Trivy database file is corrupted (interrupted download).

**Fix:**
```bash
# Clear the entire cache
trivy clean --all

# Or manually delete the cache directory
rm -rf ~/.cache/trivy/

# Re-run the scan (DB will re-download automatically)
trivy image nginx:latest
```

---

### ❌ Problem: `Error: DB file may be broken`

**Cause:** Incomplete or corrupted database download.

**Fix:**
```bash
# Delete the broken DB
rm -rf ~/.cache/trivy/db/

# Re-download fresh
trivy image --download-db-only nginx:latest

# Verify DB is healthy
trivy image nginx:latest
```

---

### ❌ Problem: `clock skew detected` warning

**Cause:** Your system clock is out of sync with the DB server.

**Fix:**
```bash
# Sync system time (Ubuntu/Debian)
sudo timedatectl set-ntp true
sudo systemctl restart systemd-timesyncd

# Check time is correct
date

# Or just skip DB update if in a hurry
trivy image --skip-db-update nginx:latest
```

---

### ❌ Problem: Database is very old — scans might miss new CVEs

**Cause:** Trivy is running in offline mode or DB update was skipped.

**Fix:**
```bash
# Check when DB was last updated
trivy image --download-db-only nginx:latest 2>&1 | grep -i "updated"

# Force update
trivy image nginx:latest    # updates automatically each run
```

---

## 3. Image Scanning Issues

---

### ❌ Problem: `Error: GET https://... 401 Unauthorized`

**Cause:** Trying to scan a private image without authentication.

**Fix:**
```bash
# Log in to Docker Hub first
docker login

# Log in to private registry
docker login myregistry.example.com

# Then scan
trivy image myregistry.example.com/myapp:latest

# Or set credentials via environment variables
TRIVY_USERNAME=myuser TRIVY_PASSWORD=mypass \
  trivy image myregistry.example.com/myapp:latest
```

---

### ❌ Problem: `Image not found: myapp:latest`

**Cause:** Image doesn't exist locally or in the registry.

**Fix:**
```bash
# Check if image exists locally
docker images | grep myapp

# Pull it first if not local
docker pull myapp:latest

# Then scan the local image
trivy image myapp:latest

# Or scan the image directly from the tar file
docker save myapp:latest > myapp.tar
trivy image --input myapp.tar
```

---

### ❌ Problem: `0 vulnerabilities found` but image is definitely old

**Cause 1:** `--ignore-unfixed` is set — only shows fixed CVEs.
**Cause 2:** DB is outdated.
**Cause 3:** Wrong scan target.

**Fix:**
```bash
# Remove --ignore-unfixed to see ALL CVEs including unfixed
trivy image nginx:1.16.0

# Force DB update
trivy clean --all && trivy image nginx:1.16.0

# Make sure you're scanning the right image
docker images | grep nginx
trivy image nginx:1.16.0 --severity LOW,MEDIUM,HIGH,CRITICAL
```

---

### ❌ Problem: Scan shows different results from yesterday

**Cause:** New CVEs were published and added to the Trivy DB.

**This is expected behavior.**

```bash
# This is normal! Vulnerability databases update daily.
# A clean scan yesterday may have new findings today.

# To reproduce old results, pin the Trivy DB version
# or use --skip-db-update to freeze the DB
trivy image --skip-db-update nginx:latest
```

---

### ❌ Problem: `--exit-code 1` is blocking ALL builds — even with no real issues

**Cause:** Low-severity CVEs are triggering the exit code.

**Fix:**
```bash
# Only fail on CRITICAL
trivy image --exit-code 1 --severity CRITICAL myapp:latest

# Only fail on fixable HIGH or CRITICAL
trivy image --exit-code 1 --severity HIGH,CRITICAL --ignore-unfixed myapp:latest
```

---

### ❌ Problem: Scanning is slow for large images

**Cause:** Large image size + DB loading time.

**Fix:**
```bash
# Use a smaller base image in your Dockerfile
FROM node:20-alpine    # much smaller than node:20

# Run scan with more parallelism
trivy image --parallel 5 nginx:latest

# Skip the DB update if already fresh
trivy image --skip-db-update myapp:latest
```

---

## 4. Filesystem & Repo Scan Issues

---

### ❌ Problem: `trivy fs` finds 0 vulnerabilities in a Node.js project

**Cause:** `node_modules/` is missing (dependencies not installed).

**Fix:**
```bash
# Install dependencies first
npm install

# Then scan
trivy fs .

# Or explicitly target the lock file
trivy fs --scanners vuln ./package-lock.json
```

---

### ❌ Problem: Secret scanner finds false positives (test/example credentials)

**Cause:** Test files, documentation, or example configs contain dummy secrets.

**Fix:**
```bash
# Add a .trivyignore to suppress specific findings
cat >> .trivyignore << 'EOF'
# Test credentials in example files — not real
generic-api-key
EOF

# Or skip specific directories
trivy fs --skip-dirs tests,examples,docs --scanners secret .
```

---

### ❌ Problem: `trivy repo` can't clone — `rate limit exceeded`

**Cause:** GitHub API rate limit hit when scanning remote repos.

**Fix:**
```bash
# Set GitHub token for higher rate limit
export GITHUB_TOKEN=ghp_yourtoken
trivy repo https://github.com/user/repo

# Or clone first and scan locally
git clone https://github.com/user/repo
trivy repo ./repo
```

---

### ❌ Problem: Scan is too slow on large monorepo

**Cause:** Scanning thousands of files takes time.

**Fix:**
```bash
# Skip directories you don't care about
trivy fs --skip-dirs node_modules,vendor,.git,dist,build .

# Only scan specific file types (vulnerabilities only)
trivy fs --scanners vuln .

# Scan a specific sub-directory
trivy fs ./src
```

---

## 5. Kubernetes Scan Issues

---

### ❌ Problem: `Error: kubectl is not configured` when running `trivy k8s`

**Cause:** kubectl not installed or kubeconfig not set up.

**Fix:**
```bash
# Check kubectl is working
kubectl cluster-info
kubectl get nodes

# If not configured, set KUBECONFIG
export KUBECONFIG=~/.kube/config

# Or specify kubeconfig explicitly
KUBECONFIG=/path/to/kubeconfig trivy k8s cluster

# Verify Trivy can reach the cluster
kubectl get pods --all-namespaces
trivy k8s --report summary cluster
```

---

### ❌ Problem: `trivy k8s` scan hangs and never finishes

**Cause:** Large cluster with many images — timeout hit.

**Fix:**
```bash
# Increase timeout
trivy k8s --timeout 30m --report summary cluster

# Scan one namespace at a time
trivy k8s --namespace production --report summary cluster
trivy k8s --namespace staging --report summary cluster

# Scan only workloads (faster than full infra scan)
trivy k8s --components workload cluster
```

---

### ❌ Problem: `ERROR: failed to pull image` for some pods

**Cause:** Trivy can't pull private images without credentials.

**Fix:**
```bash
# If using Docker Hub private images
docker login

# If using a private registry
docker login myregistry.example.com

# Set registry credentials for Trivy
TRIVY_USERNAME=user TRIVY_PASSWORD=pass \
  trivy k8s --report summary cluster
```

---

### ❌ Problem: `trivy k8s` shows permissions errors

**Cause:** Your Kubernetes user doesn't have read access to all resources.

**Fix:**
```bash
# Check your K8s permissions
kubectl auth can-i list pods --all-namespaces
kubectl auth can-i get secrets --all-namespaces

# Scan only namespaces you have access to
trivy k8s --namespace my-namespace cluster

# Or use a service account with cluster-reader role
kubectl create clusterrolebinding trivy-reader \
  --clusterrole=view --serviceaccount=default:trivy-sa
```

---

## 6. Config / IaC Scan Issues

---

### ❌ Problem: `trivy config` shows 0 failures on clearly insecure Dockerfile

**Cause:** Trivy doesn't recognize the Dockerfile format or path.

**Fix:**
```bash
# Specify the file directly
trivy config ./Dockerfile

# Make sure the file is actually named "Dockerfile"
ls -la | grep -i docker

# Show ALL results including PASSED checks
trivy config --include-non-failures ./Dockerfile
```

---

### ❌ Problem: Too many `LOW` misconfig findings cluttering output

**Cause:** Default shows all severities.

**Fix:**
```bash
# Filter to only HIGH and CRITICAL misconfigs
trivy config --severity HIGH,CRITICAL ./k8s/

# Show only failures (no passed checks)
trivy config --severity MEDIUM,HIGH,CRITICAL ./k8s/
```

---

### ❌ Problem: Custom Rego policy not being applied

**Cause:** Wrong policy directory or namespace not matching.

**Fix:**
```bash
# Use the correct flags for custom policies
trivy config \
  --policy ./my-policies \
  --namespaces custom \
  ./Dockerfile

# Verify Rego syntax is valid
opa check ./my-policies/

# Test with verbose output
trivy config --debug --policy ./my-policies --namespaces custom ./Dockerfile
```

---

## 7. CI/CD Pipeline Issues

---

### ❌ Problem: Jenkins pipeline fails even though image is clean

**Cause:** `--exit-code 1` triggers on LOW/MEDIUM CVEs too.

**Fix:**
```groovy
// Jenkinsfile — only fail on HIGH and CRITICAL
sh '''
    trivy image \
      --exit-code 1 \
      --severity HIGH,CRITICAL \
      --ignore-unfixed \
      myapp:${BUILD_NUMBER}
'''
```

---

### ❌ Problem: Trivy takes too long in CI — builds time out

**Cause:** DB download happening on every build.

**Fix:**
```yaml
# Cache the Trivy DB between builds (GitHub Actions example)
- name: Cache Trivy DB
  uses: actions/cache@v3
  with:
    path: ~/.cache/trivy
    key: trivy-db-${{ github.run_id }}
    restore-keys: trivy-db-

- name: Run Trivy scan
  run: trivy image --skip-db-update myapp:latest
```

```groovy
// Jenkins — use a shared workspace cache
sh 'trivy image --cache-dir /var/cache/trivy --skip-db-update myapp:latest'
```

---

### ❌ Problem: `trivy image` fails in Docker-in-Docker CI environment

**Cause:** Docker socket not accessible inside the CI container.

**Fix:**
```bash
# Mount the Docker socket into the CI container
docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image myapp

# Or build the image, export to tar, then scan
docker build -t myapp . && docker save myapp > myapp.tar
trivy image --input myapp.tar
```

---

### ❌ Problem: SARIF file upload fails in GitHub Actions

**Cause:** File not generated or path incorrect.

**Fix:**
```yaml
- name: Run Trivy
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: myapp:latest
    format: sarif
    output: trivy-results.sarif   # make sure this filename matches below

- name: Upload SARIF
  uses: github/codeql-action/upload-sarif@v3
  if: always()                    # upload even if scan failed
  with:
    sarif_file: trivy-results.sarif   # must match the output file above
```

---

## 8. Output & Reporting Issues

---

### ❌ Problem: JSON output is empty or malformed

**Cause:** Scan found 0 results OR output flag is wrong.

**Fix:**
```bash
# Check the correct syntax
trivy image -f json -o results.json nginx:latest
#                ↑            ↑
#           format flag    output file

# Verify file was created
ls -la results.json
cat results.json | python3 -m json.tool   # pretty print to verify

# If empty, run without filter to see all results
trivy image --severity LOW,MEDIUM,HIGH,CRITICAL -f json -o results.json nginx:latest
```

---

### ❌ Problem: HTML report shows no data

**Cause:** Template file path wrong or template not found.

**Fix:**
```bash
# Download the HTML template from Trivy's GitHub repo
curl -L https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl \
  -o html.tpl

# Run with the downloaded template
trivy image \
  --format template \
  --template "@./html.tpl" \
  --output report.html \
  nginx:latest

# Open the report
open report.html     # macOS
xdg-open report.html # Linux
```

---

### ❌ Problem: Table output is too wide / wraps on terminal

**Fix:**
```bash
# Use JSON or a more compact format
trivy image -f json nginx:latest | python3 -m json.tool | less

# Pipe to less for scrollable output
trivy image nginx:latest | less -S

# Filter to only what you need
trivy image nginx:latest | grep -E "CRITICAL|HIGH"
```

---

## 9. Performance Issues

---

### ❌ Problem: Scan takes 10+ minutes

**Root causes and fixes:**

```bash
# Cause 1 — DB re-downloading every time
# Fix — use a cached DB
trivy image --skip-db-update myapp:latest

# Cause 2 — Large image (1 GB+)
# Fix — use a smaller base image in Dockerfile
FROM node:20-alpine    # alpine is tiny

# Cause 3 — Scanning node_modules in filesystem scan
# Fix — skip known large directories
trivy fs --skip-dirs node_modules,vendor,dist .

# Cause 4 — Single-threaded scan
# Fix — increase parallel threads
trivy image --parallel 5 myapp:latest
```

---

### ❌ Problem: High memory usage during scan

**Cause:** Large image or many layers being analyzed.

**Fix:**
```bash
# Limit Docker memory for the Trivy container
docker run --memory=512m aquasec/trivy image nginx:latest

# Reduce the number of parallel jobs
trivy image --parallel 1 myapp:latest

# Use a lighter base image — fewer layers = less memory needed
```

---

## 10. False Positives & Ignore Issues

---

### ❌ Problem: CVE keeps showing even after adding to .trivyignore

**Cause 1:** Typo in the CVE ID.
**Cause 2:** Wrong ignore file location.
**Cause 3:** Not using the `--ignorefile` flag with a non-default path.

**Fix:**
```bash
# Check the CVE ID is exactly correct (case-sensitive)
cat .trivyignore
# CVE-2021-44228    ← must be exact

# Run with explicit ignore file path
trivy image --ignorefile ./.trivyignore nginx:latest

# Debug — check if Trivy reads the file
trivy image --debug --ignorefile .trivyignore nginx:latest 2>&1 | grep -i ignore
```

---

### ❌ Problem: `.trivyignore` file not being detected automatically

**Cause:** You're not running Trivy from the directory containing `.trivyignore`.

**Fix:**
```bash
# Trivy looks for .trivyignore in the current working directory
ls .trivyignore     # must be in current directory

# If not, specify the path explicitly
trivy image --ignorefile /path/to/.trivyignore nginx:latest

# Or navigate to the directory that has the file
cd /path/to/project && trivy image nginx:latest
```

---

### ❌ Problem: CVE is reported but the fix is "won't be fixed" (upstream issue)

**Cause:** The distro maintainer has decided not to patch this CVE.

**Options:**
```bash
# Option 1 — Ignore unfixed CVEs entirely
trivy image --ignore-unfixed nginx:latest

# Option 2 — Add to .trivyignore with a reason
# .trivyignore
CVE-2021-XXXXX exp:2025-12-31   # Won't be fixed upstream — accepted risk, reviewed on 2024-06-01

# Option 3 — Use a different base image
# Switch from debian-based to alpine — fewer unfixed CVEs typically
FROM nginx:alpine    # instead of nginx:latest (which is debian-based)
```

---

## 11. Network & Proxy Issues

---

### ❌ Problem: `Error: failed to download the DB: connection refused` / timeout

**Cause:** Network connectivity issue or corporate proxy blocking requests.

**Fix:**
```bash
# Test connectivity to Trivy's DB server
curl -v https://github.com/aquasecurity/trivy-db/releases/

# Set proxy settings
export HTTPS_PROXY=http://proxy.company.com:8080
export HTTP_PROXY=http://proxy.company.com:8080
export NO_PROXY=localhost,127.0.0.1,.company.com

trivy image nginx:latest

# Or for Docker-based Trivy
docker run \
  -e HTTPS_PROXY=http://proxy.company.com:8080 \
  aquasec/trivy image nginx:latest
```

---

### ❌ Problem: SSL certificate verification fails

**Cause:** Corporate proxy uses a custom SSL certificate.

**Fix:**
```bash
# Skip TLS verification (use with caution in trusted networks)
trivy image --insecure nginx:latest

# Or add your corporate CA certificate to the system
sudo cp company-ca.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

---

## 12. Air-Gapped / Offline Issues

---

### ❌ Problem: No internet access — can't download Trivy DB

**Solution — Pre-download the DB and transfer:**

```bash
# On an internet-connected machine:
# 1. Download Trivy
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
  | sh -s -- -b ./bin

# 2. Download the DB
./bin/trivy image --download-db-only nginx:latest
ls ~/.cache/trivy/db/

# 3. Package the DB
tar czf trivy-db-$(date +%Y%m%d).tar.gz ~/.cache/trivy/db/

# 4. Transfer to air-gapped machine and extract
scp trivy-db-20260101.tar.gz user@air-gapped-host:/home/user/
ssh user@air-gapped-host
mkdir -p ~/.cache/trivy/
tar xzf trivy-db-20260101.tar.gz -C ~/

# 5. Scan offline
trivy image --skip-db-update nginx:latest
```

---

### ❌ Problem: Need to scan Docker images offline

```bash
# On internet machine — save the image to a tar
docker pull nginx:latest
docker save nginx:latest > nginx.tar

# Transfer to air-gapped machine
scp nginx.tar user@air-gapped-host:/tmp/

# On air-gapped machine — scan from tar file
trivy image --skip-db-update --input /tmp/nginx.tar
```

---

## 13. Diagnostic Commands

Use these to understand what Trivy is doing when things go wrong.

```bash
# ── DEBUG OUTPUT ───────────────────────────────────────────────
trivy image --debug nginx:latest       # verbose debug logs

# ── CHECK TRIVY VERSION ────────────────────────────────────────
trivy --version
# trivy version 0.50.0

# ── CHECK DB STATUS ────────────────────────────────────────────
trivy image --download-db-only nginx:latest 2>&1

# ── CLEAR ALL CACHE ────────────────────────────────────────────
trivy clean --all
# or manually:
rm -rf ~/.cache/trivy/

# ── CHECK WHAT TRIVY IS IGNORING ──────────────────────────────
trivy image --debug --ignorefile .trivyignore nginx:latest 2>&1 | grep -i ignore

# ── TEST NETWORK CONNECTIVITY ─────────────────────────────────
curl -v https://github.com/aquasecurity/trivy-db/releases/latest

# ── CHECK DOCKER SOCKET ACCESS ────────────────────────────────
ls -la /var/run/docker.sock
docker info

# ── CHECK KUBERNETES ACCESS ───────────────────────────────────
kubectl cluster-info
kubectl get nodes
kubectl auth can-i list pods --all-namespaces

# ── VALIDATE CONFIG SCAN IS WORKING ───────────────────────────
trivy config --include-non-failures ./Dockerfile

# ── VIEW TRIVY HELP ───────────────────────────────────────────
trivy --help
trivy image --help
trivy k8s --help
trivy config --help
```

---

## Quick Troubleshooting Decision Tree

```
Trivy not working?
│
├── "command not found"
│   └── Fix: reinstall + check $PATH
│
├── "DB error" or "eof"
│   └── Fix: trivy clean --all → re-run
│
├── "0 vulnerabilities" on old image
│   └── Fix: remove --ignore-unfixed, update DB
│
├── "Unauthorized" on private image
│   └── Fix: docker login first
│
├── CI pipeline fails on "clean" image
│   └── Fix: add --severity HIGH,CRITICAL --ignore-unfixed
│
├── Scan hangs / timeout
│   └── Fix: --timeout 30m, scan one namespace at a time
│
├── Network / proxy error
│   └── Fix: set HTTPS_PROXY env var
│
└── Air-gapped environment
    └── Fix: pre-download DB, transfer, --skip-db-update
```

---

*Trivy v0.50+ | Troubleshooting Guide | DevSecOps Reference*
