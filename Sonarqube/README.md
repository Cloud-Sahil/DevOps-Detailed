# SonarQube Complete Study Guide

> A complete, modular SonarQube study guide — architecture, installation, quality gates/profiles, scanners, rules/issues, security analysis, branch/PR analysis, CI/CD integration, administration, and performance — with diagrams and commands.

---

## Table of Contents

1. [SonarQube Fundamentals & Architecture](#module-1-sonarqube-fundamentals--architecture)
2. [Installation & Setup](#module-2-installation--setup)
3. [Quality Profiles & Quality Gates](#module-3-quality-profiles--quality-gates)
4. [Scanners & Code Analysis](#module-4-scanners--code-analysis)
5. [Rules, Issues & Technical Debt](#module-5-rules-issues--technical-debt)
6. [Code Coverage Integration](#module-6-code-coverage-integration)
7. [Security Analysis: SAST, Hotspots, Vulnerabilities](#module-7-security-analysis-sast-hotspots-vulnerabilities)
8. [Branch Analysis & Pull Request Decoration](#module-8-branch-analysis--pull-request-decoration)
9. [CI/CD Integration](#module-9-cicd-integration)
10. [Administration: Users, Permissions, Projects](#module-10-administration-users-permissions-projects)
11. [SonarQube vs SonarCloud vs SonarLint](#module-11-sonarqube-vs-sonarcloud-vs-sonarlint)
12. [Database & Performance Tuning](#module-12-database--performance-tuning)
13. [Plugins & Language Support](#module-13-plugins--language-support)
14. [Best Practices & Enterprise Rollout](#module-14-best-practices--enterprise-rollout)

---

## MODULE 1: SonarQube Fundamentals & Architecture

### 1.1 What is SonarQube?

SonarQube is a self-managed static code analysis platform (SAST — Static Application Security Testing) that continuously inspects code quality and security across 30+ programming languages, surfacing bugs, vulnerabilities, code smells, and technical debt directly in developers' workflow.

---

### 1.2 SonarQube Architecture

```
                Developer / CI Pipeline
                          │
                  Scanner (sonar-scanner)
                  (runs locally or in CI, analyzes source code)
                          │
                Sends analysis report
                          │
                 SonarQube Server
        ┌─────────────────┼─────────────────┐
   Web Server         Compute Engine      Search Server
  (UI, API,           (processes and       (Elasticsearch —
   Web Server           stores analysis      indexes issues
   process)             reports, computes    for fast search/
                         measures/ratings)    filtering in UI)
                          │
                    Database (PostgreSQL)
              (stores configuration, issues, history)
```

- **Scanner** — the client-side tool that analyzes source code and sends the analysis report to the server.
- **Compute Engine** — processes incoming analysis reports asynchronously, computing measures, ratings, and applying Quality Gates.
- **Search Server (Elasticsearch)** — powers fast issue search/filtering in the UI.
- **Database** — stores all configuration, historical measures, and issue data (PostgreSQL is the only supported production database as of recent versions).

---

### 1.3 SonarQube Editions

| Edition | Key Capabilities |
|---|---|
| Community Edition | Free, core static analysis, limited language support depth |
| Developer Edition | Adds more languages (C#, PL/SQL), branch analysis, PR decoration |
| Enterprise Edition | Adds more languages (COBOL, ABAP), portfolio management, executive reporting |
| Data Center Edition | Adds high availability, horizontal scalability for very large organizations |

---

### 1.4 Key Terminology

| Term | Meaning |
|---|---|
| Issue | A specific problem detected in code (bug, vulnerability, code smell) |
| Rule | The definition/logic that detects a specific issue pattern |
| Quality Profile | A collection of rules enabled for a given language |
| Quality Gate | Pass/fail criteria applied to a project's analysis (e.g., "no new bugs") |
| Technical Debt | Estimated time to fix all maintainability issues in the codebase |
| Hotspot | Security-sensitive code requiring manual review (not automatically a "vulnerability") |

---

## MODULE 2: Installation & Setup

### 2.1 System Requirements Overview

SonarQube requires: a supported JDK (bundled in recent versions), PostgreSQL as the production database, sufficient file descriptors/virtual memory (especially for the embedded Elasticsearch component), and adequate heap memory allocated to each internal process (Web, CE, Search).

```bash
# Elasticsearch requires increased vm.max_map_count on Linux
sudo sysctl -w vm.max_map_count=524288
echo "vm.max_map_count=524288" | sudo tee -a /etc/sysctl.conf

ulimit -n 65536    # increase file descriptor limit
```

---

### 2.2 Installing SonarQube (Standalone)

```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-<version>.zip
unzip sonarqube-<version>.zip -d /opt/
/opt/sonarqube/bin/linux-x86-64/sonar.sh start
```

```properties
# conf/sonar.properties — key database configuration
sonar.jdbc.username=sonar
sonar.jdbc.password=<password>
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```

**Important:** Never run SonarQube as the `root` user — it explicitly refuses to start as root for security reasons; create a dedicated `sonarqube` system user.

---

### 2.3 Installing SonarQube via Docker

```bash
docker run -d --name sonarqube \
  -p 9000:9000 \
  -e SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonar \
  -e SONAR_JDBC_USERNAME=sonar \
  -e SONAR_JDBC_PASSWORD=sonar \
  sonarqube:lts-community
```

For production, always externalize the database (never rely on the embedded H2 database, which is explicitly for evaluation/demo purposes only and is not supported for production use).

---

### 2.4 First-Time Setup

```bash
# Default credentials on first login (must be changed immediately)
Username: admin
Password: admin
```

After initial login, generate a project analysis token (Administration → Security → Users → Tokens) for use in CI pipelines instead of embedding the admin password directly in scanner invocations.

---

## MODULE 3: Quality Profiles & Quality Gates

### 3.1 What is a Quality Profile?

A Quality Profile is a set of activated rules for a specific language — every project is assigned a Quality Profile per language it contains, determining exactly which issues get flagged during analysis.

```
Quality Profile: "Sonar way" (default, built-in)
        │
   Contains hundreds of rules for a language (e.g., Java)
        │
   Assigned to a Project
        │
   Determines which issues are raised during analysis
```

You can create custom Quality Profiles (e.g., "MyOrg Java Strict") by extending a built-in profile and activating/deactivating specific rules to match organizational standards.

---

### 3.2 What is a Quality Gate?

A Quality Gate is a set of pass/fail conditions evaluated after each analysis, determining whether a project's code meets the required quality bar — commonly used as a CI pipeline gate blocking merges/deployments if conditions aren't met.

```
Analysis Complete
        │
Quality Gate Conditions Evaluated
        │
   ┌────┴────┐
 PASSED    FAILED
   │           │
CI proceeds  CI pipeline fails
```

**Default Quality Gate ("Sonar way") conditions** focus specifically on **new code** (not the entire codebase):
- New bugs = 0
- New vulnerabilities = 0
- New security hotspots reviewed = 100%
- New code coverage ≥ 80%
- New duplicated lines < 3%
- Maintainability rating on new code = A

---

### 3.3 Why Quality Gates Focus on "New Code" (The Clean as You Code Philosophy)

Rather than demanding an entire legacy codebase retroactively meet a high bar (often practically impossible and demotivating), SonarQube's recommended approach — "Clean as You Code" — only enforces quality standards on newly written or modified code, letting teams incrementally improve overall codebase health over time without being blocked by pre-existing legacy debt.

```bash
# Define what counts as "new code" — commonly a time period or reference branch
sonar.newCode.referenceBranch=main
```

---

### 3.4 Creating a Custom Quality Gate

```
Quality Gates → Create → Add Conditions:
  - Coverage on New Code < 80% → FAIL
  - Duplicated Lines on New Code > 3% → FAIL
  - Maintainability Rating on New Code worse than A → FAIL
  - Security Rating on New Code worse than A → FAIL
```

Assign the custom Quality Gate to specific projects (Project Settings → Quality Gate), or set it as the organization-wide default.

---

## MODULE 4: Scanners & Code Analysis

### 4.1 Types of SonarQube Scanners

| Scanner | Use Case |
|---|---|
| `sonar-scanner` (CLI) | Generic, language-agnostic, most common |
| `sonar-scanner-maven` | Maven-based Java projects |
| `sonar-scanner-gradle` | Gradle-based Java projects |
| `sonar-scanner-msbuild` (.NET) | .NET/C# projects |
| `sonar-scanner-cli` for JS/TS | Node.js/JavaScript/TypeScript projects |

---

### 4.2 Basic `sonar-project.properties` Configuration

```properties
sonar.projectKey=my-app
sonar.projectName=My Application
sonar.sources=src
sonar.tests=test
sonar.sourceEncoding=UTF-8
sonar.host.url=https://sonarqube.mycompany.com
sonar.login=<token>
```

```bash
sonar-scanner \
  -Dsonar.projectKey=my-app \
  -Dsonar.sources=. \
  -Dsonar.host.url=https://sonarqube.mycompany.com \
  -Dsonar.login=$SONAR_TOKEN
```

---

### 4.3 Maven Integration

```bash
mvn clean verify sonar:sonar \
  -Dsonar.host.url=https://sonarqube.mycompany.com \
  -Dsonar.login=$SONAR_TOKEN
```

Maven-based analysis automatically detects source directories, test directories, and (with proper configuration) code coverage reports without needing a separate `sonar-project.properties` file.

---

### 4.4 What Happens During Analysis (High-Level Flow)

```
1. Scanner reads source code + configuration
2. Scanner runs language-specific analyzers (detecting issues per active Quality Profile rules)
3. Scanner packages results into an analysis report
4. Report is uploaded to the SonarQube server
5. Compute Engine processes the report asynchronously (background task)
6. Measures/ratings computed, Quality Gate evaluated
7. Results become visible in the SonarQube UI / available via API/webhook
```

**Interview point:** Analysis is not fully synchronous — the scanner uploads the report and exits quickly, but the actual processing happens asynchronously in the Compute Engine, which is why there's a brief delay before results appear (and why CI pipelines needing to gate on the Quality Gate result must explicitly wait/poll for Compute Engine task completion).

---

### 4.5 Incremental / PR Analysis vs Full Analysis

Full analysis scans the entire codebase; branch/PR analysis focuses specifically on comparing new/changed code against the target branch — critical for the "Clean as You Code" model, since it's what allows Quality Gates to evaluate only new code rather than the whole history.

---

## MODULE 5: Rules, Issues & Technical Debt

### 5.1 Issue Types in SonarQube

| Type | Description |
|---|---|
| Bug | A coding mistake that will likely cause incorrect behavior |
| Vulnerability | A security-related weakness that could be exploited |
| Code Smell | A maintainability issue — not necessarily wrong, but harder to maintain/understand |
| Security Hotspot | Security-sensitive code requiring manual human review to determine if it's actually exploitable |

---

### 5.2 Issue Severity Levels

```
BLOCKER > CRITICAL > MAJOR > MINOR > INFO
```

Severity indicates the potential impact of an issue if left unaddressed — BLOCKER issues typically represent the highest-risk problems (e.g., a bug very likely to cause a crash or data corruption in production).

---

### 5.3 Ratings: Reliability, Security, Maintainability

SonarQube computes an A-E letter rating for each of these three dimensions based on the issues present:

```
A = No issues of that type
B = At least one Minor issue
C = At least one Major issue
D = At least one Critical issue
E = At least one Blocker issue
```

---

### 5.4 Technical Debt and the Technical Debt Ratio

Technical Debt is the estimated remediation time (in effort/time units) for all maintainability issues (code smells) in the codebase — the Technical Debt Ratio compares this estimated remediation cost against the estimated cost to rewrite the code entirely, giving a percentage indicating how "indebted" the codebase is.

---

### 5.5 Marking Issues: False Positive vs Won't Fix vs Resolved

```
False Positive → the rule incorrectly flagged something that isn't actually a problem in this context
Won't Fix       → it IS a real issue, but the team has decided not to address it (documented decision)
Fixed           → resolved through an actual code change
```

Both "False Positive" and "Won't Fix" remove the issue from active counts/Quality Gate evaluation, but they mean fundamentally different things and should be used deliberately — marking something "False Positive" when it's actually a legitimate but low-priority issue misrepresents the codebase's real health.

---

## MODULE 6: Code Coverage Integration

### 6.1 How SonarQube Handles Code Coverage

SonarQube does **not** run your tests or generate coverage data itself — it only **imports** coverage reports generated by your language's native test/coverage tooling (e.g., JaCoCo for Java, Coverage.py for Python, Istanbul/nyc for JavaScript).

```
Your test framework runs tests + generates coverage report (e.g., jacoco.xml)
        │
Scanner reads the coverage report path (sonar.coverage.jacoco.xmlReportPaths)
        │
SonarQube imports and displays coverage alongside its own static analysis findings
```

---

### 6.2 Example: JaCoCo Integration for a Java/Maven Project

```xml
<!-- pom.xml -->
<plugin>
  <groupId>org.jacoco</groupId>
  <artifactId>jacoco-maven-plugin</artifactId>
  <executions>
    <execution>
      <goals><goal>prepare-agent</goal></goals>
    </execution>
    <execution>
      <id>report</id>
      <phase>test</phase>
      <goals><goal>report</goal></goals>
    </execution>
  </executions>
</plugin>
```

```properties
sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
```

---

### 6.3 Common Coverage Gap: "0% Coverage" Despite Tests Passing

This almost always means the coverage report either wasn't generated at all before the scanner ran, or the path configured in `sonar.coverage.jacoco.xmlReportPaths` (or the language-appropriate equivalent) doesn't match where the report was actually written — always verify the report file exists at the expected path **before** the scanner step runs in CI.

---

## MODULE 7: Security Analysis: SAST, Hotspots, Vulnerabilities

### 7.1 What Makes SonarQube a SAST Tool?

SonarQube performs Static Application Security Testing — analyzing source code without executing it, to detect security vulnerabilities (like SQL injection patterns, hardcoded credentials, insecure cryptographic usage) directly tied to industry standards like OWASP Top 10 and CWE.

---

### 7.2 Security Hotspots vs Vulnerabilities — A Key Distinction

```
Vulnerability     → SonarQube is confident this is an actual exploitable security problem
Security Hotspot  → SonarQube flags security-SENSITIVE code, but requires a HUMAN
                     to review the context and decide if it's actually exploitable
```

Example: use of a cryptographic function might be flagged as a Hotspot because it's *potentially* risky depending on context (e.g., is this used for a security-critical purpose or just a checksum?) — a human reviewer must mark it "Safe" or "Fixed" (if the code needs remediation) rather than the tool automatically deciding.

---

### 7.3 Security-Related Quality Gate Conditions

```
New Security Hotspots Reviewed = 100%   ← ensures no hotspot goes unexamined by a human
New Vulnerabilities = 0                  ← blocks introducing new confirmed vulnerabilities
Security Rating on New Code = A          ← no unresolved vulnerabilities in new code
```

---

### 7.4 OWASP Top 10 / CWE Mapping

SonarQube tags rules with relevant security standards (OWASP Top 10 categories, CWE identifiers), letting security teams filter/report on issues specifically relevant to compliance frameworks they need to demonstrate coverage against.

---

## MODULE 8: Branch Analysis & Pull Request Decoration

### 8.1 Branch Analysis

```bash
sonar-scanner -Dsonar.branch.name=feature/new-login
```

Analyzing a feature branch separately from `main` lets teams see issues specific to that branch's changes before merging — requires Developer Edition or above (not available in Community Edition).

---

### 8.2 Pull Request Decoration

```bash
sonar-scanner \
  -Dsonar.pullrequest.key=42 \
  -Dsonar.pullrequest.branch=feature/new-login \
  -Dsonar.pullrequest.base=main
```

PR Decoration posts analysis results (new issues, Quality Gate status) directly as comments/checks on the GitHub/GitLab/Bitbucket/Azure DevOps pull request — giving developers immediate, in-context feedback without needing to leave their PR review workflow to check the SonarQube UI separately.

---

### 8.3 Configuring the DevOps Platform Integration

```
Administration → DevOps Platform Integrations → Configure (GitHub/GitLab/Bitbucket/Azure DevOps)
```

Requires a Personal Access Token (or GitHub App installation) with appropriate repository permissions for SonarQube to post status checks and PR comments back to the source control platform.

---

## MODULE 9: CI/CD Integration

### 9.1 Jenkins Integration (SonarQube Scanner Plugin)

```groovy
pipeline {
  stages {
    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('MySonarQubeServer') {
          sh 'mvn clean verify sonar:sonar'
        }
      }
    }
    stage('Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
  }
}
```

**Critical detail:** `waitForQualityGate` is required specifically because analysis is asynchronous (see Module 4.4) — without it, the pipeline would proceed immediately after the scanner uploads the report, before the Compute Engine has actually processed it and evaluated the Quality Gate, making a "gate" step meaningless without this explicit wait.

---

### 9.2 GitHub Actions Integration

```yaml
- name: SonarQube Scan
  uses: sonarsource/sonarqube-scan-action@master
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

- name: SonarQube Quality Gate check
  uses: sonarsource/sonarqube-quality-gate-action@master
  timeout-minutes: 5
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

---

### 9.3 GitLab CI Integration

```yaml
sonarqube-check:
  stage: test
  image: sonarsource/sonar-scanner-cli:latest
  script:
    - sonar-scanner
  allow_failure: true
```

For a genuine Quality-Gate-blocking pipeline (not just running analysis), configure a webhook from SonarQube back to GitLab, or use the GitLab-specific SonarQube integration that polls for the Quality Gate result before allowing the merge request to proceed.

---

### 9.4 Webhooks for Quality Gate Notification

```
Administration → Webhooks → Add:
  URL: https://ci.mycompany.com/sonar-webhook
```

SonarQube can POST the Quality Gate result to a configured webhook URL immediately after Compute Engine finishes processing — an alternative (often more efficient) pattern to having the CI pipeline poll the SonarQube API repeatedly waiting for the result.

---

## MODULE 10: Administration: Users, Permissions, Projects

### 10.1 Permission Model

```
Global Permissions (server-wide, e.g., "Administer System")
        │
Project Permissions (per-project, e.g., "Browse", "Execute Analysis", "Administer Issues")
```

Permissions can be assigned to individual users or (recommended for manageability at scale) to Groups, avoiding the need to individually manage dozens of user-level permission grants.

---

### 10.2 Common Project Permissions

| Permission | Grants |
|---|---|
| Browse | View project analysis results |
| See Source Code | View actual source code snippets in issue details |
| Administer Issues | Mark issues as false positive/won't fix, assign issues |
| Execute Analysis | Submit new analysis reports for this project |
| Administer | Full project configuration control |

---

### 10.3 Authentication Options

SonarQube supports local (built-in) authentication, LDAP/Active Directory integration, SAML SSO, and OAuth-based integration with GitHub/GitLab/Bitbucket for identity federation — enterprise deployments typically integrate with an existing corporate identity provider rather than relying on locally-managed SonarQube accounts.

---

### 10.4 Project Analysis Tokens vs User Tokens

Best practice is to use dedicated **Project Analysis Tokens** (scoped to exactly one project, generated specifically for CI/CD use) rather than a personal User Token or (worse) the admin account's credentials — limiting the blast radius if a CI secret is ever leaked, since a project-scoped token can only submit analysis for that one specific project.

---

## MODULE 11: SonarQube vs SonarCloud vs SonarLint

### 11.1 Comparison Overview

| SonarQube | SonarCloud | SonarLint |
|---|---|---|
| Self-hosted (you manage the server) | Fully managed SaaS (SonarSource-hosted) | IDE plugin (VS Code, IntelliJ, etc.) |
| Full control over infra/data residency | No infrastructure to manage | Real-time, local analysis before even committing |
| Requires your own database/server maintenance | Free for public/open-source repos | Can connect to SonarQube/SonarCloud for "Connected Mode" |

---

### 11.2 SonarLint "Connected Mode"

SonarLint can synchronize with your organization's SonarQube/SonarCloud instance's Quality Profile, so a developer sees the **exact same** issues in their IDE in real time (before even committing) as would eventually be flagged by the server-side analysis — dramatically shifting feedback left compared to waiting for a CI pipeline run.

---

### 11.3 When to Choose Which

- **SonarCloud** — teams wanting zero infrastructure management overhead, or projects that are public/open-source.
- **SonarQube (self-hosted)** — organizations with strict data residency/compliance requirements needing code to never leave their own infrastructure, or requiring deep integration with internal identity/network infrastructure.
- **SonarLint** — always recommended as a complement to either, for immediate developer feedback regardless of which server-side option is used.

---

## MODULE 12: Database & Performance Tuning

### 12.1 Database Requirements

PostgreSQL is the only fully supported production database for recent SonarQube versions (older versions supported Oracle, SQL Server, MySQL, but this has narrowed over time) — always check the specific version's compatibility matrix before planning an installation/upgrade.

```properties
sonar.jdbc.url=jdbc:postgresql://db-host:5432/sonarqube
sonar.jdbc.maxActive=60
sonar.jdbc.minIdle=10
```

---

### 12.2 Memory/JVM Tuning

```properties
# conf/sonar.properties
sonar.web.javaOpts=-Xmx1G -Xms512m
sonar.ce.javaOpts=-Xmx2G -Xms512m
sonar.search.javaOpts=-Xmx2G -Xms2G
```

The Compute Engine (`sonar.ce.javaOpts`) often needs the most tuning attention for large organizations, since it's responsible for processing every incoming analysis report — insufficient heap here manifests as slow or stuck background analysis tasks.

---

### 12.3 Compute Engine Queue and Worker Count

```properties
sonar.ce.workerCount=4
```

Controls how many analysis reports the Compute Engine processes in parallel — increasing this (bounded by available CPU/memory) reduces queue backlog for organizations submitting many concurrent analyses across multiple projects/CI pipelines.

---

### 12.4 Elasticsearch Considerations

The embedded Elasticsearch (Search Server) requires adequate `vm.max_map_count` (see Module 2.1) and sufficient dedicated heap — under-provisioning this component commonly manifests as slow issue search/filtering in the UI even when analysis itself completes quickly.

---

## MODULE 13: Plugins & Language Support

### 13.1 Built-in vs Marketplace Plugins

Core language analyzers (Java, JavaScript, Python, etc.) ship built-in; additional capabilities (like specific DevOps platform integrations, additional language support in Community Edition via community plugins) are available through the Marketplace (Administration → Marketplace).

---

### 13.2 Community Plugins (Not Officially Supported by SonarSource)

Some languages/frameworks not natively supported in Community Edition (e.g., certain community-maintained plugins for additional languages) can be added via third-party plugins — but these aren't officially maintained by SonarSource, so compatibility with new SonarQube versions isn't guaranteed and should be tested carefully before upgrades.

---

### 13.3 Custom Rules

Organizations can write custom rules (via a plugin SDK, typically for Java) to enforce organization-specific coding standards not covered by built-in rules — a notable investment of engineering effort, usually only justified for large organizations with very specific, recurring compliance or architectural requirements.

---

## MODULE 14: Best Practices & Enterprise Rollout

### 14.1 Rollout Strategy for a Large Organization

```
1. Pilot with a small number of engaged teams first
2. Start with the default "Sonar way" Quality Profile/Gate (don't over-customize immediately)
3. Enable "Clean as You Code" so legacy debt doesn't block adoption
4. Integrate PR decoration for immediate developer feedback
5. Gradually tighten Quality Gate conditions organization-wide as teams adapt
6. Provide SonarLint to developers for local, pre-commit feedback
7. Expand rollout to remaining teams incrementally, sharing lessons learned
```

---

### 14.2 Avoiding Common Adoption Pitfalls

- Don't apply an overly strict Quality Gate to a legacy codebase on day one — this creates immediate friction and pushback; lean on "Clean as You Code" (new code focus) instead.
- Don't let teams routinely mark real issues as "False Positive" just to pass the gate — establish a review process for false-positive/won't-fix decisions to preserve the signal's integrity.
- Don't skip PR decoration — waiting for a separate CI stage or manual UI check significantly slows developer feedback loops compared to in-context PR comments.
- Don't neglect Compute Engine sizing as adoption scales — a growing number of projects/branches analyzing concurrently can overwhelm an under-provisioned Compute Engine, causing a growing backlog.

---

### 14.3 Governance: Portfolios and Executive Reporting (Enterprise Edition)

Enterprise Edition's Portfolio feature aggregates quality/security metrics across many projects into organization or business-unit-level dashboards — useful for demonstrating compliance posture or engineering quality trends to leadership without manually compiling per-project reports.

---

## Summary Cheat-Sheet: Most-Used SonarQube Commands & Config

```bash
# Start/Stop (standalone)
/opt/sonarqube/bin/linux-x86-64/sonar.sh start
/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
/opt/sonarqube/bin/linux-x86-64/sonar.sh status

# Scanning
sonar-scanner -Dsonar.projectKey=my-app -Dsonar.host.url=$SONAR_URL -Dsonar.login=$SONAR_TOKEN
mvn clean verify sonar:sonar

# Key sonar-project.properties settings
sonar.projectKey=
sonar.sources=
sonar.tests=
sonar.coverage.jacoco.xmlReportPaths=
sonar.branch.name=
sonar.pullrequest.key=

# API examples
curl -u $SONAR_TOKEN: "$SONAR_URL/api/qualitygates/project_status?projectKey=my-app"
curl -u $SONAR_TOKEN: "$SONAR_URL/api/ce/task?id=<task-id>"
```

---
