# 🛡️ Trivy — Interview Questions & Answers

> **Coverage:** Simple → Medium → Scenario-Based | Total: 45 Questions

---

## 📋 Table of Contents

| # | Section |
|---|---------|
| 1 | [Simple / Beginner (Q1–Q15)](#section-1--simple--beginner-questions) |
| 2 | [Medium / Intermediate (Q16–Q30)](#section-2--medium--intermediate-questions) |
| 3 | [Scenario-Based (Q31–Q45)](#section-3--scenario-based-questions) |

---

## Section 1 — Simple / Beginner Questions

---

### Q1. What is Trivy?

**Answer:**

Trivy is an open-source, all-in-one security scanner made by Aqua Security.
It scans for vulnerabilities, misconfigurations, and secrets in:

- Docker container images
- File systems
- Git repositories
- Kubernetes clusters
- Infrastructure as Code (Terraform, Helm, K8s YAML)

One tool, one command — comprehensive security scanning.

---

### Q2. What does CVE stand for and what is it?

**Answer:**

**CVE = Common Vulnerabilities and Exposures**

A CVE is a publicly disclosed security vulnerability with a unique ID.

**Format:**
```
CVE-YEAR-NUMBER
Example: CVE-2021-44228  ← Log4Shell vulnerability
```

Each CVE has:
- A unique ID
- A severity rating (CRITICAL, HIGH, MEDIUM, LOW)
- A description of the vulnerability
- The fixed version

---

### Q3. What is CVSS?

**Answer:**

**CVSS = Common Vulnerability Scoring System**

It is a numeric score from **0.0 to 10.0** that tells you how severe a vulnerability is.

| Score Range | Severity |
|-------------|---------|
| 9.0 – 10.0 | 🔴 CRITICAL |
| 7.0 – 8.9 | 🟠 HIGH |
| 4.0 – 6.9 | 🟡 MEDIUM |
| 0.1 – 3.9 | 🟢 LOW |
| No score | ⚪ UNKNOWN |

Higher score = more dangerous = fix first.

---

### Q4. How do you install Trivy on Linux?

**Answer:**

```bash
# Option 1 — Install script
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
  | sh -s -- -b /usr/local/bin

# Option 2 — APT (Debian/Ubuntu)
sudo apt-get install trivy

# Option 3 — Run as Docker container
docker pull aquasec/trivy

# Verify installation
trivy --version
```

---

### Q5. How do you scan a Docker image with Trivy?

**Answer:**

```bash
# Basic scan
trivy image nginx:latest

# Scan with severity filter (recommended)
trivy image --severity HIGH,CRITICAL nginx:latest

# Scan a locally built image
trivy image my-app:v1.0

# Scan and save output to JSON
trivy image -f json -o results.json nginx:latest
```

---

### Q6. What is the command to scan a file system with Trivy?

**Answer:**

```bash
# Scan current directory
trivy fs .

# Scan a specific path
trivy fs /path/to/project

# Scan for secrets also
trivy fs --scanners secret .

# Scan for IaC misconfigurations
trivy fs --scanners config .

# Scan for everything
trivy fs --scanners vuln,secret,config .
```

---

### Q7. What is Shift Left Security?

**Answer:**

Shift Left means moving security checks **earlier** in the development pipeline.

```
Traditional (Too Late):
Dev → Build → Test → Deploy → [Security Check] → Production

Shift Left (Early Detection):
Dev → [Security Check] → Build → [Security Check] → Deploy
```

**Why it matters:**

| Stage | Cost to Fix |
|-------|------------|
| Development | 💚 Low |
| CI/CD Pipeline | 🟡 Medium |
| Staging | 🟠 High |
| Production | 🔴 Very High |

Trivy enables Shift Left by running in your IDE, Git hooks, and CI/CD pipeline.

---

### Q8. What vulnerability databases does Trivy use?

**Answer:**

Trivy pulls from multiple trusted databases:

| Database | Source |
|----------|--------|
| NVD | National Vulnerability Database (NIST) |
| GitHub Advisory DB | GitHub Security Advisories |
| OSV | Open Source Vulnerabilities |
| RedHat Security | RedHat Advisories |
| Debian / Ubuntu | Distro-specific security data |
| Alpine SecDB | Alpine Linux Security Database |
| Trivy DB | Aggregated DB by Aqua Security |

Trivy automatically updates its local DB before every scan.

---

### Q9. What is SBOM?

**Answer:**

**SBOM = Software Bill of Materials**

It is a complete inventory of all software components, libraries, and dependencies inside your application or container image.

**Why it matters:**
- Know exactly what packages are inside your image
- Quickly identify which apps are affected when a new CVE is published
- Required for compliance in many industries

```bash
# Generate SBOM in CycloneDX format
trivy image -f cyclonedx nginx:latest

# Generate SBOM in SPDX format
trivy image -f spdx-json nginx:latest
```

---

### Q10. How do you scan a Kubernetes cluster with Trivy?

**Answer:**

```bash
# Scan entire cluster
trivy k8s --report summary cluster

# Scan a specific namespace
trivy k8s --namespace production --report all

# Scan a specific resource
trivy k8s deployment/my-app

# Scan and output as table
trivy k8s --report summary --format table cluster
```

---

### Q11. What does `--exit-code 1` do in Trivy?

**Answer:**

`--exit-code 1` tells Trivy to **exit with a non-zero status code** when vulnerabilities are found.

```bash
trivy image --exit-code 1 --severity CRITICAL nginx:latest
```

**Why it matters in CI/CD:**
- `exit code 0` = no issues → pipeline continues ✅
- `exit code 1` = vulnerabilities found → pipeline fails ❌

This automatically **blocks deployments** when critical vulnerabilities are detected.

---

### Q12. What is a .trivyignore file?

**Answer:**

`.trivyignore` is a file that tells Trivy to **skip specific CVEs** during scanning — similar to `.gitignore` for Git.

```bash
# .trivyignore
CVE-2021-12345                        # ignore this CVE permanently
CVE-2022-67890 exp:2024-12-31        # ignore until this date
CVE-2023-11111 # false positive       # comment explaining why
```

**Usage:**
```bash
trivy image --ignorefile .trivyignore nginx:latest
```

**When to use:** When a CVE is a known false positive, or when a fix is not yet available and you accept the risk.

---

### Q13. What output formats does Trivy support?

**Answer:**

| Format | Command | Use Case |
|--------|---------|---------|
| Table | `trivy image nginx` | Human-readable terminal output |
| JSON | `-f json` | Automation and parsing |
| SARIF | `-f sarif` | GitHub/GitLab Security tab |
| CycloneDX | `-f cyclonedx` | SBOM generation |
| SPDX | `-f spdx-json` | SBOM compliance |
| HTML | `--format template` | Reports for stakeholders |

---

### Q14. What is the difference between `trivy image` and `trivy fs`?

**Answer:**

| Command | What it scans | When to use |
|---------|--------------|-------------|
| `trivy image` | A Docker container image (built) | After building an image |
| `trivy fs` | A local file system or directory | On source code before building |

```bash
# Scan the image after build
docker build -t myapp:latest .
trivy image myapp:latest

# Scan source code before build
trivy fs .
```

---

### Q15. How do you scan a remote GitHub repository with Trivy?

**Answer:**

```bash
# Scan a remote repo
trivy repo https://github.com/username/repo-name

# Scan local repo
trivy repo .

# Scan with all scanners (vuln + secret + config)
trivy repo --scanners vuln,secret,config https://github.com/username/repo
```

---

## Section 2 — Medium / Intermediate Questions

---

### Q16. How do you integrate Trivy into a Jenkins pipeline?

**Answer:**

```groovy
pipeline {
    agent any
    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
        stage('Security Scan — Trivy') {
            steps {
                // Fail pipeline if CRITICAL vulnerabilities found
                sh '''
                    trivy image \
                      --exit-code 1 \
                      --severity HIGH,CRITICAL \
                      --no-progress \
                      myapp:${BUILD_NUMBER}
                '''
            }
        }
        stage('Push to Registry') {
            steps {
                // Only runs if Trivy scan passed
                sh 'docker push myapp:${BUILD_NUMBER}'
            }
        }
    }
}
```

**Key point:** Put the Trivy stage **after build but before push** — so broken images never reach the registry.

---

### Q17. How do you integrate Trivy into GitHub Actions?

**Answer:**

```yaml
# .github/workflows/trivy.yml
name: Trivy Security Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  trivy-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Run Trivy scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'

      - name: Upload results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

---

### Q18. What is the difference between vulnerability scanning and misconfiguration scanning in Trivy?

**Answer:**

| Type | What Trivy Looks For | Example |
|------|---------------------|---------|
| **Vulnerability** | Known CVEs in OS packages, libraries, language packages | `CVE-2021-44228` in log4j |
| **Misconfiguration** | Security issues in config files | Running container as root, no resource limits, privileged mode in K8s YAML |

```bash
# Scan for vulnerabilities only
trivy image --scanners vuln nginx:latest

# Scan for misconfigurations only
trivy config ./k8s/

# Scan for both
trivy image --scanners vuln,config myapp:latest
```

---

### Q19. What is `--ignore-unfixed` and when would you use it?

**Answer:**

`--ignore-unfixed` tells Trivy to **skip vulnerabilities that have no available fix yet**.

```bash
trivy image --ignore-unfixed nginx:latest
```

**When to use:**
- When a CVE exists but no patch is released yet
- When you only want to see actionable vulnerabilities (ones you can actually fix)
- In daily CI scans to reduce noise

**When NOT to use:**
- In final security audits — you still need to know all risks even if unfixable

---

### Q20. How does Trivy scan IaC files? Give an example.

**Answer:**

Trivy's `config` scanner checks Terraform, Kubernetes YAML, Dockerfile, and Helm charts for security misconfigurations.

```bash
# Scan a folder of K8s manifests
trivy config ./k8s/

# Scan a single Dockerfile
trivy config ./Dockerfile

# Scan Terraform files
trivy config ./terraform/

# Scan a Helm chart
trivy config ./helm-chart/
```

**Example misconfiguration Trivy would catch in a K8s deployment:**
```yaml
# This would trigger a HIGH finding:
containers:
  - name: myapp
    securityContext:
      privileged: true       # ← Trivy flags: container running as privileged
      runAsRoot: true        # ← Trivy flags: running as root user
```

---

### Q21. What is secret scanning in Trivy and why is it important?

**Answer:**

Secret scanning detects **hardcoded credentials** in source code, configuration files, and container images.

**What Trivy detects:**
- AWS access keys / secret keys
- GitHub tokens
- Docker Hub passwords
- Private SSH keys
- API keys and tokens
- Database connection strings with passwords

```bash
# Scan for secrets in file system
trivy fs --scanners secret .

# Scan for secrets in a repo
trivy repo --scanners secret https://github.com/user/repo

# Scan image for embedded secrets
trivy image --scanners secret myapp:latest
```

**Why important:** Hardcoded secrets accidentally pushed to Git are one of the most common causes of data breaches.

---

### Q22. What is the Trivy DB and how often is it updated?

**Answer:**

The **Trivy DB** is a local vulnerability database that Trivy downloads and caches on your machine. It aggregates data from:
- NVD, GitHub Advisory, OSV, RedHat, Debian, Alpine, and more

**Update behavior:**
- Trivy **automatically updates** the DB before each scan
- DB is updated by Aqua Security **every 6 hours** from upstream sources
- Cached locally at: `~/.cache/trivy/`

```bash
# Force update the DB manually
trivy image --download-db-only

# Skip DB update (offline / air-gapped environments)
trivy image --skip-db-update nginx:latest

# Set custom cache dir
trivy image --cache-dir /tmp/trivy-cache nginx:latest
```

---

### Q23. How do you scan only for specific severity levels in Trivy?

**Answer:**

```bash
# Only CRITICAL
trivy image --severity CRITICAL nginx:latest

# CRITICAL and HIGH only
trivy image --severity HIGH,CRITICAL nginx:latest

# All severities (default behavior)
trivy image nginx:latest

# In CI/CD — fail only on CRITICAL
trivy image \
  --severity CRITICAL \
  --exit-code 1 \
  nginx:latest
```

**Best practice in CI/CD:**
- Block deployment on `CRITICAL`
- Alert/warn on `HIGH`
- Report `MEDIUM` and `LOW` without blocking

---

### Q24. How do you run Trivy in an air-gapped (offline) environment?

**Answer:**

In air-gapped environments, you cannot download the DB from the internet. Download it separately and copy it over.

```bash
# Step 1: On an internet-connected machine — download the DB
trivy image --download-db-only
# DB saved at: ~/.cache/trivy/db/

# Step 2: Copy the DB to the air-gapped server
tar czf trivy-db.tar.gz ~/.cache/trivy/db/
scp trivy-db.tar.gz user@airgapped-server:/tmp/

# Step 3: On air-gapped server — extract and use
mkdir -p ~/.cache/trivy/
tar xzf /tmp/trivy-db.tar.gz -C ~/.cache/trivy/

# Step 4: Scan with --skip-db-update
trivy image --skip-db-update myapp:latest
```

---

### Q25. What is the difference between `trivy image` scanning a remote vs local image?

**Answer:**

```bash
# Remote image — Trivy pulls from Docker Hub automatically
trivy image nginx:latest

# Local image — image must already be built on your machine
docker build -t myapp:local .
trivy image myapp:local

# Image from a tar file (saved image)
docker save myapp:local -o myapp.tar
trivy image --input myapp.tar
```

**Key difference:** For remote images, Trivy can scan without Docker being installed using its own layer downloading mechanism.

---

### Q26. How do you generate a security report with Trivy for a team / manager?

**Answer:**

```bash
# HTML report (best for sharing with non-technical stakeholders)
trivy image \
  --format template \
  --template "@contrib/html.tpl" \
  -o security-report.html \
  nginx:latest

# JSON report (for automated parsing or dashboards)
trivy image -f json -o results.json nginx:latest

# SARIF (for GitHub Security tab — team can review in GitHub UI)
trivy image -f sarif -o results.sarif nginx:latest

# Table with only HIGH and CRITICAL (clean executive summary)
trivy image --severity HIGH,CRITICAL nginx:latest
```

---

### Q27. What scanners does Trivy support and how do you combine them?

**Answer:**

| Scanner | Flag | What it finds |
|---------|------|--------------|
| `vuln` | `--scanners vuln` | CVEs in packages and libraries |
| `config` | `--scanners config` | Misconfigurations in YAML, Dockerfile |
| `secret` | `--scanners secret` | Hardcoded credentials and API keys |
| `license` | `--scanners license` | Unapproved open-source licenses |

```bash
# Run all scanners at once
trivy image --scanners vuln,config,secret myapp:latest

# Run only vulnerability + secret scanning
trivy fs --scanners vuln,secret .

# Default (vuln only for images)
trivy image nginx:latest
```

---

### Q28. How do you use Trivy with GitLab CI?

**Answer:**

```yaml
# .gitlab-ci.yml
stages:
  - build
  - security
  - deploy

build-image:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

trivy-scan:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy image
        --exit-code 1
        --severity HIGH,CRITICAL
        --no-progress
        $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  allow_failure: false     # fail the pipeline on vulnerabilities

deploy:
  stage: deploy
  script:
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - main
```

---

### Q29. What is the difference between OS-level and library-level CVE scanning in Trivy?

**Answer:**

| Level | What Trivy Scans | Examples |
|-------|-----------------|---------|
| **OS-level** | Packages installed by the OS package manager | `apt`, `yum`, `apk` packages (curl, openssl, bash) |
| **Library-level** | Language-specific packages | npm (Node.js), pip (Python), Maven (Java), Gemfile (Ruby) |

```bash
# Trivy scans BOTH automatically in one command
trivy image myapp:latest

# Example output:
# OS packages:    openssl 1.1.1f  →  CVE-2022-0778  HIGH
# Node.js packages: express 4.17.1 →  CVE-2022-24999 HIGH
```

**Why it matters:** A clean OS base image can still have critical CVEs in your `package.json` dependencies.

---

### Q30. What happens if Trivy finds 0 vulnerabilities — does that mean the image is 100% safe?

**Answer:**

**No.** Zero CVEs does not mean 100% secure.

Trivy only finds:
- **Known** vulnerabilities (already in the CVE database)
- **Zero-day vulnerabilities** (not yet disclosed) are NOT detected
- **Logic flaws** in your application code are NOT detected
- **Runtime misconfigurations** that only appear under specific conditions may be missed

**What Trivy zero result means:**
- No known CVEs in the scanned components at the time of scan
- Misconfigurations and secrets may still exist (run those scanners too)
- You should re-scan regularly because new CVEs are published daily

**Best practice:** Zero CVEs + `--scanners vuln,config,secret` = reasonably secure at scan time. Re-scan daily.

---

## Section 3 — Scenario-Based Questions

---

### Q31. Your Jenkins pipeline deployed a container with a Log4Shell (CVE-2021-44228) vulnerability to production. How would you prevent this from happening again?

**Answer:**

**Immediate fix:**
```bash
# Identify affected containers in production
trivy image --severity CRITICAL \
  --format json myapp:latest | grep "CVE-2021-44228"

# Rebuild image with patched Log4j version
# Update pom.xml or build.gradle to log4j 2.17.1+
docker build -t myapp:patched .
```

**Prevent recurrence — add to Jenkins pipeline:**
```groovy
stage('Trivy Security Gate') {
    steps {
        sh '''
            trivy image \
              --exit-code 1 \
              --severity CRITICAL \
              --ignore-unfixed \
              myapp:${BUILD_NUMBER}
        '''
    }
    post {
        failure {
            error "CRITICAL vulnerabilities found — deployment blocked!"
        }
    }
}
```

**Policy changes:**
- Add Trivy scan as a required pipeline stage — merge blocked without it
- Set `--exit-code 1` for CRITICAL so deployments auto-fail
- Add `.trivyignore` only with documented, approved exceptions
- Schedule daily re-scans of all production images

---

### Q32. A developer complains that Trivy is blocking their pull request because of a CVE that has no fix yet. How do you handle this?

**Answer:**

**Step 1 — Verify it is truly unfixed:**
```bash
trivy image --severity HIGH,CRITICAL nginx:latest | grep "CVE-XXXX-XXXXX"
# Check "Fixed Version" column — if empty, no fix exists yet
```

**Step 2 — Assess the risk:**
- Is the vulnerable code path actually reachable in your app?
- Is there a workaround (config change, WAF rule)?

**Step 3 — Options:**

```bash
# Option A: Use --ignore-unfixed in CI (skip CVEs with no fix)
trivy image --ignore-unfixed --severity HIGH,CRITICAL myapp:latest

# Option B: Add to .trivyignore with expiry date and reason
# .trivyignore
CVE-2023-XXXXX exp:2024-06-30  # no fix available, monitored weekly
```

**Step 4 — Document the exception:**
- Track it in your vulnerability register
- Set a calendar reminder to re-check weekly
- Remove from `.trivyignore` as soon as a fix is available

**Never ignore permanently without review.**

---

### Q33. Your Docker image is 2 GB. Trivy reports 47 HIGH and 12 CRITICAL vulnerabilities. How do you approach fixing them systematically?

**Answer:**

**Step 1 — Get structured output:**
```bash
trivy image -f json -o vuln-report.json myapp:latest
trivy image --severity CRITICAL,HIGH myapp:latest
```

**Step 2 — Fix CRITICAL first (12 issues):**
```bash
# Find which packages have CRITICAL CVEs
trivy image --severity CRITICAL myapp:latest

# Common fix: upgrade the base image
# Change FROM ubuntu:20.04 → ubuntu:22.04 in Dockerfile
# Or switch to a smaller, more secure base:
# FROM ubuntu:20.04 → FROM alpine:3.19
```

**Step 3 — Reduce image size (fewer packages = fewer CVEs):**
```dockerfile
# Before: large base image with many packages
FROM ubuntu:20.04

# After: minimal base image
FROM alpine:3.19
# OR use distroless (no shell, minimal attack surface)
FROM gcr.io/distroless/java21:latest
```

**Step 4 — Fix library CVEs:**
```bash
# Update dependencies in package.json / pom.xml / requirements.txt
npm audit fix       # Node.js
pip install --upgrade package-name   # Python
```

**Step 5 — Rebuild and re-scan:**
```bash
docker build -t myapp:patched .
trivy image --severity CRITICAL,HIGH myapp:patched
# Target: 0 CRITICAL, minimal HIGH
```

---

### Q34. You are setting up DevSecOps for a new project from scratch. How would you integrate Trivy at every stage?

**Answer:**

**Stage 1 — Local Development (Pre-commit):**
```bash
# Developer scans before committing
trivy fs --scanners vuln,secret,config .

# Add to pre-commit hook (.git/hooks/pre-commit)
#!/bin/bash
trivy fs --exit-code 1 --severity CRITICAL .
```

**Stage 2 — Pull Request (GitHub Actions):**
```yaml
- name: Trivy PR scan
  uses: aquasecurity/trivy-action@master
  with:
    scan-type: 'fs'
    scan-ref: '.'
    severity: 'CRITICAL,HIGH'
    exit-code: '1'
```

**Stage 3 — CI/CD Pipeline (Jenkins/GitLab):**
```bash
# After docker build, before push
trivy image --exit-code 1 --severity CRITICAL myapp:${BUILD_NUMBER}
```

**Stage 4 — Container Registry:**
- Enable Trivy scanning in Docker Hub, ECR, or Harbor
- Block images with CRITICAL CVEs from being pulled

**Stage 5 — Production / Kubernetes:**
```bash
# Daily scheduled scan of running images
trivy k8s --report summary cluster
```

**Result:** Security checked at every stage — code → PR → build → registry → runtime.

---

### Q35. A newly published CVE affects a package in 50+ container images across your organization. How do you identify all affected images quickly?

**Answer:**

```bash
# Step 1: Find the CVE details first
trivy image --format json nginx:latest | \
  python3 -c "import sys,json; \
  [print(v['VulnerabilityID'], v['PkgName']) \
  for r in json.load(sys.stdin)['Results'] \
  for v in r.get('Vulnerabilities', []) \
  if v['VulnerabilityID']=='CVE-XXXX-XXXXX']"

# Step 2: Script to scan all images
#!/bin/bash
IMAGES=("service-a:latest" "service-b:latest" "service-c:latest")
CVE_ID="CVE-2021-44228"

for IMAGE in "${IMAGES[@]}"; do
    echo "Scanning $IMAGE..."
    RESULT=$(trivy image --format json "$IMAGE" | \
        grep "$CVE_ID")
    if [ -n "$RESULT" ]; then
        echo "AFFECTED: $IMAGE contains $CVE_ID"
    fi
done

# Step 3: Fix → rebuild → re-scan → redeploy affected services
# Step 4: Document findings and remediation in incident report
```

---

### Q36. Trivy is slowing down your CI/CD pipeline significantly. How do you speed it up without reducing security coverage?

**Answer:**

```bash
# Problem: Trivy downloads the DB on every scan → slow!

# Fix 1: Cache the Trivy DB in CI
# GitHub Actions example:
- name: Cache Trivy DB
  uses: actions/cache@v3
  with:
    path: ~/.cache/trivy
    key: trivy-db-${{ github.run_id }}
    restore-keys: trivy-db-

- name: Run Trivy
  run: trivy image --skip-db-update myapp:latest
  # Uses cached DB — much faster!

# Fix 2: Only scan on specific triggers (not every commit)
on:
  push:
    branches: [main]        # scan on merge to main
  schedule:
    - cron: '0 2 * * *'    # daily scan at 2 AM

# Fix 3: Use --skip-files to exclude known-clean paths
trivy image --skip-files "*.test.js" myapp:latest

# Fix 4: Run in parallel with build (don't wait for build to finish)
# Scan the previous version while new one is building

# Fix 5: In PR checks scan only changed files
trivy fs --scanners secret,config .    # fast, no DB download needed
```

---

### Q37. Your security team requires all production images to have zero CRITICAL CVEs. A critical CVE is found 2 hours before a release. What do you do?

**Answer:**

**Step 1 — Assess immediately (10 min):**
```bash
trivy image --severity CRITICAL -f json myapp:release | \
  grep -E "VulnerabilityID|PkgName|FixedVersion|Severity"
```

Questions to answer:
- What package is affected?
- Is the fixed version available?
- Is the vulnerable code path actually used in our app?

**Step 2A — If fix is available:**
```bash
# Update the package in Dockerfile or dependency file
# Example: upgrade the base image
docker build --no-cache -t myapp:release-patched .
trivy image --severity CRITICAL myapp:release-patched
# If 0 CRITICAL → proceed with release
```

**Step 2B — If NO fix available:**
- Document the risk formally (CVE ID, severity, exploitability)
- Get sign-off from security lead and product owner
- Add to `.trivyignore` with expiry date and reason
- Add to security risk register for tracking
- Deploy with compensating controls (WAF, network policy)

**Step 3 — Prevent recurrence:**
- Add Trivy scan 48h before release deadline — not 2h before
- Schedule nightly scans on release candidates

---

### Q38. How would you handle Trivy false positives — CVEs reported but not actually applicable to your application?

**Answer:**

**What is a false positive?**
Trivy reports a CVE in a package, but the vulnerable function in that package is never called by your app.

**How to handle:**

**Step 1 — Verify it is truly a false positive:**
```bash
# Check if the vulnerable code path exists in your app
trivy image -f json myapp:latest | grep "CVE-XXXX"
# Read the CVE description carefully
# Check if your code calls the vulnerable function
```

**Step 2 — Document and suppress with expiry:**
```bash
# .trivyignore
CVE-2023-12345  exp:2024-12-31  # false positive — uses only safe API, not vulnerable function
```

**Step 3 — Use VEX (Vulnerability Exploitability eXchange):**
```bash
# Trivy supports VEX documents to formally state non-applicability
trivy image --vex vex.json myapp:latest
```

**Step 4 — Review regularly:**
- Re-evaluate false positives quarterly
- When the package is upgraded, verify if the CVE is still not applicable

**Key rule:** Every `.trivyignore` entry must have a written justification and expiry date. No permanent ignores without approval.

---

### Q39. You are asked to implement a security gate that allows HIGH vulnerabilities in development but blocks them in production. How do you set this up?

**Answer:**

```groovy
// Jenkinsfile — environment-aware Trivy policy
pipeline {
    agent any

    environment {
        IMAGE_TAG = "myapp:${BUILD_NUMBER}"
    }

    stages {
        stage('Build') {
            steps {
                sh "docker build -t ${IMAGE_TAG} ."
            }
        }

        stage('Trivy Scan — Dev Policy') {
            when {
                branch 'develop'
            }
            steps {
                // Dev: only block on CRITICAL
                sh """
                    trivy image \
                      --exit-code 1 \
                      --severity CRITICAL \
                      ${IMAGE_TAG}
                """
            }
        }

        stage('Trivy Scan — Prod Policy') {
            when {
                branch 'main'
            }
            steps {
                // Production: block on HIGH and CRITICAL
                sh """
                    trivy image \
                      --exit-code 1 \
                      --severity HIGH,CRITICAL \
                      --ignore-unfixed \
                      ${IMAGE_TAG}
                """
            }
        }
    }
}
```

**GitHub Actions version using environments:**
```yaml
- name: Trivy Scan
  run: |
    if [ "${{ github.ref }}" == "refs/heads/main" ]; then
      SEVERITY="HIGH,CRITICAL"
    else
      SEVERITY="CRITICAL"
    fi
    trivy image --exit-code 1 --severity $SEVERITY myapp:latest
```

---

### Q40. A developer pushed a Docker image with AWS credentials hardcoded in the Dockerfile ENV. How does Trivy help and what do you do?

**Answer:**

**How Trivy detects it:**
```bash
# Secret scanning finds hardcoded credentials
trivy image --scanners secret myapp:latest

# Output:
# HIGH: AWS Access Key ID found
# File: Dockerfile
# Match: AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
```

**Immediate response (treat as security incident):**
```bash
# Step 1: Rotate the exposed AWS credentials IMMEDIATELY
# Go to AWS IAM → Revoke the compromised key → Create new key

# Step 2: Remove the image from ALL registries
docker rmi myapp:compromised
# Also delete from Docker Hub / ECR

# Step 3: Check AWS CloudTrail for unauthorized usage
# (Was the key used maliciously?)

# Step 4: Rebuild image correctly using secrets
```

**Correct way to pass secrets:**
```dockerfile
# WRONG — secret in ENV
ENV AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE

# CORRECT — pass at runtime, never in image
docker run -e AWS_ACCESS_KEY_ID=$AWS_KEY myapp:latest

# BEST — use AWS IAM roles for EC2/ECS/EKS
# No credentials needed at all!
```

**Prevent recurrence:**
```bash
# Add to pre-commit hook and CI
trivy fs --scanners secret --exit-code 1 .
```

---

### Q41. How would you use Trivy to ensure Kubernetes manifests follow security best practices before deploying?

**Answer:**

```bash
# Scan all K8s manifests before applying
trivy config ./k8s/

# What Trivy checks in K8s YAML:
# - Containers running as root
# - Privileged containers
# - Missing resource limits
# - Containers with hostPID / hostNetwork
# - Writable root filesystem
# - Missing readinessProbe / livenessProbe
# - Capabilities not dropped
```

**Example misconfiguration Trivy catches:**
```yaml
# deployment.yml — Trivy would flag these:
containers:
  - name: myapp
    securityContext:
      privileged: true        # CRITICAL: privileged container
      runAsUser: 0            # HIGH: running as root
    resources: {}             # MEDIUM: no resource limits set
```

**Integrate into GitOps pipeline:**
```bash
# In CI — scan before kubectl apply
trivy config ./k8s/ --exit-code 1 --severity HIGH,CRITICAL

# Only deploy if scan passes
kubectl apply -f ./k8s/
```

---

### Q42. Your team uses Helm charts to deploy to Kubernetes. How do you scan Helm charts with Trivy?

**Answer:**

```bash
# Scan a Helm chart directory
trivy config ./my-helm-chart/

# Scan with values file
helm template my-release ./my-chart -f values-prod.yaml | \
  trivy config --file-patterns '*.yaml' -

# Render Helm chart first, then scan rendered YAML
helm template my-release ./my-chart > rendered.yaml
trivy config rendered.yaml

# Scan with strict mode — fail on any misconfiguration
trivy config --exit-code 1 ./my-helm-chart/
```

**In CI/CD pipeline:**
```groovy
stage('Scan Helm Chart') {
    steps {
        sh '''
            helm template myapp ./helm/ -f values-prod.yaml > rendered.yaml
            trivy config \
              --exit-code 1 \
              --severity HIGH,CRITICAL \
              rendered.yaml
        '''
    }
}
```

---

### Q43. How would you set up automated daily Trivy scanning of all running production container images?

**Answer:**

```bash
#!/bin/bash
# daily-scan.sh — runs via cron every day at 2 AM

# Get all running image names from Kubernetes
IMAGES=$(kubectl get pods --all-namespaces \
  -o jsonpath='{.items[*].spec.containers[*].image}' | \
  tr ' ' '\n' | sort | uniq)

REPORT_FILE="/tmp/trivy-daily-$(date +%F).json"
CRITICAL_COUNT=0

for IMAGE in $IMAGES; do
    echo "Scanning: $IMAGE"
    RESULT=$(trivy image \
        --severity HIGH,CRITICAL \
        --format json \
        --quiet \
        "$IMAGE")

    # Count CRITICAL findings
    COUNT=$(echo $RESULT | \
        python3 -c "import sys,json; \
        data=json.load(sys.stdin); \
        print(sum(len([v for v in r.get('Vulnerabilities',[]) \
        if v['Severity']=='CRITICAL']) \
        for r in data.get('Results',[])))")

    if [ "$COUNT" -gt "0" ]; then
        echo "ALERT: $IMAGE has $COUNT CRITICAL CVEs!"
        CRITICAL_COUNT=$((CRITICAL_COUNT + COUNT))
        # Send Slack/email alert here
    fi
done

echo "Daily scan complete. Total CRITICAL: $CRITICAL_COUNT"
```

**Cron setup:**
```bash
# Run daily at 2 AM
0 2 * * * /opt/scripts/daily-scan.sh >> /var/log/trivy-scan.log 2>&1
```

---

### Q44. You are presenting Trivy's value to a non-technical manager. How do you explain it?

**Answer:**

**Simple business-focused explanation:**

> "Trivy is like a health check for our software before it goes live.
>
> Every day, thousands of new security vulnerabilities are discovered in software components. Without scanning, we wouldn't know if our applications contain known vulnerabilities that hackers could exploit.
>
> Trivy automatically checks every container we build — in under 60 seconds — and tells us if it contains any known security risks, categorized by severity.
>
> We then block anything with critical risks from deploying to production automatically.
>
> **Business impact:**
> - Prevents security breaches before they happen
> - Finds issues when they cost almost nothing to fix (development) vs. millions after a breach
> - Gives us audit evidence for compliance (SOC2, ISO 27001, PCI-DSS)
> - Reduces manual security review time by 80%"

**ROI numbers to mention:**
- Average cost of a data breach in 2024: **$4.88 million** (IBM report)
- Cost to fix a bug in development: **$80**
- Cost to fix the same bug in production: **$7,600**

---

### Q45. Trivy scan shows a CRITICAL CVE in a base image that is maintained by a third party and you cannot change it. What are your options?

**Answer:**

**Option 1 — Find an alternative base image:**
```dockerfile
# Vulnerable base
FROM python:3.9       # has CRITICAL CVEs

# Try alternatives
FROM python:3.9-slim   # smaller, fewer packages
FROM python:3.9-alpine # minimal attack surface
FROM gcr.io/distroless/python3  # Google distroless — almost no packages
```

**Option 2 — Override the vulnerable package inside your Dockerfile:**
```dockerfile
FROM python:3.9

# Upgrade the vulnerable package even if base image doesn't
RUN pip install --upgrade vulnerable-package==safe-version

# Or for OS packages
RUN apt-get update && apt-get upgrade -y && \
    rm -rf /var/lib/apt/lists/*
```

**Option 3 — Suppress with documented justification:**
```bash
# .trivyignore
CVE-2023-XXXXX exp:2024-06-30  # third-party base image, vendor aware, monitoring fix ETA
```

**Option 4 — Apply compensating controls:**
- WAF rule to block exploitation attempts
- Network policy to limit container's network access
- AppArmor/seccomp profile to restrict syscalls

**Option 5 — Apply pressure to the vendor:**
- Open a GitHub issue on the base image repo
- Subscribe to their security advisory channel
- Set a calendar reminder to upgrade when they patch it

**Document your decision** with risk assessment, approved by security lead.

---

## 📝 Quick Revision Summary

| Topic | Key Point |
|-------|-----------|
| What is Trivy | Open-source security scanner for images, FS, K8s, IaC, repos |
| CVE | Unique ID for a known vulnerability (CVE-YEAR-NUMBER) |
| CVSS | Numeric severity score 0–10 |
| Shift Left | Find bugs early when they are cheap to fix |
| `--exit-code 1` | Fails CI/CD pipeline when vulnerabilities found |
| `--ignore-unfixed` | Skip CVEs with no available fix |
| `.trivyignore` | Suppress specific CVEs (must document reason + expiry) |
| SBOM | Full inventory of all packages in your software |
| Air-gapped | Download DB separately + use `--skip-db-update` |
| False positives | Suppress with `.trivyignore` + VEX, always with justification |
| Scanners | `vuln`, `config`, `secret`, `license` — combine with `--scanners` |
| DB update | Every 6 hours by Aqua Security — cached locally |

---

