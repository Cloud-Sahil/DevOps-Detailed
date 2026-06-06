# <img width="80" height="80" alt="image" src="https://github.com/user-attachments/assets/9a41a608-9624-4abf-a9a7-931c15580474" /> — Complete Security Scanner Guide

> **Trivy** is an open-source, all-in-one vulnerability and misconfiguration scanner by Aqua Security.
> One tool. One command. Complete DevSecOps security coverage.

---

## 📋 Table of Contents

| # | Module | Topics |
|---|--------|--------|
| 1 | [What is Trivy?](#module-1--what-is-trivy) | Overview, Purpose, Why use it |
| 2 | [Core Concepts](#module-2--core-concepts) | CVE, CVSS, SBOM, Shift Left |
| 3 | [Vulnerability Databases](#module-3--vulnerability-databases) | NVD, OSV, GitHub Advisory |
| 4 | [Installation](#module-4--installation) | All platforms, Docker, verify |
| 5 | [Scan Types](#module-5--scan-types) | Image, FS, Repo, K8s, Config |
| 6 | [Output Formats](#module-6--output-formats) | Table, JSON, SARIF, SBOM |
| 7 | [Flags & Options](#module-7--flags--options) | All important flags explained |
| 8 | [.trivyignore File](#module-8--trivyignore-file) | Ignore CVEs selectively |
| 9 | [CI/CD Integration](#module-9--cicd-integration) | Jenkins, GitHub Actions, GitLab |
| 10 | [DevSecOps & Shift Left](#module-10--devsecops--shift-left) | Security in the pipeline |
| 11 | [Scanning Policies](#module-11--scanning-policies) | OPA, Rego policy rules |
| 12 | [Real-World Workflows](#module-12--real-world-workflows) | End-to-end scan examples |
| 13 | [Quick Reference Cheatsheet](#module-13--quick-reference-cheatsheet) | All commands in one place |

---

## Module 1 — What is Trivy?

### Overview

**Trivy** (Triangle + Ivy — developed by Aqua Security) is a free, open-source security scanner that covers the **entire software supply chain** from code to cloud.

It answers the question: *"Does my application have security vulnerabilities?"*

```
Your Code  →  Dockerfile  →  Image  →  K8s Cluster  →  Production
    │              │            │            │
  trivy fs    trivy config  trivy image  trivy k8s
    ✅             ✅            ✅           ✅
```

### What Can Trivy Scan?

| Target | Command | What It Finds |
|--------|---------|---------------|
| Container Image | `trivy image` | CVEs in OS packages & app libraries |
| File System | `trivy fs` | Vulnerabilities, secrets, misconfigs |
| Git Repository | `trivy repo` | CVEs, hardcoded secrets |
| Kubernetes Cluster | `trivy k8s` | Misconfigurations, vulnerabilities |
| IaC / Config Files | `trivy config` | Dockerfile, Helm, Terraform, K8s YAML issues |
| SBOM | `trivy sbom` | Vulnerabilities from an SBOM file |

### What Does Trivy Detect?

| Category | Examples |
|----------|---------|
| **CVEs** | Vulnerable library versions (Log4Shell, Heartbleed) |
| **Misconfigurations** | Containers running as root, privileged pods |
| **Secrets** | AWS keys, GitHub tokens hardcoded in files |
| **License Risks** | GPL libraries in proprietary software |
| **SBOM** | Full inventory of all software components |

### Why Trivy Over Other Tools?

| Feature | Trivy | Snyk | Clair | Grype |
|---------|-------|------|-------|-------|
| Free & Open Source | ✅ | Partial | ✅ | ✅ |
| Single binary | ✅ | ❌ | ❌ | ✅ |
| Image + IaC + K8s | ✅ | ❌ | ❌ | ❌ |
| Secret scanning | ✅ | ✅ | ❌ | ❌ |
| CI/CD integration | ✅ | ✅ | Partial | ✅ |
| Setup time | < 1 min | 10+ min | Complex | < 1 min |

---

## Module 2 — Core Concepts

### CVE — Common Vulnerabilities and Exposures

A **CVE** is a unique, publicly disclosed identifier for a known security vulnerability.

```
CVE - YEAR - NUMBER
CVE - 2021 - 44228   ← This is Log4Shell (Log4j vulnerability)
```

**Every CVE contains:**

| Field | Description | Example |
|-------|-------------|---------|
| ID | Unique identifier | `CVE-2021-44228` |
| Severity | Impact level | `CRITICAL` |
| Description | What the flaw is | Remote code execution via JNDI lookup |
| CVSS Score | Numeric severity (0–10) | `10.0` |
| Affected Package | Which library/version | `log4j 2.0 – 2.14.1` |
| Fixed Version | Patched version | `log4j 2.15.0+` |

---

### CVSS — Common Vulnerability Scoring System

CVSS is a numeric score from **0.0 to 10.0** that tells you how serious a vulnerability is.

```
┌────────────────────────────────────────────────────────────┐
│                  CVSS SEVERITY SCALE                       │
├─────────────┬──────────────┬────────┬─────────────────────┤
│  CRITICAL   │  9.0 – 10.0  │  🔴   │ Fix IMMEDIATELY     │
│  HIGH       │  7.0 –  8.9  │  🟠   │ Fix as soon as poss │
│  MEDIUM     │  4.0 –  6.9  │  🟡   │ Plan to fix soon    │
│  LOW        │  0.1 –  3.9  │  🟢   │ Fix when convenient │
│  UNKNOWN    │  No score    │  ⚪   │ Review manually     │
└─────────────┴──────────────┴────────┴─────────────────────┘
```

**CVSS is calculated from:**
- **Attack Vector** — how the attacker reaches the system (network, physical, local)
- **Attack Complexity** — how hard is the attack to execute
- **Privileges Required** — does attacker need existing access
- **Impact** — confidentiality, integrity, availability impact

---

### SBOM — Software Bill of Materials

An **SBOM** is a complete inventory of all components in your software — like a nutrition label for code.

```
Your Application
├── React 18.2.0
│   ├── react-dom 18.2.0
│   └── scheduler 0.23.0
├── Express 4.18.2
│   ├── body-parser 1.20.1
│   └── path-to-regexp 0.1.7  ← Has CVE-2024-45296!
└── axios 1.6.2
```

Trivy can **generate** SBOMs and also **scan** existing SBOMs for vulnerabilities.

---

### Shift Left Security

**Shift Left** means moving security checks **earlier** in the software development lifecycle.

```
❌ TRADITIONAL (Shift Right — security at the end):
Dev → Code → Build → Test → Deploy → [Security Scan] → Production
                                             ↑
                                    Too late! Expensive to fix here.

✅ SHIFT LEFT with Trivy:
Dev → [trivy fs] → Code → [trivy image] → Build → [trivy k8s] → Deploy
        ↑                       ↑                      ↑
   Catch secrets &       Catch CVEs in          Catch cluster
   misconfigs in code    Docker image           misconfigurations
```

**Why does it matter?**

| Stage Found | Cost to Fix | Time to Fix |
|-------------|-------------|------------|
| Development | 💚 $80 | Minutes |
| Build/CI | 🟡 $240 | Hours |
| QA/Testing | 🟠 $960 | Days |
| Production | 🔴 $7,600+ | Days–Weeks + breach risk |

---

### IaC — Infrastructure as Code

IaC means writing infrastructure configuration as code files. Trivy scans these for security misconfigurations.

| IaC Type | Examples | What Trivy Checks |
|----------|---------|------------------|
| Dockerfile | `Dockerfile` | Root user, exposed secrets, no HEALTHCHECK |
| Kubernetes | `deployment.yml`, `service.yml` | Privileged pods, no resource limits |
| Terraform | `main.tf` | Public S3 buckets, open security groups |
| Helm | `values.yaml`, `Chart.yaml` | Insecure template defaults |
| CloudFormation | `template.yaml` | Open ports, no encryption |

---

## Module 3 — Vulnerability Databases

Trivy pulls from multiple trusted sources and combines them into its own database, updated **every 6 hours**.

| Database | Source | Covers |
|----------|--------|--------|
| **NVD** | NIST National Vulnerability Database | All public CVEs |
| **GitHub Advisory DB** | GitHub Security Advisories | npm, pip, Go, Ruby, etc. |
| **OSV** | Open Source Vulnerabilities | Multi-ecosystem |
| **RedHat** | RedHat Security Data | RHEL, CentOS, Fedora |
| **Debian** | Debian Security Tracker | Debian packages |
| **Ubuntu** | Ubuntu CVE Tracker | Ubuntu packages |
| **Alpine SecDB** | Alpine Linux | Alpine packages |
| **Trivy DB** | Aqua Security (aggregated) | All of the above, unified |

### How Trivy Updates Its Database

```bash
# Trivy auto-updates before every scan
trivy image nginx:latest
# [00:00:01] Downloading vulnerability DB...
# [00:00:03] Scan complete.

# Force manual update
trivy image --download-db-only nginx:latest

# Skip update (use cached DB — for offline/air-gapped)
trivy image --skip-db-update nginx:latest
```

---

## Module 4 — Installation

### Linux

```bash
# Method 1 — Install script (recommended)
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
  | sh -s -- -b /usr/local/bin

# Method 2 — APT (Ubuntu/Debian)
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" \
  | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y

# Method 3 — RPM (RHEL/CentOS/Amazon Linux)
sudo rpm -ivh https://github.com/aquasecurity/trivy/releases/download/v0.50.0/trivy_0.50.0_Linux-64bit.rpm
```

### macOS

```bash
# Homebrew
brew install trivy

# Or direct binary
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
  | sh -s -- -b /usr/local/bin
```

### Windows

```powershell
# Scoop
scoop install trivy

# Or download binary from GitHub releases
# https://github.com/aquasecurity/trivy/releases
```

### Docker (No Install Needed)

```bash
# Run Trivy as a Docker container
docker run --rm aquasec/trivy image nginx:latest

# Scan the host filesystem
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image nginx:latest

# Scan a local directory
docker run --rm -v $(pwd):/project aquasec/trivy fs /project
```

### Verify Installation

```bash
trivy --version
# trivy version 0.50.0

trivy --help
# Shows all available commands
```

---

## Module 5 — Scan Types

### 1. Image Scanning — `trivy image`

Scans a Docker container image for:
- CVEs in OS packages (apt, apk, yum)
- CVEs in application libraries (npm, pip, go.mod, pom.xml)

```bash
# Scan a public image
trivy image nginx:latest

# Scan a specific version
trivy image nginx:1.25.3

# Scan a locally built image
trivy image myapp:v1.0

# Scan only HIGH and CRITICAL
trivy image --severity HIGH,CRITICAL nginx:latest

# Scan and fail if CRITICAL found (for CI/CD)
trivy image --exit-code 1 --severity CRITICAL nginx:latest

# Scan image from a saved tar file
docker save nginx:latest > nginx.tar
trivy image --input nginx.tar

# Ignore vulnerabilities with no fix yet
trivy image --ignore-unfixed nginx:latest

# Output as JSON for automation
trivy image -f json -o results.json nginx:latest

# Show detailed info per vulnerability
trivy image --format table nginx:latest
```

**Example output:**
```
nginx:latest (debian 12.5)

Total: 25 (CRITICAL: 2, HIGH: 8, MEDIUM: 12, LOW: 3)

┌─────────────────┬──────────────────┬──────────┬──────────────┬───────────────┐
│    Library      │  Vulnerability   │ Severity │  Installed   │  Fixed In     │
├─────────────────┼──────────────────┼──────────┼──────────────┼───────────────┤
│ openssl         │ CVE-2024-12345   │ CRITICAL │ 3.0.11       │ 3.0.13        │
│ libexpat1       │ CVE-2023-52425   │ HIGH     │ 2.5.0        │ 2.6.0         │
└─────────────────┴──────────────────┴──────────┴──────────────┴───────────────┘
```

---

### 2. File System Scanning — `trivy fs`

Scans your local project directory for:
- CVEs in dependency files (package.json, requirements.txt, pom.xml)
- Hardcoded secrets
- IaC misconfigurations

```bash
# Scan current directory
trivy fs .

# Scan a specific path
trivy fs /path/to/project

# Scan for vulnerabilities only
trivy fs --scanners vuln .

# Scan for secrets only
trivy fs --scanners secret .

# Scan for misconfigurations only
trivy fs --scanners config .

# Scan for all types at once
trivy fs --scanners vuln,secret,config .

# Scan a Node.js project
trivy fs --scanners vuln ./my-node-app

# Skip specific directories
trivy fs --skip-dirs node_modules,vendor .
```

---

### 3. Repository Scanning — `trivy repo`

Scans a Git repository (local or remote) without needing to clone it first.

```bash
# Scan a remote GitHub repo
trivy repo https://github.com/username/repo

# Scan local git repository
trivy repo .

# Scan with all scanner types
trivy repo --scanners vuln,secret,config .

# Scan a specific branch
trivy repo --branch develop https://github.com/username/repo

# Scan a specific commit
trivy repo --commit abc1234 https://github.com/username/repo

# Scan a specific tag
trivy repo --tag v1.2.3 https://github.com/username/repo
```

---

### 4. Kubernetes Scanning — `trivy k8s`

Scans a live Kubernetes cluster or individual resources.

```bash
# Scan the entire cluster (requires kubectl configured)
trivy k8s --report summary cluster

# Detailed report of everything
trivy k8s --report all cluster

# Scan a specific namespace
trivy k8s --namespace production --report summary cluster

# Scan a specific deployment
trivy k8s deployment/myapp

# Scan a specific pod
trivy k8s pod/mypod-abc123

# Scan all resources in a namespace
trivy k8s --namespace default all

# Output as JSON
trivy k8s --format json --report all cluster

# Scan only for misconfigurations
trivy k8s --scanners config cluster
```

---

### 5. Config / IaC Scanning — `trivy config`

Scans infrastructure-as-code files for security misconfigurations.

```bash
# Scan a Dockerfile
trivy config ./Dockerfile

# Scan Kubernetes manifest folder
trivy config ./k8s/

# Scan a single K8s YAML file
trivy config ./k8s/deployment.yml

# Scan Terraform files
trivy config ./terraform/

# Scan Helm charts
trivy config ./helm-chart/

# Scan with custom policy directory
trivy config --policy ./my-policies ./k8s/

# Show all results including PASSED checks
trivy config --include-non-failures ./k8s/
```

**Example — Dockerfile misconfig output:**
```
Dockerfile (dockerfile)

Tests: 20 (SUCCESSES: 17, FAILURES: 3, EXCEPTIONS: 0)

FAILURES: 3

HIGH: Specify at least 1 USER command in Dockerfile
  → Running containers as root is a security risk
  → Line: 12

MEDIUM: Add HEALTHCHECK instruction in Dockerfile
  → No health check defined — container may run unhealthy
  → Line: N/A
```

---

### 6. SBOM Commands

```bash
# Generate an SBOM from an image (CycloneDX format)
trivy image --format cyclonedx -o sbom.json nginx:latest

# Generate SBOM in SPDX format
trivy image --format spdx-json -o sbom.spdx.json nginx:latest

# Scan an existing SBOM file for vulnerabilities
trivy sbom ./sbom.json

# Generate SBOM from filesystem
trivy fs --format cyclonedx -o sbom.json .
```

---

## Module 6 — Output Formats

```bash
# Table (default — human readable, best for terminal)
trivy image nginx:latest

# JSON (best for automation, parsing, dashboards)
trivy image -f json -o results.json nginx:latest

# SARIF (for GitHub Security tab, GitLab SAST)
trivy image -f sarif -o results.sarif nginx:latest

# CycloneDX SBOM (standard supply chain format)
trivy image -f cyclonedx -o sbom.cyclonedx.json nginx:latest

# SPDX SBOM (SPDX standard format)
trivy image -f spdx-json -o sbom.spdx.json nginx:latest

# HTML report (custom template)
trivy image --format template \
  --template "@contrib/html.tpl" \
  -o report.html nginx:latest

# GitHub Annotations (for PR comments)
trivy image -f github nginx:latest
```

| Format | Best For |
|--------|---------|
| `table` | Developers reading in terminal |
| `json` | Automation, scripts, dashboards |
| `sarif` | GitHub/GitLab Security tab integration |
| `cyclonedx` | Supply chain compliance, SBOM sharing |
| `spdx-json` | Legal/compliance SBOM requirements |
| `template` | Custom HTML reports for teams |

---

## Module 7 — Flags & Options

### Most Important Flags

```bash
# ── SEVERITY FILTER ────────────────────────────────────────────
--severity CRITICAL,HIGH          # only show these severities
--severity CRITICAL               # only CRITICAL
--severity LOW,MEDIUM,HIGH,CRITICAL  # show all (same as default)

# ── EXIT CODE (for CI/CD — fail the pipeline) ──────────────────
--exit-code 1                     # exit with code 1 if vulnerabilities found
--exit-code 0                     # always exit 0 (never fail pipeline)

# ── IGNORE OPTIONS ─────────────────────────────────────────────
--ignore-unfixed                  # skip CVEs with no fix available yet
--ignorefile .trivyignore         # use custom ignore file
--skip-dirs node_modules,vendor   # skip specific directories

# ── OUTPUT ─────────────────────────────────────────────────────
-f table                          # output format (table/json/sarif/cyclonedx)
-o results.json                   # save output to file
-q                                # quiet mode (less verbose)

# ── DATABASE ───────────────────────────────────────────────────
--skip-db-update                  # use cached DB (offline mode)
--download-db-only                # only update DB, don't scan
--cache-dir /tmp/trivy-cache      # custom cache directory

# ── SCANNER SELECTION ──────────────────────────────────────────
--scanners vuln                   # vulnerabilities only
--scanners secret                 # secrets only
--scanners config                 # misconfigurations only
--scanners vuln,secret,config     # all scanners

# ── PERFORMANCE ────────────────────────────────────────────────
--timeout 10m                     # set timeout (default 5m)
--parallel 5                      # parallel scanning threads

# ── KUBERNETES SPECIFIC ────────────────────────────────────────
--namespace default               # scan specific K8s namespace
--report summary                  # summary report
--report all                      # full detailed report
--components workload             # scan only workloads
--components infra                # scan only infrastructure
```

### Common Flag Combinations

```bash
# CI/CD — fail build on CRITICAL only
trivy image --exit-code 1 --severity CRITICAL myapp:latest

# CI/CD — fail build on HIGH or CRITICAL, skip unfixed
trivy image --exit-code 1 --severity HIGH,CRITICAL --ignore-unfixed myapp:latest

# Developer — quick scan, readable output
trivy image --severity HIGH,CRITICAL --format table nginx:latest

# Full scan — all types, all formats, save to file
trivy fs --scanners vuln,secret,config -f json -o results.json .

# Kubernetes — scan cluster with detailed output
trivy k8s --report all --format json -o k8s-report.json cluster
```

---

## Module 8 — .trivyignore File

The `.trivyignore` file tells Trivy which CVEs to skip — useful for false positives, accepted risks, or vulnerabilities you're already tracking.

### Format

```bash
# .trivyignore
# Lines starting with # are comments

# Ignore a CVE permanently (false positive)
CVE-2021-12345

# Ignore with an expiry date (re-evaluate after this date)
CVE-2022-67890 exp:2025-12-31

# Ignore with a reason comment
CVE-2023-11111  # Not applicable — we don't use this code path

# Ignore a specific misconfiguration
AVD-DS-0002

# Ignore a specific secret finding
generic-api-key
```

### Use the ignore file in a scan

```bash
# Uses .trivyignore in current directory (auto-detected)
trivy image nginx:latest

# Specify a custom ignore file location
trivy image --ignorefile /path/to/.trivyignore nginx:latest
```

### Best Practices for .trivyignore

- Always add a **comment** explaining why it's ignored
- Always add an **expiry date** — forces regular review
- Never ignore CRITICAL CVEs without documented approval
- Commit `.trivyignore` to git so the team sees what's ignored

---

## Module 9 — CI/CD Integration

### Jenkins Pipeline

```groovy
// Jenkinsfile — Security scan stage
pipeline {
    agent any

    stages {

        stage('Build Image') {
            steps {
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }

        stage('Trivy Security Scan') {
            steps {
                // Install Trivy if not already installed
                sh '''
                    which trivy || curl -sfL \
                      https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
                      | sh -s -- -b /usr/local/bin
                '''

                // Run scan — fail pipeline on CRITICAL vulnerabilities
                sh '''
                    trivy image \
                      --exit-code 1 \
                      --severity HIGH,CRITICAL \
                      --ignore-unfixed \
                      --format json \
                      --output trivy-report.json \
                      myapp:${BUILD_NUMBER}
                '''
            }
            post {
                always {
                    // Archive the report regardless of pass/fail
                    archiveArtifacts artifacts: 'trivy-report.json', allowEmptyArchive: true
                }
                failure {
                    echo "❌ Security scan failed — CRITICAL or HIGH vulnerabilities found!"
                    echo "   Review trivy-report.json for details."
                }
                success {
                    echo "✅ Security scan passed — no HIGH/CRITICAL vulnerabilities found."
                }
            }
        }

        stage('Push Image') {
            // Only push if security scan passed
            steps {
                sh 'docker push myapp:${BUILD_NUMBER}'
            }
        }

    }
}
```

---

### GitHub Actions

```yaml
# .github/workflows/trivy.yml
name: Trivy Security Scan

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  trivy-scan:
    runs-on: ubuntu-latest
    steps:

      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

      # Option A — Using the official Trivy action
      - name: Run Trivy image scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:${{ github.sha }}
          format: sarif
          output: trivy-results.sarif
          severity: CRITICAL,HIGH
          exit-code: '1'

      # Upload results to GitHub Security tab
      - name: Upload results to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        if: always()   # upload even if scan fails
        with:
          sarif_file: trivy-results.sarif

      # Option B — Scan the filesystem (repo code)
      - name: Scan filesystem for secrets and misconfigs
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: fs
          scan-ref: .
          scanners: secret,config
          format: table
```

---

### GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - build
  - security
  - deploy

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

trivy-image-scan:
  stage: security
  image: aquasec/trivy:latest
  script:
    # Scan the built image
    - trivy image
        --exit-code 1
        --severity HIGH,CRITICAL
        --ignore-unfixed
        --format json
        --output trivy-report.json
        $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  artifacts:
    when: always
    reports:
      # Shows in GitLab's Security dashboard
      container_scanning: trivy-report.json
    paths:
      - trivy-report.json
  allow_failure: false

trivy-config-scan:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy config --exit-code 1 --severity HIGH,CRITICAL ./k8s/
  allow_failure: true  # warn but don't block deployment
```

---

## Module 10 — DevSecOps & Shift Left

### DevSecOps Pipeline with Trivy

```
Code Commit  ─────────────────────────────────────────► Production
     │              │              │              │
     ▼              ▼              ▼              ▼
 trivy fs       trivy image    trivy config   trivy k8s
  Secrets        CVEs in        K8s YAML       Live cluster
  CVEs in        Docker image   Misconfigs     scan
  code deps
```

### Implementing a Gate Policy

A **security gate** stops the CI/CD pipeline if vulnerabilities are found above your threshold.

```bash
# Gate 1 — block on CRITICAL vulnerabilities
trivy image --exit-code 1 --severity CRITICAL myapp:latest

# Gate 2 — block on HIGH or CRITICAL (stricter)
trivy image --exit-code 1 --severity HIGH,CRITICAL myapp:latest

# Gate 3 — block only on fixable vulnerabilities
trivy image --exit-code 1 --severity HIGH,CRITICAL \
  --ignore-unfixed myapp:latest
```

**Recommended gate policy:**

| Environment | Severity Gate | Unfixed? |
|-------------|--------------|----------|
| Development | MEDIUM+ | Include |
| CI/CD | HIGH+ | Skip unfixed |
| Staging | CRITICAL | Skip unfixed |
| Production | Any CRITICAL | Block |

---

## Module 11 — Scanning Policies

### Custom OPA/Rego Policies

Trivy supports custom security policies written in **Rego** (Open Policy Agent language).

```rego
# my-policies/deny-root-user.rego
package custom.dockerfile.deny_root

# Deny any Dockerfile that doesn't have a USER instruction
deny[msg] {
    input.Type == "dockerfile"
    not has_user_instruction
    msg := "Dockerfile must specify a non-root USER instruction"
}

has_user_instruction {
    input.Stages[_].Commands[_].Cmd == "user"
}
```

```bash
# Apply custom policy during scan
trivy config --policy ./my-policies --namespaces custom ./Dockerfile

# Combine built-in + custom policies
trivy config \
  --policy ./my-policies \
  --namespaces custom \
  ./k8s/
```

---

## Module 12 — Real-World Workflows

### Workflow 1 — Scanning Before Every Docker Push

```bash
#!/bin/bash
# scan-before-push.sh

IMAGE_NAME="sahil0117/myntraa"
IMAGE_TAG="${1:-latest}"
FULL_IMAGE="${IMAGE_NAME}:${IMAGE_TAG}"

echo "🔍 Building image: ${FULL_IMAGE}"
docker build -t "${FULL_IMAGE}" .

echo "🛡️ Running Trivy security scan..."
trivy image \
  --exit-code 1 \
  --severity HIGH,CRITICAL \
  --ignore-unfixed \
  --format table \
  "${FULL_IMAGE}"

if [ $? -eq 0 ]; then
    echo "✅ No HIGH/CRITICAL vulnerabilities found. Pushing image..."
    docker push "${FULL_IMAGE}"
else
    echo "❌ Vulnerabilities found! Push BLOCKED."
    exit 1
fi
```

---

### Workflow 2 — Daily Cluster Audit

```bash
#!/bin/bash
# daily-cluster-audit.sh

DATE=$(date +%Y-%m-%d)
REPORT_DIR="./security-reports"
mkdir -p "${REPORT_DIR}"

echo "🛡️ Running daily Kubernetes cluster security audit — ${DATE}"

# Scan the full cluster
trivy k8s \
  --report all \
  --format json \
  --output "${REPORT_DIR}/cluster-audit-${DATE}.json" \
  cluster

echo "📊 Generating summary..."
trivy k8s --report summary cluster

echo "✅ Report saved to ${REPORT_DIR}/cluster-audit-${DATE}.json"
```

---

### Workflow 3 — Scan All Images in a Cluster

```bash
#!/bin/bash
# scan-all-cluster-images.sh

# Get all unique images from the cluster
IMAGES=$(kubectl get pods --all-namespaces \
  -o jsonpath="{.items[*].spec.containers[*].image}" \
  | tr ' ' '\n' | sort -u)

echo "Found $(echo "$IMAGES" | wc -l) unique images to scan"

for IMAGE in $IMAGES; do
    echo "🔍 Scanning: ${IMAGE}"
    trivy image \
      --severity CRITICAL,HIGH \
      --ignore-unfixed \
      --quiet \
      "${IMAGE}" || echo "⚠️  Issues found in: ${IMAGE}"
done
```

---

## Module 13 — Quick Reference Cheatsheet

```bash
# ── INSTALLATION ──────────────────────────────────────────────
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
  | sh -s -- -b /usr/local/bin
trivy --version

# ── IMAGE SCANNING ────────────────────────────────────────────
trivy image nginx:latest                          # full scan
trivy image --severity HIGH,CRITICAL nginx        # filter severity
trivy image --exit-code 1 --severity CRITICAL img # fail on CRITICAL
trivy image --ignore-unfixed nginx                # skip unfixed
trivy image -f json -o results.json nginx         # JSON output
trivy image --input myimage.tar                   # scan tar file

# ── FILESYSTEM SCANNING ───────────────────────────────────────
trivy fs .                                        # scan current dir
trivy fs --scanners vuln .                        # vulns only
trivy fs --scanners secret .                      # secrets only
trivy fs --scanners config .                      # misconfigs only
trivy fs --scanners vuln,secret,config .          # all scanners

# ── REPO SCANNING ─────────────────────────────────────────────
trivy repo https://github.com/user/repo           # remote repo
trivy repo .                                      # local repo

# ── CONFIG / IaC SCANNING ────────────────────────────────────
trivy config ./Dockerfile                         # scan Dockerfile
trivy config ./k8s/                               # scan K8s manifests
trivy config ./terraform/                         # scan Terraform

# ── KUBERNETES SCANNING ───────────────────────────────────────
trivy k8s --report summary cluster                # cluster summary
trivy k8s --report all cluster                    # full details
trivy k8s --namespace default cluster             # one namespace
trivy k8s deployment/myapp                        # one resource

# ── SBOM ──────────────────────────────────────────────────────
trivy image -f cyclonedx -o sbom.json nginx       # generate SBOM
trivy sbom ./sbom.json                            # scan SBOM file

# ── OUTPUT FORMATS ────────────────────────────────────────────
trivy image nginx                                 # table (default)
trivy image -f json nginx                         # JSON
trivy image -f sarif nginx                        # SARIF
trivy image -f cyclonedx nginx                    # CycloneDX

# ── DATABASE ──────────────────────────────────────────────────
trivy image --skip-db-update nginx                # offline mode
trivy image --download-db-only nginx              # update DB only

# ── CLEANUP ───────────────────────────────────────────────────
trivy clean --all                                 # clear all cache
trivy clean --scan-cache                          # clear scan cache
```

---

## Resources

- 📖 Official Docs: [aquasecurity.github.io/trivy](https://aquasecurity.github.io/trivy)
- 🐙 GitHub: [github.com/aquasecurity/trivy](https://github.com/aquasecurity/trivy)
- 🗄️ Trivy DB: Updated every 6 hours from upstream sources
- 📦 Trivy Action: [github.com/aquasecurity/trivy-action](https://github.com/aquasecurity/trivy-action)

---


