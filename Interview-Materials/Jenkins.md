# Jenkins & CI/CD — Interview Questions

> **Covers:** Simple → Intermediate → Advanced → Scenario-Based  
> **Format:** Question + Detailed Answer + Key Points to Remember

---

## Table of Contents

1. [Simple / Beginner Questions](#1-simple--beginner-questions)
2. [Intermediate Questions](#2-intermediate-questions)
3. [Advanced Questions](#3-advanced-questions)
4. [Scenario-Based Questions](#4-scenario-based-questions)
5. [CI/CD Concepts Questions](#5-cicd-concepts-questions)
6. [Architecture & Design Questions](#6-architecture--design-questions)
7. [Pipeline & Groovy Questions](#7-pipeline--groovy-questions)
8. [Security Questions](#8-security-questions)
9. [Quick-Fire Q&A](#9-quick-fire-qa)

---

## 1. Simple / Beginner Questions

---

**Q1. What is Jenkins?**

**A:** Jenkins is an open-source automation server written in Java. It is used to automate repetitive tasks in the software development lifecycle, primarily building, testing, and deploying applications. Jenkins enables Continuous Integration and Continuous Deployment (CI/CD), which helps teams deliver software faster and more reliably.

**Key points:**
- Open-source, written in Java
- Automates CI/CD pipelines
- 1,800+ plugins for extensibility
- Self-hosted (you manage your own server)

---

**Q2. What is Continuous Integration (CI)?**

**A:** Continuous Integration is the practice of developers merging their code changes into a shared repository frequently (multiple times per day). Each merge triggers an automated build and test sequence, so integration problems are detected early. The goal is to always have a working, tested codebase.

**Key points:**
- Frequent code integration
- Automated build + test on every commit
- Catch bugs early
- Reduces "integration hell"

---

**Q3. What is the difference between Continuous Delivery and Continuous Deployment?**

**A:**

| | Continuous Delivery | Continuous Deployment |
|--|--------------------|-----------------------|
| **Definition** | Code is automatically tested and made *ready* to deploy | Code is automatically deployed to production with no human intervention |
| **Manual step** | Yes — human approves the production release | No — fully automated end-to-end |
| **Risk** | Lower (human gate) | Higher (requires excellent test coverage) |
| **Use case** | Most companies | High-confidence teams (Netflix, Amazon) |

---

**Q4. What is a Jenkins Job / Project?**

**A:** A Jenkins Job (also called a Project) is the basic unit of work in Jenkins. It defines:
- What source code to check out
- How to build and test it
- What to do with results (notifications, archive artifacts, deploy)

**Types of jobs:** Freestyle, Pipeline, Multibranch Pipeline, Folder, Multi-configuration.

---

**Q5. What is a Jenkinsfile?**

**A:** A Jenkinsfile is a text file that contains the definition of a Jenkins Pipeline. It is written in Groovy-based DSL and is stored in the root of the source code repository. This approach is called "Pipeline as Code."

**Benefits:**
- Version controlled alongside application code
- Reviewed via pull requests
- Reusable across teams
- Audit trail for pipeline changes

---

**Q6. What are Jenkins Agents (Slaves)?**

**A:** Agents (previously called Slaves or Nodes) are machines that execute the actual build jobs assigned by the Jenkins Master. The master schedules and orchestrates, while agents do the heavy lifting. An agent can run on any OS and can have specific tools installed (Java, Node.js, Docker, etc.).

---

**Q7. What are the two types of Jenkins Pipeline syntax?**

**A:**

**Declarative Pipeline** — Structured, easier syntax with predefined directives (`pipeline`, `agent`, `stages`, `steps`). Recommended for most use cases.

**Scripted Pipeline** — Full Groovy scripting wrapped in `node {}` blocks. More flexible but more complex. Used for advanced use cases.

---

**Q8. What is a Jenkins Plugin?**

**A:** Plugins are extensions that add functionality to Jenkins. Jenkins has 1,800+ plugins available through its plugin marketplace. Examples: Git plugin (source control), Slack Notification (alerts), Docker plugin (containerized builds), SonarQube scanner (code quality).

**Install:** Dashboard → Manage Jenkins → Plugins → Available Plugins.

---

**Q9. What port does Jenkins run on by default?**

**A:** Jenkins runs on port **8080** by default for the web UI. Agents communicate with the master on port **50000** (JNLP).

---

**Q10. What is the difference between `sh` and `bat` steps in a Jenkinsfile?**

**A:**
- `sh 'command'` — runs a shell command on **Linux/macOS** agents
- `bat 'command'` — runs a batch command on **Windows** agents

```groovy
steps {
    sh 'mvn clean install'   // Linux
    bat 'mvn clean install'  // Windows
}
```

---

## 2. Intermediate Questions

---

**Q11. Explain the Jenkins Master-Agent architecture.**

**A:** Jenkins uses a distributed, Master-Agent architecture:

- **Master (Controller):** The central Jenkins server. Responsibilities include scheduling build jobs, managing agents, storing build results, serving the web UI, and managing configurations and credentials. The master should NOT run heavy builds.

- **Agents (Nodes):** Worker machines that execute build jobs sent by the master. Each agent has one or more executors (slots for concurrent builds). Agents connect to master via SSH or JNLP protocol.

**Why this matters:** It allows Jenkins to scale horizontally, use different OS environments for different builds, and distribute workload across many machines.

---

**Q12. What is a Multibranch Pipeline and why is it useful?**

**A:** A Multibranch Pipeline automatically creates a pipeline for each branch and pull request in a repository. Jenkins scans the repository, discovers all branches that contain a Jenkinsfile, and creates a separate job for each one.

**Benefits:**
- Automatic job creation per branch — no manual setup
- Feature branches get their own isolated builds
- Pull requests can trigger builds and post status checks
- Stale branch jobs are auto-deleted when branches are removed

---

**Q13. What are Build Triggers in Jenkins? Name the common ones.**

**A:** Build Triggers define *when* a Jenkins job should run:

| Trigger | How it works |
|---------|-------------|
| **Webhook (GitHub/GitLab)** | Git server sends HTTP POST to Jenkins when code is pushed. Instant. |
| **Poll SCM** | Jenkins periodically checks the repo for changes (`H/5 * * * *`). Less efficient. |
| **Scheduled (Cron)** | Runs at fixed times, like a cron job (`H 2 * * *` = daily at 2am). |
| **Upstream Job** | Triggers after another job completes successfully. |
| **Manual** | Developer clicks "Build Now" in the UI. |
| **API Trigger** | HTTP POST to Jenkins REST API endpoint. |

---

**Q14. What is the `post` section in a Declarative Pipeline?**

**A:** The `post` section defines actions that run after the pipeline or stage completes. It can be placed at pipeline level (runs after all stages) or inside a stage (runs after that stage).

**Conditions:**
- `always` — runs regardless of result
- `success` — only when build passes
- `failure` — only when build fails
- `unstable` — tests failed but build didn't error
- `changed` — result changed compared to previous build
- `fixed` — was failing, now passing
- `aborted` — build was manually stopped

---

**Q15. How do you pass credentials securely in Jenkins pipelines?**

**A:** Use the Jenkins Credential Store and `withCredentials()` block. Never hardcode passwords in Jenkinsfiles.

```groovy
withCredentials([usernamePassword(
    credentialsId: 'docker-hub-creds',
    usernameVariable: 'DOCKER_USER',
    passwordVariable: 'DOCKER_PASS'
)]) {
    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
}
```

The credentials are automatically masked in console logs as `****`.

---

**Q16. What are Jenkins Shared Libraries?**

**A:** Shared Libraries allow you to extract common pipeline logic into reusable Groovy code stored in a separate Git repository. Multiple Jenkinsfiles can then import and call these shared functions, avoiding code duplication.

**Structure:**
```
vars/        ← Global variables/functions (called from pipeline)
src/         ← Groovy classes
resources/   ← Static files (scripts, configs)
```

**Usage:**
```groovy
@Library('my-shared-lib@main') _
pipeline {
    stages {
        stage('Deploy') {
            steps { deployToKubernetes(namespace: 'prod') }
        }
    }
}
```

---

**Q17. What is Blue Ocean in Jenkins?**

**A:** Blue Ocean is a modern, visually redesigned UI plugin for Jenkins that provides:
- Clear, visual pipeline stage view
- Better visualization of parallel stages
- Easy pipeline creation via UI wizard
- Clean branch and PR views
- Readable console output

It's a plugin that runs alongside the classic Jenkins UI.

---

**Q18. What is `stash` and `unstash` in Jenkins?**

**A:** `stash` saves a set of files to the Jenkins master temporarily. `unstash` retrieves them on a different node. This is used to pass files between stages running on different agents.

```groovy
stage('Build') {
    agent { label 'builder' }
    steps {
        sh 'mvn package'
        stash name: 'app-jar', includes: 'target/*.jar'
    }
}
stage('Deploy') {
    agent { label 'deployer' }
    steps {
        unstash 'app-jar'
        sh 'deploy.sh target/app.jar'
    }
}
```

---

**Q19. What is `archiveArtifacts` and when do you use it?**

**A:** `archiveArtifacts` saves build outputs (JARs, WARs, Docker image tarballs, test reports) to the Jenkins build record so they can be downloaded from the UI or used by other jobs.

```groovy
archiveArtifacts artifacts: 'target/*.jar, reports/**/*.html',
                 fingerprint: true,
                 allowEmptyArchive: false
```

Unlike `stash`, archived artifacts are permanently stored (until build retention cleans them).

---

**Q20. What is `cleanWs()` and why use it?**

**A:** `cleanWs()` is a Pipeline step (from Workspace Cleanup plugin) that deletes the entire workspace after the build. This prevents stale files from previous builds from affecting future builds and saves disk space.

Best practice: Call it in `post { always { cleanWs() } }`.

---

## 3. Advanced Questions

---

**Q21. How do you implement a Jenkins pipeline that deploys to multiple environments with approval gates?**

**A:**

```groovy
stages {
    stage('Deploy to Dev') {
        steps { sh 'deploy.sh dev' }
    }
    stage('Deploy to Staging') {
        steps { sh 'deploy.sh staging' }
    }
    stage('Approval: Prod') {
        steps {
            timeout(time: 24, unit: 'HOURS') {
                input message: 'Deploy to PRODUCTION?',
                      submitter: 'tech-lead,release-manager',
                      ok: 'Approve'
            }
        }
    }
    stage('Deploy to Production') {
        steps { sh 'deploy.sh prod' }
    }
}
```

The `input` step pauses the pipeline and waits for a named approver to click approve in the Jenkins UI. It times out after 24 hours to avoid indefinitely paused pipelines.

---

**Q22. How do you run stages in parallel in Jenkins?**

**A:** Use the `parallel` directive inside a stage:

```groovy
stage('Test Suite') {
    parallel {
        stage('Unit Tests') {
            steps { sh 'mvn test -Punit' }
        }
        stage('Integration Tests') {
            steps { sh 'mvn test -Pintegration' }
        }
        stage('Security Scan') {
            steps { sh 'trivy scan .' }
        }
    }
}
```

All three run simultaneously, and the parent stage waits for all to complete. If any fails, the parent stage fails.

---

**Q23. How do Jenkins Kubernetes agents work?**

**A:** The Kubernetes plugin allows Jenkins to dynamically provision Kubernetes Pods as build agents. When a pipeline requests a Kubernetes agent, Jenkins:

1. Calls the Kubernetes API to create a Pod with the requested containers
2. Pod starts (includes a JNLP container that connects back to Jenkins master)
3. Build runs inside the Pod containers
4. Pod is destroyed after build completes

**Benefits:** Unlimited scalability, fresh environment per build, resource isolation, no idle agents wasting money.

---

**Q24. How do you handle a failing Jenkins job that needs to retry certain steps?**

**A:** Multiple approaches:

```groovy
// Option 1: Retry an entire step block N times
retry(3) {
    sh 'curl http://flaky-service/endpoint'
}

// Option 2: Retry a stage
stage('Flaky Tests') {
    steps {
        retry(2) {
            sh 'mvn test -Pflaky'
        }
    }
}

// Option 3: Scripted retry with delay
script {
    def maxRetries = 3
    def attempts = 0
    while (attempts < maxRetries) {
        try {
            sh 'deploy.sh'
            break
        } catch (Exception e) {
            attempts++
            if (attempts == maxRetries) throw e
            sleep(30)
        }
    }
}
```

---

**Q25. What are Jenkins Declarative `when` conditions and how are they evaluated?**

**A:** `when` conditions determine if a stage should execute. They are evaluated *before* the agent is allocated for the stage (unless `beforeAgent true` is set to false).

```groovy
when {
    // Basic conditions
    branch 'main'                          // branch name matches
    changeRequest()                        // is a pull request
    tag 'v*'                               // is a tag matching pattern
    
    // Compound conditions
    allOf { branch 'main'; not { changeRequest() } }
    anyOf { branch 'main'; branch 'develop' }
    
    // Custom expression
    expression { return env.DEPLOY_ENV == 'prod' }
    
    // File changed
    changeset pattern: '**/*.java'
    
    // Environment variable
    environment name: 'CI', value: 'true'
}
```

---

**Q26. What is Groovy Sandbox in Jenkins and why does it matter?**

**A:** Jenkins runs pipeline Groovy code in a security sandbox that restricts which Java/Groovy methods can be called without explicit admin approval. This prevents malicious pipelines from executing dangerous system operations.

**How it works:**
- Safe methods are on an allowlist (auto-allowed)
- Unsafe methods are blocked and require admin approval via "In-process Script Approval"
- Declarative pipelines are generally safer; scripted pipelines have more sandbox restrictions

**In practice:** If a build fails with "Scripts not permitted to use method...", an admin must approve the specific method in **Manage Jenkins → In-process Script Approval**.

---

**Q27. What is the difference between `node` and `agent` in Jenkins pipelines?**

**A:**

| | `node` | `agent` |
|--|--------|---------|
| **Used in** | Scripted Pipeline | Declarative Pipeline |
| **Syntax** | `node('label') { ... }` | `agent { label 'label' }` |
| **Scope** | Wraps any code block | Defined at pipeline or stage level |
| **Groovy** | Full Groovy available | Restricted Declarative syntax |

`agent` is the Declarative equivalent of `node`. Both allocate an executor on a matching node and create a workspace.

---

**Q28. How do you integrate SonarQube with Jenkins?**

**A:**

1. Install **SonarQube Scanner** plugin in Jenkins
2. Configure SonarQube server in **Manage Jenkins → Configure System → SonarQube**
3. Add SonarQube token as Jenkins credential
4. In pipeline:

```groovy
stage('Code Quality') {
    steps {
        withSonarQubeEnv('SonarQube-Server') {
            sh 'mvn sonar:sonar -Dsonar.projectKey=myapp'
        }
    }
}
stage('Quality Gate') {
    steps {
        timeout(time: 10, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

The `waitForQualityGate` step pauses the pipeline until SonarQube webhook reports pass/fail of quality gate rules.

---

## 4. Scenario-Based Questions

---

**Q29. Scenario: Your build is passing but tests in Jenkins are failing intermittently. How do you debug?**

**A:**

**Step 1 — Gather information:**
- Check if failures are in the same tests each time, or random
- Compare environment: local vs Jenkins (Java version, OS, env vars)
- Check if failures correlate with time of day (resource contention?)

**Step 2 — Common causes and fixes:**

| Root Cause | Investigation | Fix |
|------------|--------------|-----|
| Race conditions in tests | Review test logs for timing | Add test isolation, mock time |
| Resource contention | Check CPU/memory on agent during build | Add more executors or agents |
| Hardcoded paths | Compare local vs Jenkins workspace paths | Use relative paths |
| Environment variables | Print `env` in build | Use `withEnv()` to set needed vars |
| Network flakiness | Check external service calls | Mock external services in tests |
| Test ordering dependency | Run tests in different order | Use test isolation |

**Step 3 — Add retry for known-flaky tests:**
```groovy
retry(3) { sh 'mvn test' }
```

**Step 4 — Enable test history and track which tests are consistently flaky** using JUnit plugin trend reports.

---

**Q30. Scenario: Your Jenkins pipeline deploys to production accidentally when it should only deploy from the `main` branch. How do you fix this?**

**A:**

**Immediate fix — add `when` condition:**
```groovy
stage('Deploy to Production') {
    when {
        allOf {
            branch 'main'
            not { changeRequest() }  // Not a PR
        }
    }
    steps {
        sh 'deploy.sh prod'
    }
}
```

**Deeper fix — environment-based protection:**
```groovy
stage('Deploy') {
    when {
        expression {
            return env.BRANCH_NAME == 'main' &&
                   currentBuild.result == null // No failures so far
        }
    }
}
```

**Best practice — use Multibranch Pipeline** so each branch has its own job, and production deployment logic is branch-aware from the start.

**Add manual approval gate** for production:
```groovy
stage('Approval') {
    when { branch 'main' }
    steps {
        input message: 'Deploy to production?', submitter: 'ops-team'
    }
}
```

---

**Q31. Scenario: Your Jenkins master runs out of disk space. What steps do you take?**

**A:**

**Immediate actions:**

```bash
# 1. Find large directories
du -sh /var/lib/jenkins/workspace/*
du -sh /var/lib/jenkins/jobs/*/builds

# 2. Clear old workspaces
find /var/lib/jenkins/workspace -maxdepth 1 -mtime +7 -exec rm -rf {} \;

# 3. Archive and delete old build logs
find /var/lib/jenkins/jobs -name "log" -mtime +30 -delete
```

**Preventive fixes:**

```groovy
// In each pipeline — add to options
options {
    buildDiscarder(logRotator(
        numToKeepStr: '20',        // Keep 20 builds
        artifactNumToKeepStr: '5'  // Keep 5 artifact builds
    ))
}

// Always clean workspace
post { always { cleanWs() } }
```

**Long-term:**
- Use external artifact storage (Nexus, Artifactory, S3)
- Move build agents to more storage
- Use Docker agents (ephemeral, no workspace accumulation)
- Set up monitoring/alerting on disk usage

---

**Q32. Scenario: You need to implement a rollback mechanism in your Jenkins CD pipeline. How would you design it?**

**A:**

**Strategy 1 — Tag-based rollback:**
```groovy
pipeline {
    parameters {
        string(name: 'DEPLOY_VERSION', defaultValue: 'latest')
        booleanParam(name: 'ROLLBACK', defaultValue: false)
    }

    stages {
        stage('Deploy') {
            steps {
                script {
                    def version = params.ROLLBACK ?
                        sh(returnStdout: true, script: 'get-previous-version.sh').trim() :
                        params.DEPLOY_VERSION
                    sh "helm upgrade myapp ./chart --set image.tag=${version}"
                }
            }
        }
    }
}
```

**Strategy 2 — Keep previous release:**
```groovy
stage('Deploy with Rollback') {
    steps {
        script {
            // Save current version before deploying
            def currentVersion = sh(
                returnStdout: true,
                script: 'kubectl get deployment myapp -o jsonpath="{.spec.template.spec.containers[0].image}"'
            ).trim()
            
            try {
                sh "kubectl set image deployment/myapp app=${NEW_IMAGE}"
                sh "kubectl rollout status deployment/myapp --timeout=5m"
            } catch (e) {
                echo "Deploy failed. Rolling back to ${currentVersion}"
                sh "kubectl set image deployment/myapp app=${currentVersion}"
                throw e
            }
        }
    }
}
```

---

**Q33. Scenario: A developer commits a secret (API key) to the repository. It ends up in a Jenkins build log. What do you do?**

**A:**

**Immediate response:**
1. Revoke/rotate the compromised credential immediately
2. Remove the secret from Git history using `git filter-branch` or BFG Repo Cleaner
3. Force-push cleaned history (coordinate with team)
4. Clear Jenkins build logs that contain the secret

**In Jenkins — mask secrets:**
```groovy
// Use credentials binding — auto-masks in logs
withCredentials([string(credentialsId: 'api-key', variable: 'API_KEY')]) {
    sh 'curl -H "Authorization: ${API_KEY}" https://api.example.com'
    // Logs show: curl -H "Authorization: ****"
}
```

**Preventive measures:**
- Pre-commit hooks (git-secrets, detect-secrets, truffleHog)
- Repository scanning CI step
- Developer education
- Secret management system (HashiCorp Vault, AWS Secrets Manager)
- Jenkins Credential Store — never put secrets in Jenkinsfile

---

**Q34. Scenario: You have 50 microservices. How do you manage their Jenkins pipelines efficiently?**

**A:**

**Strategy 1 — Shared Library (single Jenkinsfile template):**

```groovy
// Each microservice Jenkinsfile (3 lines!)
@Library('company-pipelines@main') _
microservicePipeline(
    serviceName: 'order-service',
    dockerRepo: 'company/order-service',
    deployNamespace: 'microservices'
)
```

The actual logic lives in `vars/microservicePipeline.groovy` in the shared library.

**Strategy 2 — Folder organization:**
```
Jenkins Jobs:
├── microservices/
│   ├── order-service/      (Multibranch Pipeline)
│   ├── payment-service/    (Multibranch Pipeline)
│   ├── inventory-service/  (Multibranch Pipeline)
│   └── ...
├── infrastructure/
│   ├── terraform-apply
│   └── k8s-upgrade
└── release/
    └── release-coordinator
```

**Strategy 3 — Job DSL / Configuration as Code:**
Generate all 50 jobs programmatically from a YAML config using Jenkins Job DSL plugin or JCasC (Jenkins Configuration as Code).

---

**Q35. Scenario: Your pipeline takes 45 minutes. How do you speed it up?**

**A:**

**Step 1 — Profile:** Check stage view to find the slowest stages.

**Step 2 — Parallelise independent stages:**
```groovy
stage('Validate') {
    parallel {
        stage('Unit Tests')      { steps { sh 'mvn test -Punit' } }
        stage('Lint')            { steps { sh 'npm run lint' } }
        stage('Security Scan')   { steps { sh 'trivy scan' } }
        stage('Dependency Check'){ steps { sh 'mvn dependency:analyze' } }
    }
}
```

**Step 3 — Cache dependencies:**
```groovy
// Mount Maven cache so dependencies aren't re-downloaded
agent {
    docker {
        image 'maven:3.9-jdk-17'
        args  '-v /root/.m2:/root/.m2'  // Cache mount
    }
}
```

**Step 4 — Fail fast:**
Run fast unit tests before slow integration tests. Abort early.

**Step 5 — Incremental builds:**
Only build changed modules using change detection.

**Step 6 — More agents:**
Add more parallel executor capacity so jobs don't wait in queue.

---

## 5. CI/CD Concepts Questions

---

**Q36. What is a pipeline in Jenkins and why is "Pipeline as Code" important?**

**A:** A pipeline is a series of automated steps that take code from commit to deployment. "Pipeline as Code" means the pipeline definition is stored in a file (`Jenkinsfile`) in the same repository as the application code.

**Why it's important:**
- **Version control** — pipeline changes are tracked, reviewable, and reversible
- **Peer review** — pipeline changes go through the same PR process as code
- **Consistency** — same pipeline runs in all environments
- **Disaster recovery** — if Jenkins is lost, pipelines can be restored from Git
- **Team ownership** — developers own and maintain their own pipelines

---

**Q37. What is the difference between Build, Release, and Deploy?**

**A:**

| Term | Definition | Example |
|------|-----------|---------|
| **Build** | Compile source code into runnable artifact | `mvn package` → `app.jar` |
| **Release** | Package artifact with config for a specific environment | `app-v1.2.3-prod.jar` |
| **Deploy** | Install the release into an environment so it runs | Copy JAR to server, restart service |

Jenkins automates all three steps in a CI/CD pipeline.

---

**Q38. What is the role of artifact repositories like Nexus or Artifactory in a CI/CD pipeline?**

**A:** Artifact repositories are centralized stores for build outputs:

```
Flow:
Developer → Jenkins Build → Artifact Repository → Deploy to Env

Jenkins uploads:                Teams/tools download:
app-1.2.3.jar                  Deployment scripts
docker-image:1.2.3             Kubernetes Helm charts
npm-package@1.2.3              Other microservices (as dependency)
```

**Benefits:**
- Single source of truth for artifacts
- Artifacts are immutable — same binary tested in staging gets deployed to prod
- Dependency management (proxy public repos like Maven Central, npm)
- Retention policies, access control

---

## 6. Architecture & Design Questions

---

**Q39. How would you design a Jenkins setup for a team of 200 developers?**

**A:**

```
High Availability Setup:

Load Balancer (Nginx/HAProxy)
       │
  ┌────┴────┐
  │  Jenkins │   ← Active master (primary)
  │  Master  │
  └────┬────┘
       │ rsync / shared storage
  ┌────┴────┐
  │  Jenkins │   ← Standby master (for HA)
  │  Backup  │
  └─────────┘

Agent Pool:
  - 10-20 permanent Linux agents (Java, Maven, Gradle)
  - 5 Windows agents (for .NET, Windows tests)
  - Kubernetes cluster for dynamic Docker agents
  - Cloud agents (AWS EC2) for burst capacity
```

**Recommendations:**
- Separate master from builds — no executors on master
- Use Kubernetes for dynamic scaling of agents
- Shared library for common pipeline patterns
- JCasC (Configuration as Code) for reproducible setup
- External artifact store (Nexus/Artifactory)
- Centralized secret management (HashiCorp Vault)
- Monitoring (Prometheus + Grafana for Jenkins metrics)

---

**Q40. What is JCasC (Jenkins Configuration as Code)?**

**A:** JCasC is a plugin that allows Jenkins configuration (global settings, credentials, security realms, cloud agents) to be defined in YAML files and stored in version control — instead of clicking through the UI.

```yaml
# jenkins.yaml
jenkins:
  systemMessage: "Production Jenkins Server"
  numExecutors: 0
  securityRealm:
    ldap:
      configurations:
        - server: ldap://company.com
          rootDN: "dc=company,dc=com"
  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: "admin"
            permissions:
              - "Overall/Administer"

credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              id: "docker-hub"
              username: "myuser"
              password: "${DOCKER_HUB_PASSWORD}"  # from env var
```

---

## 7. Pipeline & Groovy Questions

---

**Q41. How do you capture the output of a shell command in a pipeline variable?**

**A:**

```groovy
script {
    // Capture stdout
    def gitCommit = sh(
        script: 'git rev-parse --short HEAD',
        returnStdout: true
    ).trim()  // .trim() removes trailing newline
    
    echo "Commit: ${gitCommit}"
    env.GIT_SHORT = gitCommit  // Make available globally
    
    // Capture exit code
    def exitCode = sh(
        script: 'test -f config.yaml',
        returnStatus: true
    )
    if (exitCode != 0) {
        error 'config.yaml not found!'
    }
}
```

---

**Q42. How do you use `currentBuild` in a Jenkins pipeline?**

**A:** `currentBuild` is a global variable with properties and methods about the running build:

```groovy
// Properties
currentBuild.number          // Build number (int)
currentBuild.result          // 'SUCCESS', 'FAILURE', 'UNSTABLE', null (in progress)
currentBuild.currentResult   // Same but includes in-progress state
currentBuild.displayName     // Shown in UI (can be overridden)
currentBuild.description     // Short description (can be set)
currentBuild.duration        // Duration in milliseconds
currentBuild.previousBuild   // Reference to previous build object
currentBuild.changeSets      // List of SCM changes

// Set a custom display name
currentBuild.displayName = "#${env.BUILD_NUMBER} - v${params.VERSION}"
currentBuild.description = "Deployed to ${params.ENVIRONMENT}"
```

---

**Q43. What is the `script` block in a Declarative Pipeline and when is it needed?**

**A:** The `script` block allows you to run Scripted Pipeline (Groovy) code inside a Declarative Pipeline. It's needed when you require:

- Variables and control flow (if/else, loops)
- Dynamic behaviour based on runtime values
- Method calls not available as Declarative steps

```groovy
steps {
    script {
        def services = ['user-service', 'order-service', 'payment-service']
        for (service in services) {
            sh "docker build -t myapp/${service}:latest ./${service}"
        }

        if (env.BRANCH_NAME == 'main') {
            sh 'helm upgrade --install myapp ./chart --namespace prod'
        }
    }
}
```

---

## 8. Security Questions

---

**Q44. How do you implement Role-Based Access Control (RBAC) in Jenkins?**

**A:**

1. Install **Role-Based Authorization Strategy** plugin
2. Go to **Manage Jenkins → Security → Authorization → Role-Based Strategy**
3. Define roles at **Manage Jenkins → Manage and Assign Roles**

**Example role setup:**

| Role | Permissions |
|------|------------|
| `admin` | All permissions |
| `developer` | Build, Cancel, Read jobs; View workspace |
| `viewer` | Read-only access |
| `deployer` | Can run deploy jobs only |

4. Assign users/groups to roles globally or per-project (folder-based).

---

**Q45. What is CSRF protection in Jenkins?**

**A:** CSRF (Cross-Site Request Forgery) protection prevents malicious websites from triggering Jenkins actions through a victim's browser session. Jenkins generates a unique **crumb** (token) that must be included in every state-changing request (form submissions, API calls).

**In API calls, include the crumb:**
```bash
# Get crumb
CRUMB=$(curl -s -u user:token \
  http://jenkins:8080/crumbIssuer/api/json \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['crumbRequestField']+':'+d['crumb'])")

# Use crumb in POST request
curl -X POST -H "$CRUMB" http://user:token@jenkins:8080/job/myjob/build
```

CSRF protection is enabled by default in Jenkins 2.x.

---

## 9. Quick-Fire Q&A

```
Q: What language is Jenkinsfile written in?
A: Groovy (a JVM scripting language, Java superset)

Q: What is JENKINS_HOME?
A: Default: /var/lib/jenkins — contains all configs, jobs, plugins, credentials

Q: What is a Jenkins Executor?
A: A slot on a node that can run one build at a time

Q: What plugin gives Jenkins the modern pipeline UI?
A: Blue Ocean

Q: What does `input` step do?
A: Pauses the pipeline and waits for human approval

Q: What is the default Jenkins admin password file location?
A: /var/lib/jenkins/secrets/initialAdminPassword

Q: What is the Pipeline plugin also known as?
A: Workflow plugin

Q: How do you skip tests in Maven within Jenkins?
A: sh 'mvn package -DskipTests'

Q: What Cron expression runs a job every 15 minutes?
A: H/15 * * * *

Q: What is `H` in Jenkins cron expressions?
A: Hash — Jenkins picks a random minute within the range to spread load

Q: What plugin is needed to archive test results?
A: JUnit Plugin

Q: Can you run Jenkins without Java?
A: No. Jenkins requires Java 11 or 17 to run.

Q: What command restarts Jenkins?
A: http://jenkins:8080/restart or systemctl restart jenkins

Q: What is a Webhook?
A: An HTTP callback — Git sends a POST to Jenkins when code is pushed

Q: What is a Build Artifact?
A: Output file from a build (JAR, WAR, Docker image, ZIP) stored in Jenkins

Q: What does `returnStdout: true` do in `sh()`?
A: Returns the stdout of the command as a String instead of printing it

Q: What is `withEnv()`?
A: Sets environment variables for the duration of a block

Q: What is the difference between Global and System credential scope?
A: Global = available to all jobs; System = Jenkins internals only

Q: What is a Jenkins Seed Job?
A: A job that uses Job DSL to programmatically create/update other jobs

Q: How do you make a stage always run, even if previous stages failed?
A: Use post { always { ... } } or check currentBuild.result in a try/finally block
```

---

## Interview Tips

```
Top things interviewers test:
──────────────────────────────────────────────────────────────────

CONCEPT CLARITY
  Can you explain CI vs CD vs CD clearly?
  Do you know the difference between Declarative and Scripted?
  Can you draw the Master-Agent architecture?

PRACTICAL KNOWLEDGE
  Have you written a Jenkinsfile? Can you walk through it?
  How do you handle secrets? (Answer: Credential Store + withCredentials)
  What triggers have you used?

PROBLEM SOLVING
  "Build works locally but fails in Jenkins" — debug approach
  "Pipeline is slow" — optimization strategies
  "Deployment went to wrong env" — prevention and fix

BEST PRACTICES
  Pipeline as Code (Jenkinsfile in Git)
  Shared Libraries for reusability
  Clean workspace, timeouts, build retention
  No hardcoded secrets
  Fail fast (test before deploy)

ARCHITECTURE
  When would you use multiple agents?
  How would you design for 100+ developers?
  How do you achieve high availability?
```

---

