# <img src="https://cdn.simpleicons.org/jenkins" width="40" /> Jenkins & CI/CD — Complete Guide

---

## Table of Contents

1. [CI/CD Fundamentals](#1-cicd-fundamentals)
2. [What is Jenkins?](#2-what-is-jenkins)
3. [Jenkins Architecture](#3-jenkins-architecture)
4. [Installation & Setup](#4-installation--setup)
5. [Core Concepts](#5-core-concepts)
6. [Jenkinsfile & Pipeline Syntax](#6-jenkinsfile--pipeline-syntax)
7. [Declarative Pipeline — Deep Dive](#7-declarative-pipeline--deep-dive)
8. [Scripted Pipeline — Deep Dive](#8-scripted-pipeline--deep-dive)
9. [Plugins Ecosystem](#9-plugins-ecosystem)
10. [Distributed Builds — Master-Agent](#10-distributed-builds--master-agent)
11. [Real-World Workflows](#11-real-world-workflows)
12. [Security & Best Practices](#12-security--best-practices)
13. [Jenkins vs Competitors](#13-jenkins-vs-competitors)
14. [Troubleshooting & Tips](#14-troubleshooting--tips)

---

## 1. CI/CD Fundamentals

### What is CI/CD?

**CI/CD** stands for **Continuous Integration / Continuous Delivery (or Deployment)** — a set of practices that automate the software delivery pipeline from code commit to production.

```
Developer → Commit → Build → Test → Deploy → Monitor
    ↑                                              |
    └──────────── Feedback Loop ───────────────────┘
```

### Continuous Integration (CI)

Developers merge code changes into a shared branch **frequently** (multiple times per day). Each merge triggers an automated build and test sequence.

**Problems CI solves:**
- "Integration hell" — merging weeks of separate work at once
- Bugs discovered late in the cycle (expensive to fix)
- "Works on my machine" syndrome
- Lack of visibility into code health

**CI Pipeline Flow:**

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Code    │───▶│  Source  │───▶│  Build   │───▶│  Test    │───▶│  Report  │
│  Commit  │    │  Fetch   │    │  Compile │    │  Unit    │    │  Results │
└──────────┘    └──────────┘    └──────────┘    │  Integr. │    └──────────┘
                                                 │  E2E     │
                                                 └──────────┘
```

### Continuous Delivery (CD)

Every code change that passes CI is **automatically deployed to a staging environment** and is *ready* for production release (manual approval gate).

### Continuous Deployment

Goes one step further — every passing change is **automatically deployed to production** with zero human intervention.

```
CI/CD Spectrum:
─────────────────────────────────────────────────────────
Manual   →   Continuous   →   Continuous   →   Continuous
Release       Integration      Delivery         Deployment
                              (staging auto,    (prod auto)
                               prod manual)
```

### The DevOps Pipeline (Big Picture)

```
┌─────────────────────────────────────────────────────────────────────┐
│                         DevOps Lifecycle                            │
│                                                                     │
│   PLAN → CODE → BUILD → TEST → RELEASE → DEPLOY → OPERATE → MONITOR│
│     ↑                                                        |      │
│     └──────────────── Feedback ────────────────────────────-┘      │
│                                                                     │
│   Tools:                                                            │
│   PLAN:    Jira, Trello        RELEASE:  Jenkins, ArgoCD            │
│   CODE:    Git, GitHub         DEPLOY:   Kubernetes, Ansible        │
│   BUILD:   Maven, Gradle       OPERATE:  Docker, Terraform          │
│   TEST:    JUnit, Selenium     MONITOR:  Prometheus, Grafana        │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. What is Jenkins?

Jenkins is an **open-source automation server** written in Java, used to automate the parts of software development related to building, testing, and deploying — facilitating CI/CD.

### Core Definition

| Property | Value |
|----------|-------|
| **Type** | Automation Server |
| **Language** | Java |
| **First Release** | 2004 (as Hudson), renamed Jenkins 2011 |
| **Current** | Jenkins 2.x LTS |
| **Plugins** | 1,800+ |

### The Assembly Line Analogy

```
Raw Material (Code) 
      │
      ▼
┌─────────────────────────────────────────────────────┐
│                  Jenkins Factory                     │
│                                                     │
│  [Fetch Code] → [Compile] → [Test] → [Package]     │
│                                                     │
│  Quality checks at every stage                     │
│  Automatic rejection if defective                  │
└─────────────────────────────────────────────────────┘
      │
      ▼
Finished Product (Deployed App)
```

### Why Jenkins Dominates

```
Market Position (CI/CD Tools):

Jenkins      ████████████████████████████████  70% enterprises
GitHub Act.  █████████████████                 35%
GitLab CI    ████████████                      25%
Azure DevOps █████████                         20%
CircleCI     ██████                            13%
TeamCity     █████                             10%
```

### Key Strengths

- **Free & Open Source** — No licensing cost, only infra
- **Plugin Ecosystem** — 1,800+ plugins, integrate with anything
- **Platform Independent** — Windows, Linux, macOS, Docker
- **Mature & Stable** — 20+ years of battle-tested production use
- **Pipeline as Code** — Jenkinsfile stored in Git
- **Distributed** — Scale horizontally with agents
- **Community** — Largest CI/CD community worldwide

---

## 3. Jenkins Architecture

### High-Level Architecture

```
                    ┌──────────────────────────────────┐
                    │      DEVELOPER WORKSTATIONS       │
                    │   (Git commit triggers webhook)   │
                    └──────────────┬───────────────────┘
                                   │ webhook / poll
                                   ▼
┌──────────────────────────────────────────────────────────────────┐
│                    JENKINS MASTER (CONTROLLER)                   │
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │  Web UI     │  │  Job Sched- │  │   Plugin Manager        │  │
│  │  Dashboard  │  │  uler       │  │   Config Store          │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │  Build      │  │  Result     │  │   Credential            │  │
│  │  Queue      │  │  Storage    │  │   Vault                 │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
└────────────────────────┬─────────────────────────────────────────┘
                         │ TCP / SSH / JNLP
            ┌────────────┼────────────┐
            ▼            ▼            ▼
    ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
    │   AGENT 1    │ │   AGENT 2    │ │   AGENT 3    │
    │              │ │              │ │              │
    │  Linux/Ubuntu│ │  Windows     │ │  macOS       │
    │  Java builds │ │  .NET builds │ │  iOS builds  │
    │              │ │              │ │              │
    └──────────────┘ └──────────────┘ └──────────────┘
```

### Master Responsibilities

| Responsibility | Description |
|---------------|-------------|
| Schedule jobs | Decide when and where builds run |
| Manage agents | Monitor health, take online/offline |
| Store results | Build history, logs, artifacts |
| Serve Web UI | Dashboard accessible on port 8080 |
| Security | Authentication, authorization, credentials |
| Plugin management | Install, update, remove plugins |

> **Rule:** Master should NOT run heavy builds — delegate to agents!

### Agent Types

```
Agent Types:
─────────────────────────────────────────────────────────────────

PERMANENT AGENTS          CLOUD AGENTS              DOCKER AGENTS
─────────────────         ─────────────             ─────────────
Always running            Provisioned on demand     Container-based
Static IP/hostname        AWS EC2, Azure VMs        Ephemeral
Manual setup              Auto scales               Clean env each build
Good for: heavy/long      Good for: variable load   Good for: isolation
builds                    
```

### Communication Protocols

```
Master ←→ Agent Communication:

1. SSH (most common for Linux agents)
   Master initiates connection to agent
   
2. JNLP / TCP (for Windows / firewalled agents)
   Agent initiates connection to master
   
3. Inbound agent (agent.jar)
   java -jar agent.jar -jnlpUrl http://master:8080/...
```

### Build Executor

Each node (master or agent) has **executors** — slots that can run one build at a time.

```
Agent with 3 executors:
┌────────────────────────────────┐
│  Executor 1: [BUILD job-A ███] │
│  Executor 2: [BUILD job-B ██ ] │
│  Executor 3: [IDLE           ] │
└────────────────────────────────┘
```

---

## 4. Installation & Setup

### Prerequisites

```
System Requirements:
─────────────────────────────────
RAM:  Minimum 256MB, Recommended 1GB+
Disk: 10GB+
JDK:  Java 11 or Java 17 (LTS)
OS:   Linux (Ubuntu/CentOS), Windows, macOS, Docker
```

### Install on Ubuntu/Debian

```bash
# Step 1: Add Jenkins repository key
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key \
  | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

# Step 2: Add Jenkins repository
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" \
  | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Step 3: Install Java and Jenkins
sudo apt-get update
sudo apt-get install -y openjdk-17-jre jenkins

# Step 4: Start and enable Jenkins service
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Step 5: Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# Step 6: Access Jenkins at http://localhost:8080
```

### Install via Docker (Recommended for Dev)

```bash
# Run Jenkins in Docker
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts-jdk17

# Get admin password
docker exec jenkins \
  cat /var/jenkins_home/secrets/initialAdminPassword
```

### Initial Setup Wizard

```
http://localhost:8080
       │
       ▼
[Enter Admin Password]
       │
       ▼
[Install Suggested Plugins] ← Choose this for beginners
       │
       ▼
[Create Admin User]
       │
       ▼
[Configure Instance URL]
       │
       ▼
[Jenkins Ready!]
```

### Jenkins Directory Structure

```
/var/lib/jenkins/                     # JENKINS_HOME
├── config.xml                        # Global configuration
├── credentials.xml                   # Encrypted credentials
├── secrets/
│   └── initialAdminPassword          # First login password
├── plugins/                          # Installed plugins (.jpi)
│   ├── git/
│   ├── pipeline/
│   └── ...
├── jobs/                             # All job configurations
│   ├── my-pipeline/
│   │   ├── config.xml
│   │   └── builds/
│   │       ├── 1/
│   │       ├── 2/
│   │       └── ...
│   └── another-job/
├── workspace/                        # Build workspaces (per job)
│   └── my-pipeline/
│       └── (cloned source code)
└── logs/                             # Jenkins system logs
```

---

## 5. Core Concepts

### Jobs / Projects

A **Job** (also called a Project) is the basic unit of work in Jenkins. It defines what to build, how to build it, and what to do with the results.

```
Job Types:
──────────────────────────────────────────────────────────────────

FREESTYLE           PIPELINE            MULTIBRANCH        FOLDER
──────────          ────────            ───────────        ──────
GUI-configured      Code-defined        Auto-discovers     Organizes
Simple builds       Jenkinsfile         branches/PRs       related jobs
Click-and-go        Version controlled  in Git repo        
Legacy approach     Recommended         Modern approach    
```

### Freestyle Job — Quick Setup

```
New Item → Freestyle Project

Source Code Management:
  Git URL: https://github.com/user/repo.git
  Branch: */main

Build Triggers:
  ☑ GitHub hook trigger for GITScm polling

Build Steps:
  Execute shell:
    mvn clean install
    mvn test

Post-build Actions:
  Publish JUnit test result report: target/surefire-reports/*.xml
  Email notification on failure
```

### Build Triggers

```
Trigger Types:
────────────────────────────────────────────────────────────────

WEBHOOK                    POLL SCM               SCHEDULED
───────                    ────────               ─────────
Git push → HTTP POST       Jenkins checks Git     Cron expression
to Jenkins URL             every N minutes        
Instant, recommended       Less efficient         H 2 * * *
                                                  (daily 2am)

MANUAL                     UPSTREAM JOB           API TRIGGER
──────                     ────────────           ───────────
Click "Build Now"          After another          HTTP POST to
in UI                      job completes          /job/NAME/build
```

### Build Environments

```
Workspace per Job:
┌──────────────────────────────────────────────────┐
│  /var/lib/jenkins/workspace/my-pipeline/         │
│                                                  │
│  ├── src/                 ← cloned source code   │
│  ├── pom.xml                                     │
│  ├── target/              ← compiled output      │
│  └── test-results/        ← test reports         │
└──────────────────────────────────────────────────┘
```

### Artifacts

Build outputs saved and accessible after the build:

```bash
# In pipeline - archive artifacts
archiveArtifacts artifacts: 'target/*.jar', fingerprint: true

# In pipeline - stash/unstash between stages
stash name: 'compiled-app', includes: 'target/**'
unstash 'compiled-app'
```

### Environment Variables

```groovy
// Built-in Jenkins variables:
BUILD_NUMBER        // Current build number (1, 2, 3...)
BUILD_ID            // Build ID (same as BUILD_NUMBER)
BUILD_URL           // Full URL to this build
JOB_NAME            // Name of the job
JOB_BASE_NAME       // Job name without folder path
WORKSPACE           // Absolute path to workspace
NODE_NAME           // Name of agent running this build
GIT_COMMIT          // Git commit SHA
GIT_BRANCH          // Branch being built
JENKINS_URL         // Root Jenkins URL

// Usage in pipeline:
echo "Building ${env.JOB_NAME} #${env.BUILD_NUMBER}"
echo "Commit: ${env.GIT_COMMIT}"
```

---

## 6. Jenkinsfile & Pipeline Syntax

### What is a Jenkinsfile?

A **Jenkinsfile** is a text file (Groovy-based DSL) that defines the entire CI/CD pipeline as code, stored alongside source code in version control.

```
Repository Structure:
├── src/
├── tests/
├── pom.xml
└── Jenkinsfile          ← Pipeline definition lives here
```

### Two Pipeline Syntaxes

```
DECLARATIVE (Recommended)         SCRIPTED (Advanced)
─────────────────────             ───────────────────
Structured, opinionated           Full Groovy freedom
Easier to learn                   More flexible
Better syntax validation          Older syntax
Has built-in directives           Manual error handling
Preferred for most cases          For complex needs
```

### Declarative Pipeline — Skeleton

```groovy
pipeline {
    agent any                          // Where to run

    environment {                      // Global variables
        APP_NAME = 'myapp'
        VERSION  = '1.0.0'
    }

    options {                          // Pipeline options
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    triggers {                         // Auto-triggers
        pollSCM('H/5 * * * *')
    }

    parameters {                       // Build parameters
        string(name: 'BRANCH', defaultValue: 'main')
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'])
        booleanParam(name: 'RUN_TESTS', defaultValue: true)
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh './deploy.sh'
            }
        }
    }

    post {
        success {
            echo 'Build succeeded!'
            slackSend color: 'good', message: "✅ ${env.JOB_NAME} passed"
        }
        failure {
            echo 'Build failed!'
            mail to: 'team@company.com',
                 subject: "❌ ${env.JOB_NAME} failed",
                 body: "Check: ${env.BUILD_URL}"
        }
        always {
            cleanWs()
        }
    }
}
```

---

## 7. Declarative Pipeline — Deep Dive

### `agent` Directive

```groovy
// Run on any available agent
agent any

// Run on master (not recommended for heavy builds)
agent none  // and specify per-stage

// Run on specific label
agent { label 'linux-build-agent' }

// Run in Docker container
agent {
    docker {
        image 'maven:3.9.0-jdk-17'
        args  '-v /root/.m2:/root/.m2'
    }
}

// Run in Kubernetes pod
agent {
    kubernetes {
        yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.9.0-jdk-17
    command: ['sleep', 'infinity']
"""
    }
}
```

### `stages` and `stage`

```groovy
stages {
    // Sequential stages
    stage('Stage 1') { ... }
    stage('Stage 2') { ... }

    // Parallel stages
    stage('Parallel Tests') {
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

    // Nested stages (Jenkins 2.x+)
    stage('QA') {
        stages {
            stage('Lint')    { steps { sh 'npm run lint' } }
            stage('Unit')    { steps { sh 'npm test' } }
            stage('E2E')     { steps { sh 'cypress run' } }
        }
    }
}
```

### `when` Conditions

```groovy
// Run only on main branch
when { branch 'main' }

// Run only on specific branches
when { anyOf { branch 'main'; branch 'develop' } }

// Run only when parameter is true
when { expression { return params.RUN_TESTS == true } }

// Run only when file changed
when { changeset "**/*.java" }

// Run only in specific environment
when { environment name: 'DEPLOY_ENV', value: 'production' }

// Combine conditions
when {
    allOf {
        branch 'main'
        not { changeRequest() }
        expression { return currentBuild.result == null }
    }
}
```

### `post` Actions

```groovy
post {
    always       { /* Always runs — cleanup, notifications */ }
    success      { /* Only on success */ }
    failure      { /* Only on failure */ }
    unstable     { /* Tests failed but build succeeded */ }
    changed      { /* Status changed from last build */ }
    fixed        { /* Was failing, now passing */ }
    regression   { /* Was passing, now failing */ }
    aborted      { /* Build was manually aborted */ }
    cleanup      { /* Runs last, even if always fails */ }
}
```

### `options` Directive

```groovy
options {
    // Timeout entire pipeline
    timeout(time: 1, unit: 'HOURS')
    
    // Abort if same job is already running
    disableConcurrentBuilds()
    
    // Skip checkout (manual control)
    skipDefaultCheckout()
    
    // Keep only N builds in history
    buildDiscarder(logRotator(
        numToKeepStr: '20',
        artifactNumToKeepStr: '5'
    ))
    
    // Add timestamps to console log
    timestamps()
    
    // Retry failed build N times
    retry(3)
    
    // ANSI colors in console
    ansiColor('xterm')
}
```

### Credentials Handling

```groovy
// Username/Password
withCredentials([usernamePassword(
    credentialsId: 'docker-hub-creds',
    usernameVariable: 'DOCKER_USER',
    passwordVariable: 'DOCKER_PASS'
)]) {
    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
}

// Secret text
withCredentials([string(
    credentialsId: 'api-token',
    variable: 'API_TOKEN'
)]) {
    sh 'curl -H "Authorization: Bearer $API_TOKEN" https://api.example.com'
}

// SSH key
withCredentials([sshUserPrivateKey(
    credentialsId: 'deploy-ssh-key',
    keyFileVariable: 'SSH_KEY'
)]) {
    sh 'ssh -i $SSH_KEY user@server.com "deploy.sh"'
}

// File (e.g., kubeconfig)
withCredentials([file(
    credentialsId: 'kubeconfig',
    variable: 'KUBECONFIG'
)]) {
    sh 'kubectl apply -f deployment.yaml'
}
```

---

## 8. Scripted Pipeline — Deep Dive

### Basic Structure

```groovy
// Scripted pipeline: full Groovy, runs in node blocks
node('linux-agent') {
    try {
        stage('Checkout') {
            checkout scm
        }

        stage('Build') {
            def mvnHome = tool 'Maven-3.9'
            sh "${mvnHome}/bin/mvn clean package"
        }

        stage('Test') {
            sh 'mvn test'
        }

        currentBuild.result = 'SUCCESS'
    } catch (e) {
        currentBuild.result = 'FAILED'
        throw e
    } finally {
        // Always runs
        stage('Cleanup') {
            cleanWs()
        }
    }
}
```

### Dynamic Node Selection

```groovy
// Scripted: run stages on different nodes
stage('Build on Linux') {
    node('linux') {
        checkout scm
        sh 'make build'
        stash name: 'build-output', includes: 'dist/**'
    }
}

stage('Test on Windows') {
    node('windows') {
        unstash 'build-output'
        bat 'run-tests.bat'
    }
}
```

### Advanced Groovy in Scripted Pipeline

```groovy
node {
    // Define functions
    def deployToEnv(String env, String artifact) {
        echo "Deploying ${artifact} to ${env}"
        sh "ansible-playbook -i ${env} deploy.yml -e artifact=${artifact}"
    }

    // Use maps and lists
    def environments = ['dev', 'staging', 'prod']
    def results = [:]

    stage('Build') {
        checkout scm
        sh 'mvn package'
    }

    // Dynamic parallel stages
    stage('Deploy to All') {
        def parallelStages = [:]
        environments.each { env ->
            def envName = env  // Important: capture variable
            parallelStages[env] = {
                node('deploy-agent') {
                    deployToEnv(envName, 'myapp.jar')
                }
            }
        }
        parallel parallelStages
    }
}
```

---

## 9. Plugins Ecosystem

### Must-Have Plugins

```
Category              Plugin                    Purpose
─────────────         ──────────────────────    ─────────────────────────────
Source Control        Git Plugin                Clone Git repositories
                      GitHub Integration        Webhooks, PR status
                      GitLab Plugin             GitLab integration
                      
Build Tools           Maven Integration         Maven build support
                      Gradle Plugin             Gradle build support
                      NodeJS Plugin             Node.js build environments
                      
Pipeline              Pipeline                  Core pipeline support
                      Blue Ocean                Modern pipeline UI
                      Pipeline: Stage View      Stage visualization
                      
Docker                Docker Plugin             Build Docker images
                      Docker Pipeline           Use Docker in pipelines
                      
Kubernetes            Kubernetes Plugin         Dynamic K8s agents
                      
Testing               JUnit Plugin              Publish test results
                      Jacoco Plugin             Code coverage
                      
Code Quality          SonarQube Scanner         Static code analysis
                      Checkstyle Plugin         Style checking
                      
Security              OWASP Dependency Check    Vulnerability scanning
                      Role-based Auth           RBAC for Jenkins
                      Credentials Binding       Secure secret injection
                      
Notifications         Slack Notification        Slack messages
                      Email Extension           Rich email alerts
                      MS Teams Plugin           Microsoft Teams alerts
                      
Deployment            Kubernetes CD             Deploy to Kubernetes
                      AWS Steps                 AWS operations
                      Azure Credentials         Azure integration
                      
Artifact Mgmt         Nexus Artifact Uploader   Upload to Nexus
                      Artifactory Plugin        JFrog Artifactory
```

### Installing Plugins

```
Dashboard → Manage Jenkins → Plugins → Available Plugins
Search → Select → Install without restart (or with restart)
```

```bash
# Install via Jenkins CLI
java -jar jenkins-cli.jar -s http://localhost:8080/ \
  install-plugin git github pipeline-stage-view blueocean
```

---

## 10. Distributed Builds — Master-Agent

### Setting Up a Permanent Agent

```
Dashboard → Manage Jenkins → Nodes → New Node

Node Name:     linux-agent-01
Type:          Permanent Agent

Configuration:
  # of Executors:  4
  Remote root dir: /home/jenkins/agent
  Labels:          linux java-build
  Launch method:   Launch agents via SSH
  
SSH Settings:
  Host: 192.168.1.100
  Credentials: (SSH private key)
  Host Key Verification: Known hosts file
```

### Docker Agent in Pipeline

```groovy
pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            label 'docker-enabled'
            // Mount caches for faster builds
            args '-v /tmp/.npm:/root/.npm'
        }
    }

    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}
```

### Kubernetes Agent — Full Example

```groovy
pipeline {
    agent {
        kubernetes {
            label 'k8s-agent'
            defaultContainer 'maven'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: jenkins-agent
spec:
  containers:
  - name: maven
    image: maven:3.9-jdk-17
    command: ['sleep', '999d']
    resources:
      requests:
        cpu: '500m'
        memory: '512Mi'
  - name: docker
    image: docker:dind
    securityContext:
      privileged: true
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ['sleep', '999d']
"""
        }
    }

    stages {
        stage('Build') {
            container('maven') {
                steps { sh 'mvn package' }
            }
        }
        stage('Docker Build') {
            container('docker') {
                steps { sh 'docker build -t myapp:latest .' }
            }
        }
        stage('Deploy') {
            container('kubectl') {
                steps { sh 'kubectl apply -f k8s/' }
            }
        }
    }
}
```

---

## 11. Real-World Workflows

### Full Java/Spring Boot CI/CD Pipeline

```groovy
pipeline {
    agent { label 'java-build' }

    environment {
        APP_NAME    = 'spring-petclinic'
        DOCKER_REPO = 'mycompany/spring-petclinic'
        SONAR_URL   = 'http://sonarqube:9000'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_SHORT = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                    env.IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_SHORT}"
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile -q'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test -pl unit-tests'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco(
                        execPattern: 'target/jacoco.exec',
                        classPattern: 'target/classes',
                        sourcePattern: 'src/main/java'
                    )
                }
            }
        }

        stage('Code Quality') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh 'mvn org.owasp:dependency-check-maven:check'
            }
            post {
                always {
                    dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        docker build -t ${DOCKER_REPO}:${IMAGE_TAG} .
                        docker build -t ${DOCKER_REPO}:latest .
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_REPO}:${IMAGE_TAG}
                        docker push ${DOCKER_REPO}:latest
                    """
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                sh "helm upgrade --install ${APP_NAME} ./helm \
                    --set image.tag=${IMAGE_TAG} \
                    --namespace dev \
                    --create-namespace"
            }
        }

        stage('Integration Tests') {
            steps {
                sh 'mvn test -pl integration-tests -Dbase.url=http://dev.myapp.com'
            }
        }

        stage('Deploy to Staging') {
            when { branch 'main' }
            steps {
                sh "helm upgrade --install ${APP_NAME} ./helm \
                    --set image.tag=${IMAGE_TAG} \
                    --namespace staging"
            }
        }

        stage('Approval for Production') {
            when { branch 'main' }
            steps {
                timeout(time: 24, unit: 'HOURS') {
                    input message: "Deploy ${IMAGE_TAG} to PRODUCTION?",
                          submitter: 'tech-lead,devops-team',
                          ok: 'Deploy'
                }
            }
        }

        stage('Deploy to Production') {
            when { branch 'main' }
            steps {
                sh "helm upgrade --install ${APP_NAME} ./helm \
                    --set image.tag=${IMAGE_TAG} \
                    --namespace prod"
            }
        }
    }

    post {
        success {
            slackSend(
                channel: '#deployments',
                color: 'good',
                message: "✅ *${APP_NAME}* deployed to production!\nImage: `${IMAGE_TAG}`\nBuild: ${env.BUILD_URL}"
            )
        }
        failure {
            slackSend(
                channel: '#alerts',
                color: 'danger',
                message: "❌ *${APP_NAME}* pipeline failed!\nBranch: `${env.BRANCH_NAME}`\nBuild: ${env.BUILD_URL}"
            )
        }
        always { cleanWs() }
    }
}
```

### Node.js / React Pipeline

```groovy
pipeline {
    agent { docker { image 'node:18-alpine' } }

    environment {
        CI           = 'true'
        REACT_APP_ENV = 'production'
    }

    stages {
        stage('Install')         { steps { sh 'npm ci' } }
        stage('Lint')            { steps { sh 'npm run lint' } }
        stage('Test')            { steps { sh 'npm test -- --coverage --ci' } }
        stage('Build')           { steps { sh 'npm run build' } }
        stage('E2E Tests') {
            steps {
                sh 'npx cypress run --headless'
            }
            post { always { publishHTML([
                reportDir: 'cypress/reports',
                reportFiles: 'index.html',
                reportName: 'Cypress Report'
            ])}}
        }
        stage('Deploy S3') {
            when { branch 'main' }
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials']]) {
                    sh 'aws s3 sync build/ s3://my-react-app --delete'
                    sh 'aws cloudfront create-invalidation --distribution-id ABCDEF --paths "/*"'
                }
            }
        }
    }
}
```

### Multibranch Pipeline Strategy

```
Repository Branches → Auto-discovered by Jenkins

main branch          → Full pipeline: build, test, scan, deploy prod
develop branch       → Build + test + deploy dev
feature/* branches   → Build + test only (fast feedback)
release/* branches   → Full pipeline + manual approval
hotfix/* branches    → Fast track: build, test, deploy staging, manual prod
PR branches          → Build + test + SonarQube + comment status on PR
```

```groovy
// Branch-specific logic in Jenkinsfile
stage('Deploy') {
    when {
        anyOf {
            branch 'main'
            branch 'develop'
            branch pattern: 'release/*', comparator: 'GLOB'
        }
    }
    steps {
        script {
            def targetEnv = env.BRANCH_NAME == 'main' ? 'prod' :
                            env.BRANCH_NAME == 'develop' ? 'dev' : 'staging'
            sh "deploy.sh ${targetEnv}"
        }
    }
}
```

---

## 12. Security & Best Practices

### Security Hardening

```
Security Checklist:
─────────────────────────────────────────────────────────────────

[✓] AUTHENTICATION
    - Integrate with LDAP/Active Directory or OAuth (GitHub, Google)
    - Disable "Allow users to sign up" if using internal auth
    - Enable "Jenkins' own user database" with strong passwords

[✓] AUTHORIZATION
    - Install Role-Based Authorization Strategy plugin
    - Create roles: Admin, Developer, Viewer
    - Principle of least privilege

[✓] CREDENTIALS
    - NEVER hardcode secrets in Jenkinsfile
    - Use Jenkins Credential Store for all secrets
    - Rotate secrets regularly
    - Use withCredentials() block — auto-masks in logs

[✓] NETWORK
    - Run Jenkins behind reverse proxy (Nginx)
    - Enable HTTPS (TLS/SSL)
    - Firewall: only expose port 443 externally
    - Agent communication on 50000 — internal only

[✓] MASTER PROTECTION
    - Disable builds on master node
    - Limit Groovy script execution (Script Security plugin)
    - Enable CSRF protection (enabled by default in 2.x)
    - Keep Jenkins and plugins updated

[✓] AUDIT
    - Enable Audit Trail plugin
    - Log all user actions
    - Monitor failed builds for security issues
```

### Shared Libraries

```
vars/
└── deployToK8s.groovy
src/
└── com/company/jenkins/
    └── Utils.groovy
resources/
└── scripts/
    └── deploy.sh

Jenkinsfile:
────────────
@Library('company-jenkins-lib@main') _

pipeline {
    stages {
        stage('Deploy') {
            steps {
                deployToK8s(
                    namespace: 'production',
                    image: "myapp:${env.BUILD_NUMBER}"
                )
            }
        }
    }
}
```

### Pipeline Best Practices

```
DO:                                   DON'T:
────────────────────────────────      ──────────────────────────────────
✅ Store Jenkinsfile in SCM           ❌ Inline pipeline in Jenkins UI
✅ Use Declarative syntax             ❌ Overly complex Scripted pipelines
✅ Fail fast — test first             ❌ Deploy before tests pass
✅ Use stages for visibility          ❌ One giant stage with all steps
✅ Archive important artifacts        ❌ Rely on workspace persistence
✅ Set timeouts on all stages         ❌ Let builds run forever
✅ Clean workspace after build        ❌ Accumulate files across builds
✅ Use parallel for independent work  ❌ Serialize everything sequentially
✅ Version-pin Docker images          ❌ Use :latest tags in production
✅ Use meaningful stage names         ❌ "Stage 1", "Stage 2" naming
```

---

## 13. Jenkins vs Competitors

| Feature | Jenkins | GitHub Actions | GitLab CI | CircleCI | Azure DevOps |
|---------|---------|----------------|-----------|----------|--------------|
| **Type** | Self-hosted | Cloud/SaaS | Cloud+Self | Cloud/SaaS | Cloud+On-prem |
| **Config File** | `Jenkinsfile` | `.github/workflows/*.yml` | `.gitlab-ci.yml` | `.circleci/config.yml` | `azure-pipelines.yml` |
| **Cost** | Free (infra) | Free+PAYG | Free+$19/user | Free+$15/mo | Free+$6/user |
| **Free Minutes** | Unlimited* | 2,000/mo | 400/mo | 6,000/mo | 1,800/mo |
| **Setup Time** | Hours | Minutes | Minutes | Minutes | 30-60 min |
| **Learning Curve** | Steep | Easy | Easy | Moderate | Moderate |
| **Plugins** | 1,800+ | 1,000+ actions | Limited | Limited | Marketplace |
| **Docker Native** | Via plugin | Yes | Yes | Yes | Yes |
| **Kubernetes** | Via plugin | Yes | Yes | Yes | Azure AKS |
| **Maintenance** | You manage | Zero | Minimal | Zero | Minimal |
| **Self-Hosted** | Yes | Runners | Yes | Runners | Yes |
| **Open Source** | Yes | No | Core only | No | Partial |
| **Market Share** | #1 (70%) | Fast growing | Growing | Growing | Strong (MS) |

*Self-hosted: your infrastructure costs apply

### When to Choose What

```
Jenkins:       Complex workflows · Full control · On-premise · Multi-tool
GitHub Actions: GitHub users · Quick start · Simple CI · Generous free tier  
GitLab CI:     GitLab repos · All-in-one · Built-in security · Auto DevOps
CircleCI:      Zero maintenance · Fast setup · Docker-native workflows
Azure DevOps:  Microsoft/Azure stack · .NET projects · Enterprise MS shops
```

---

## 14. Troubleshooting & Tips

### Common Issues & Fixes

```
ISSUE: Build stuck "waiting for available executor"
FIX:   Check agent connectivity, add more executors, verify labels match

ISSUE: "No such DSL method" error
FIX:   Install required plugin, restart Jenkins

ISSUE: Git clone fails
FIX:   Check credentials, SSH keys, firewall, Git plugin installed

ISSUE: Out of disk space
FIX:   Set buildDiscarder in options, clear old workspaces:
       find /var/lib/jenkins/workspace -maxdepth 1 -mtime +30 -exec rm -rf {} \;

ISSUE: Build passes locally, fails in Jenkins
FIX:   Check Java/tool versions, environment variables, PATH differences

ISSUE: Jenkins UI very slow
FIX:   Increase heap: JAVA_OPTS="-Xmx4g -Xms2g", add more agents

ISSUE: Credentials not found
FIX:   Verify credential ID, scope (Global vs System), restart Jenkins

ISSUE: "Unable to connect to agent"
FIX:   Check SSH/JNLP settings, firewall port 50000, Java on agent
```

### Useful CLI Commands

```bash
# Jenkins CLI jar
wget http://localhost:8080/jnlpJars/jenkins-cli.jar

# List all jobs
java -jar jenkins-cli.jar -s http://localhost:8080/ list-jobs

# Trigger a build
java -jar jenkins-cli.jar -s http://localhost:8080/ build my-pipeline

# Restart Jenkins
java -jar jenkins-cli.jar -s http://localhost:8080/ restart

# Reload configuration
java -jar jenkins-cli.jar -s http://localhost:8080/ reload-configuration

# Get build log
java -jar jenkins-cli.jar -s http://localhost:8080/ console my-pipeline 42
```

### REST API Examples

```bash
# Get list of jobs
curl http://user:token@localhost:8080/api/json?pretty=true

# Trigger build
curl -X POST http://user:token@localhost:8080/job/my-pipeline/build

# Trigger with parameters
curl -X POST http://user:token@localhost:8080/job/my-pipeline/buildWithParameters \
  --data "ENV=prod&VERSION=1.2.3"

# Get build info
curl http://user:token@localhost:8080/job/my-pipeline/lastBuild/api/json

# Get build status
curl http://user:token@localhost:8080/job/my-pipeline/lastBuild/api/json \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['result'])"
```

---

## Quick Reference Card

```
Jenkins Pipeline Cheat Sheet
═══════════════════════════════════════════════════════════════════════

STRUCTURE                         COMMON STEPS
─────────                         ────────────
pipeline {                        sh 'command'           // Linux shell
  agent any/none/{label}          bat 'command'          // Windows cmd
  environment { KEY='val' }       echo 'message'
  options { timeout() ... }       checkout scm           // Clone repo
  parameters { string() ... }     git url: '...', branch: 'main'
  triggers { pollSCM() ... }      archiveArtifacts '**/*.jar'
  stages {                        junit '**/test-results/*.xml'
    stage('Name') {               stash name: 'x', includes: '**'
      agent { ... }               unstash 'x'
      when { branch 'main' }      input message: 'Approve?'
      steps { ... }               sleep 30
      post { ... }                readFile 'config.txt'
    }                             writeFile file: 'out.txt', text: 'hi'
  }                               withEnv(['VAR=value']) { ... }
  post { always/success/failure } withCredentials([...]) { ... }
}                                 parallel { stage() {} stage() {} }

AGENTS                            TRIGGERS
──────                            ────────
agent any                         cron('H 2 * * *')
agent none                        pollSCM('H/5 * * * *')
agent { label 'linux' }           upstream(upstreamProjects: 'job-A')
agent { docker { image '...' }}   githubPush()

WHEN CONDITIONS                   POST CONDITIONS
───────────────                   ───────────────
branch 'main'                     always
changeRequest()                   success
expression { return true }        failure
environment name: 'X', value: 'Y' unstable
not { branch 'dev' }              changed
anyOf { branch 'a'; branch 'b' }  fixed
allOf { ... }                     aborted
```

---

*Guide covers Jenkins 2.x LTS. Always check official docs: https://www.jenkins.io/doc/*
