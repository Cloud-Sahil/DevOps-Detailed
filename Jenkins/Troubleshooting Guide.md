# Jenkins & CI/CD — Troubleshooting Guide

> **Purpose:** Diagnose and fix real-world Jenkins & CI/CD issues fast  
> **Structure:** Problem → Symptom → Root Cause → Fix → Prevention  
> **Level:** Beginner to Production SRE

---

## Table of Contents

1. [Quick Diagnostic Checklist](#1-quick-diagnostic-checklist)
2. [Jenkins Startup & Installation Issues](#2-jenkins-startup--installation-issues)
3. [Build & Job Failures](#3-build--job-failures)
4. [Pipeline & Jenkinsfile Errors](#4-pipeline--jenkinsfile-errors)
5. [Agent / Node Issues](#5-agent--node-issues)
6. [Git & SCM Issues](#6-git--scm-issues)
7. [Plugin Issues](#7-plugin-issues)
8. [Credentials & Security Issues](#8-credentials--security-issues)
9. [Performance & Disk Issues](#9-performance--disk-issues)
10. [Docker-in-Jenkins Issues](#10-docker-in-jenkins-issues)
11. [Kubernetes Agent Issues](#11-kubernetes-agent-issues)
12. [Webhook & Trigger Issues](#12-webhook--trigger-issues)
13. [Notification Issues](#13-notification-issues)
14. [CI/CD Pipeline Anti-Patterns](#14-cicd-pipeline-anti-patterns)
15. [Jenkins Recovery & Disaster Scenarios](#15-jenkins-recovery--disaster-scenarios)
16. [Useful Commands Cheat Sheet](#16-useful-commands-cheat-sheet)

---

## 1. Quick Diagnostic Checklist

Before diving deep, run through this checklist first — it resolves 80% of issues.

```
FIRST-RESPONSE CHECKLIST
══════════════════════════════════════════════════════════════════════

[ ] Is Jenkins running?
      systemctl status jenkins
      curl -s http://localhost:8080/api/json | head

[ ] Is the agent connected?
      Dashboard → Manage Jenkins → Nodes → Check status

[ ] Is there disk space?
      df -h /var/lib/jenkins

[ ] Is there memory available?
      free -h

[ ] Are plugins up to date?
      Dashboard → Manage Jenkins → Plugins → Updates

[ ] Is the Jenkinsfile syntax valid?
      http://localhost:8080/pipeline-syntax/

[ ] Are credentials still valid?
      Dashboard → Manage Jenkins → Credentials → Check expiry

[ ] What does the console log say?
      Job → Build #N → Console Output (read last 50 lines)

[ ] Did something change recently?
      Last successful build vs current build — what's different?

[ ] Check system log:
      Dashboard → Manage Jenkins → System Log → All Jenkins Logs
```

---

## 2. Jenkins Startup & Installation Issues

---

### 2.1 Jenkins Won't Start

**Symptoms:**
- `systemctl start jenkins` fails
- No response on `http://localhost:8080`
- `Connection refused` error in browser

**Diagnosis:**

```bash
# Check service status and error logs
sudo systemctl status jenkins -l

# Check Jenkins logs directly
sudo journalctl -u jenkins -n 100 --no-pager

# Check if port 8080 is already in use
sudo ss -tlnp | grep 8080
sudo lsof -i :8080

# Check if Java is installed and correct version
java -version
which java
```

**Common Causes & Fixes:**

| Cause | Error Message | Fix |
|-------|--------------|-----|
| Port 8080 in use | `Address already in use` | Change Jenkins port or kill conflicting process |
| Wrong Java version | `Unsupported class file` | Install Java 11 or 17 |
| Corrupt JENKINS_HOME | `Failed to load` config | Restore from backup or remove corrupt config.xml |
| Out of memory | `Java heap space` | Increase heap via JAVA_OPTS |
| Missing Java | `No such file: java` | Install JDK: `apt install openjdk-17-jre` |

**Fix — Change Jenkins port:**

```bash
# Edit Jenkins default config
sudo nano /etc/default/jenkins

# Change:
HTTP_PORT=8080
# To:
HTTP_PORT=9090

sudo systemctl restart jenkins
```

**Fix — Increase Java heap:**

```bash
# /etc/default/jenkins  (Debian/Ubuntu)
JAVA_ARGS="-Xmx4g -Xms2g -XX:+UseG1GC"

# /etc/sysconfig/jenkins  (RHEL/CentOS)
JENKINS_JAVA_OPTIONS="-Xmx4g -Xms2g"

sudo systemctl restart jenkins
```

---

### 2.2 Jenkins UI Not Loading / Very Slow

**Symptoms:**
- Browser spins for 30+ seconds
- `504 Gateway Timeout` behind Nginx
- Dashboard partially loads

**Diagnosis:**

```bash
# Check CPU and memory usage
top -u jenkins
free -h

# Check Jenkins thread count
ps -eLf | grep jenkins | wc -l

# Check GC pressure in logs
sudo grep "GC" /var/log/jenkins/jenkins.log | tail -20
```

**Fixes:**

```bash
# Fix 1: Increase heap memory
JAVA_ARGS="-Xmx4g -Xms2g -XX:+UseG1GC -XX:MaxGCPauseMillis=200"

# Fix 2: Reduce build history (less data to load)
# Add to each job's Jenkinsfile:
options {
    buildDiscarder(logRotator(numToKeepStr: '20'))
}

# Fix 3: Disable unused plugins (fewer plugins = faster startup)
# Manage Jenkins → Plugins → Installed → Disable unused ones

# Fix 4: Restart Jenkins cleanly
sudo systemctl restart jenkins
# Or via URL:
curl -X POST http://admin:token@localhost:8080/restart
```

---

### 2.3 Forgotten Admin Password

**Symptoms:**
- Can't log in to Jenkins
- Password reset email not received

**Fix — Reset via filesystem:**

```bash
# Step 1: Find Jenkins config
sudo cat /var/jenkins_home/config.xml   # Docker
sudo cat /var/lib/jenkins/config.xml    # System

# Step 2: Disable security temporarily
sudo nano /var/lib/jenkins/config.xml
# Change:
<useSecurity>true</useSecurity>
# To:
<useSecurity>false</useSecurity>

# Step 3: Restart Jenkins
sudo systemctl restart jenkins

# Step 4: Log in without credentials, go to:
# Manage Jenkins → Security → Set new admin password → Re-enable security
```

**Fix — Docker container:**

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

---

### 2.4 Jenkins Update Broke Everything

**Symptoms:**
- After `apt upgrade`, Jenkins won't start
- Plugins show compatibility errors

**Diagnosis:**

```bash
sudo journalctl -u jenkins -n 200 | grep -i "error\|failed\|exception"
```

**Fix — Rollback Jenkins version:**

```bash
# Check available versions
apt-cache showpkg jenkins

# Install specific version
sudo apt-get install jenkins=2.387.3

# Hold version to prevent auto-upgrade
sudo apt-mark hold jenkins
```

**Fix — Rollback via WAR file:**

```bash
# Download specific WAR
wget https://updates.jenkins.io/download/war/2.387.3/jenkins.war

# Replace current WAR
sudo cp /usr/share/java/jenkins.war /usr/share/java/jenkins.war.bak
sudo cp jenkins.war /usr/share/java/jenkins.war

sudo systemctl restart jenkins
```

---

## 3. Build & Job Failures

---

### 3.1 Build Fails Immediately — "No such DSL method"

**Symptoms:**

```
groovy.lang.MissingMethodException: No signature of method
WorkflowScript.docker() is applicable
```

**Root Cause:** Required plugin not installed or outdated.

**Fix:**

```bash
# Identify which plugin provides the method
# docker()        → Docker Pipeline plugin
# kubernetes()    → Kubernetes plugin
# withSonarQubeEnv() → SonarQube Scanner plugin
# slackSend()     → Slack Notification plugin

# Install via Manage Jenkins → Plugins → Available
# Then restart Jenkins
```

---

### 3.2 Build Stuck — "Waiting for next available executor"

**Symptoms:**
- Job queued for minutes/hours
- Console shows: `Still waiting to schedule task`
- `Waiting for next available executor` in build log

**Diagnosis:**

```bash
# Check node status
curl http://admin:token@localhost:8080/computer/api/json?pretty=true | \
  python3 -c "import sys,json; nodes=json.load(sys.stdin)['computer']; \
  [print(n['displayName'], 'offline:', n['offline']) for n in nodes]"
```

**Common Causes & Fixes:**

| Cause | Fix |
|-------|-----|
| All executors busy | Add more agents or increase executor count |
| Label mismatch | Job requires `label 'linux'` but no agent has that label |
| All agents offline | Reconnect agents or check SSH connectivity |
| Master has 0 executors | Enable executors on master (not recommended for prod) |
| Job requires offline node | Update job/pipeline to use available label |

**Fix — Check and fix label mismatch:**

```groovy
// Jenkinsfile — check your label matches an active agent
agent { label 'linux-build' }    // Must match agent label exactly

// Or use 'any' for any available agent
agent any
```

**Fix — Add executors to an agent:**

```
Dashboard → Manage Jenkins → Nodes → [Agent Name]
→ Configure → Number of executors → Increase
```

---

### 3.3 Build Passes Locally but Fails in Jenkins

**Symptoms:**
- `mvn test` works on developer machine
- Same command fails in Jenkins pipeline

**Diagnosis Framework:**

```bash
# In pipeline, print environment to compare
steps {
    sh '''
        echo "=== Java Version ===" && java -version
        echo "=== Maven Version ===" && mvn -version
        echo "=== PATH ===" && echo $PATH
        echo "=== User ===" && whoami
        echo "=== Working Dir ===" && pwd
        echo "=== Env Vars ===" && env | sort
    '''
}
```

**Common Causes & Fixes:**

```
CAUSE                        FIX
─────────────────────────    ──────────────────────────────────────────
Different Java version        Install correct Java on agent, set JAVA_HOME
Different Maven/tool version  Use Jenkins tool installer for consistent versions
Missing env variable          Add to withEnv() or Jenkins global env config
Hardcoded absolute path       Use relative paths or $WORKSPACE variable
File permission issue         chmod before running, check jenkins user perms
Missing system dependency     Install on agent: apt install <package>
Network access blocked        Whitelist URLs or use local artifact mirror
Different working directory   Use sh 'cd mydir && command' or dir() step
```

**Fix — Consistent tool versions:**

```groovy
pipeline {
    agent any
    tools {
        maven 'Maven-3.9.0'   // Configured in Manage Jenkins → Tools
        jdk   'JDK-17'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
    }
}
```

---

### 3.4 Build Timeout — Job Runs Forever

**Symptoms:**
- Build running for hours with no output
- `Timeout has been exceeded` error
- Agent becomes unresponsive

**Fix — Add timeouts:**

```groovy
pipeline {
    options {
        timeout(time: 30, unit: 'MINUTES')  // Global pipeline timeout
    }
    stages {
        stage('Long Running Test') {
            options {
                timeout(time: 10, unit: 'MINUTES')  // Per-stage timeout
            }
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {  // Inline timeout
                    sh './deploy.sh'
                }
            }
        }
    }
}
```

**Fix — Kill a stuck build:**

```bash
# Via Jenkins UI: Build → Stop Build (×)

# Via REST API:
curl -X POST http://admin:token@localhost:8080/job/my-job/lastBuild/stop

# Via CLI:
java -jar jenkins-cli.jar -s http://localhost:8080 \
  stop-builds my-job
```

---

### 3.5 Flaky Tests — Intermittent Failures

**Symptoms:**
- Same test passes sometimes, fails sometimes
- No code change between pass and fail
- Failure is in different tests each time

**Diagnosis:**

```bash
# Check if failure correlates with time/load
# In Jenkins: Job → Trend → Test Result Trend

# Check for race conditions in logs
grep -i "timeout\|connection refused\|race\|deadlock" build.log
```

**Fix — Retry flaky tests:**

```groovy
// Option 1: Retry step
steps {
    retry(3) {
        sh 'mvn test -Punit'
    }
}

// Option 2: Retry specific test classes (Maven)
sh 'mvn test -Dsurefire.rerunFailingTestsCount=2'

// Option 3: Mark build unstable instead of failed on test failure
post {
    always {
        junit testResults: '**/surefire-reports/*.xml',
              allowEmptyResults: true
    }
}
// Then set threshold:
// unstableThreshold: 5  (5% test failure = unstable, not failed)
```

---

### 3.6 "Exit Code 1" — Command Failed

**Symptoms:**

```
+ ./deploy.sh
./deploy.sh: line 12: command not found
ERROR: script returned exit code 1
```

**Fix — Debug shell commands:**

```groovy
steps {
    sh '''
        set -e          # Exit on any error
        set -x          # Print each command (debug mode)
        set -u          # Error on undefined variables
        set -o pipefail # Pipe failure propagates

        ./deploy.sh
    '''
}
```

**Fix — Capture and inspect return code:**

```groovy
script {
    def exitCode = sh(
        script: './deploy.sh',
        returnStatus: true
    )
    if (exitCode != 0) {
        error "Deploy script failed with exit code: ${exitCode}"
    }
}
```

---

## 4. Pipeline & Jenkinsfile Errors

---

### 4.1 Jenkinsfile Syntax Error

**Symptoms:**

```
org.codehaus.groovy.control.MultipleCompilationErrorsException:
startup failed: WorkflowScript: 15: unexpected token: } @ line 15
```

**Fix — Validate before committing:**

```bash
# Method 1: Jenkins Lint (via API)
curl -X POST -F "jenkinsfile=<Jenkinsfile" \
  http://admin:token@localhost:8080/pipeline-model-converter/validate

# Method 2: Use Jenkins Pipeline Linter in VS Code
# Install extension: Jenkins Pipeline Linter Connector

# Method 3: Replay in Blue Ocean UI
# Open failed build → Replay → Edit and re-run
```

**Common Syntax Mistakes:**

```groovy
// ❌ WRONG — Missing quotes around label
agent { label linux }

// ✅ CORRECT
agent { label 'linux' }

// ❌ WRONG — steps inside parallel directly
parallel {
    sh 'command'
}

// ✅ CORRECT
parallel {
    stage('A') {
        steps { sh 'command' }
    }
}

// ❌ WRONG — String interpolation with single quotes
sh 'echo $BUILD_NUMBER'  // Won't expand variable in Groovy

// ✅ CORRECT — Use double quotes for variable expansion
sh "echo ${env.BUILD_NUMBER}"

// ❌ WRONG — def inside steps (not in script block)
steps {
    def x = 'hello'
}

// ✅ CORRECT
steps {
    script {
        def x = 'hello'
        echo x
    }
}
```

---

### 4.2 "Scripts not permitted to use method"

**Symptoms:**

```
org.jenkinsci.plugins.scriptsecurity.sandbox.RejectedAccessException:
Scripts not permitted to use method groovy.lang.GroovyObject
```

**Root Cause:** Groovy Sandbox blocked the method for security reasons.

**Fix:**

```
Dashboard → Manage Jenkins → In-process Script Approval
→ Find the blocked method → Approve
```

**Prevent by using allowed alternatives:**

```groovy
// ❌ Blocked: Direct file read
def content = new File('/etc/passwd').text

// ✅ Allowed: Use Jenkins step
def content = readFile('/path/to/file')

// ❌ Blocked: System.exit()
System.exit(1)

// ✅ Allowed:
error 'Something went wrong'

// ❌ Blocked: Thread.sleep()
Thread.sleep(5000)

// ✅ Allowed:
sleep(5)
```

---

### 4.3 "currentBuild.result" Is Null

**Symptoms:**

```groovy
// This returns null mid-pipeline
echo currentBuild.result  // prints "null"
```

**Explanation:** `currentBuild.result` is `null` while the build is running. It's only set after the build completes.

**Fix — Use `currentBuild.currentResult` instead:**

```groovy
// During pipeline execution:
echo currentBuild.currentResult  // Returns: 'SUCCESS', 'FAILURE', 'UNSTABLE'

// Or check in post section where result is finalized:
post {
    always {
        echo "Final result: ${currentBuild.result}"
    }
}
```

---

### 4.4 `when` Condition Not Working

**Symptoms:**
- Stage runs when it shouldn't (or doesn't run when it should)
- `when { branch 'main' }` not triggering correctly

**Diagnosis:**

```groovy
// Debug: Print the branch name
steps {
    echo "Branch: ${env.BRANCH_NAME}"
    echo "GIT_BRANCH: ${env.GIT_BRANCH}"
}
```

**Common `when` Pitfalls:**

```groovy
// ❌ PROBLEM: BRANCH_NAME only works in Multibranch Pipeline
// In a regular Pipeline job, BRANCH_NAME is not set
when { branch 'main' }

// ✅ FIX for regular pipeline jobs:
when {
    expression {
        return env.GIT_BRANCH == 'origin/main' ||
               env.GIT_BRANCH == 'main'
    }
}

// ❌ PROBLEM: Case sensitivity
when { branch 'Main' }   // Won't match 'main'

// ✅ FIX: Match exactly
when { branch 'main' }

// ❌ PROBLEM: Glob pattern needs correct comparator
when { branch 'feature/*' }   // May not work as expected

// ✅ FIX: Specify comparator
when {
    branch pattern: 'feature/.*', comparator: 'REGEXP'
}
```

---

### 4.5 Parallel Stage Failures Not Propagating

**Symptoms:**
- One parallel branch fails but pipeline continues
- Final build result shows success even though a stage failed

**Fix:**

```groovy
stage('Parallel Tests') {
    parallel {
        stage('Unit') {
            steps { sh 'mvn test -Punit' }
        }
        stage('Integration') {
            steps { sh 'mvn test -Pintegration' }
        }
    }
    // By default, Jenkins DOES fail the parent if any parallel branch fails
    // If it's not failing, check for try/catch swallowing errors:
}

// ❌ PROBLEM: Swallowed exception
stage('Integration') {
    steps {
        script {
            try {
                sh 'mvn test -Pintegration'
            } catch (e) {
                echo "Tests failed but continuing..."
                // Error is swallowed! Build continues as SUCCESS
            }
        }
    }
}

// ✅ FIX: Re-throw or mark unstable
script {
    try {
        sh 'mvn test -Pintegration'
    } catch (e) {
        currentBuild.result = 'UNSTABLE'
        // Don't throw — marks build as unstable not failed
        // Or: throw e  — to actually fail the build
    }
}
```

---

### 4.6 `input` Step Blocking Pipeline

**Symptoms:**
- Pipeline stuck waiting forever
- "Paused for Input" message in build log
- No one received the approval notification

**Fix — Add timeout to `input`:**

```groovy
stage('Approve Production Deploy') {
    steps {
        timeout(time: 24, unit: 'HOURS') {
            input(
                message: 'Deploy to production?',
                submitter: 'admin,release-team',
                ok: 'Deploy Now'
            )
        }
    }
}

// Handle timeout gracefully
stage('Approve') {
    steps {
        script {
            try {
                timeout(time: 1, unit: 'HOURS') {
                    input message: 'Approve?', ok: 'Yes'
                }
            } catch (err) {
                // Timeout expired — abort deployment
                currentBuild.result = 'ABORTED'
                error 'Approval timed out — deployment cancelled'
            }
        }
    }
}
```

---

## 5. Agent / Node Issues

---

### 5.1 Agent Goes Offline / Disconnects

**Symptoms:**
- Node shows red X in Dashboard
- `Agent is not connected` error in build
- Builds fail mid-execution: `Remote call on [agent] failed`

**Diagnosis:**

```bash
# Check agent's Jenkins log (on the agent machine)
tail -100 /var/log/jenkins/agent.log

# Check network connectivity from master to agent
ping <agent-ip>
ssh jenkins@<agent-ip> "echo connected"

# Check agent process on agent machine
ps aux | grep jenkins
```

**Fix — Reconnect via UI:**

```
Dashboard → Manage Jenkins → Nodes → [Offline Agent]
→ Launch Agent → or → Mark this node online
```

**Fix — Restart agent via SSH (from master):**

```bash
# Kill old agent process
ssh jenkins@agent-host "pkill -f 'agent.jar' || true"

# Re-launch agent (JNLP method)
ssh jenkins@agent-host "nohup java -jar ~/agent.jar \
  -jnlpUrl http://master:8080/computer/agent1/jenkins-agent.jnlp \
  -secret <secret-token> \
  -workDir /home/jenkins &"
```

**Fix — Agent keeps disconnecting (keep-alive):**

```bash
# Add to agent SSH config on master
# /etc/ssh/ssh_config or ~/.ssh/config
Host agent-host
    ServerAliveInterval 60
    ServerAliveCountMax 10
    TCPKeepAlive yes
```

**Prevention — Automatic agent restart script:**

```bash
#!/bin/bash
# /usr/local/bin/jenkins-agent-watchdog.sh
AGENT_URL="http://master:8080/computer/agent1/jenkins-agent.jnlp"
SECRET="<agent-secret>"

while true; do
    if ! pgrep -f "agent.jar" > /dev/null; then
        echo "Agent down, restarting..."
        java -jar ~/agent.jar \
          -jnlpUrl $AGENT_URL \
          -secret $SECRET \
          -workDir /home/jenkins &
    fi
    sleep 30
done
```

---

### 5.2 Agent Connection Refused (Port 50000)

**Symptoms:**

```
JNLP agent connection error: Connection refused: master-host/192.168.1.1:50000
```

**Fix:**

```bash
# On master — check if port 50000 is listening
sudo ss -tlnp | grep 50000

# If not listening, enable in Jenkins:
# Manage Jenkins → Security → Agents
# TCP port for inbound agents: Fixed = 50000

# Open firewall on master
sudo ufw allow 50000/tcp
sudo firewall-cmd --add-port=50000/tcp --permanent
sudo firewall-cmd --reload
```

---

### 5.3 "No Space Left on Agent"

**Symptoms:**

```
IOException: No space left on device
error: failed to push some refs to repository
```

**Fix:**

```bash
# Check disk space on agent
df -h

# Find large Jenkins workspace directories
du -sh /home/jenkins/workspace/* | sort -rh | head -20

# Clean old workspaces (older than 7 days)
find /home/jenkins/workspace -maxdepth 1 -mtime +7 -exec rm -rf {} \;

# Clean Docker images if Docker is used on agent
docker system prune -af --volumes

# In pipeline — always clean workspace
post {
    always {
        cleanWs()
    }
}
```

---

## 6. Git & SCM Issues

---

### 6.1 Git Clone Fails

**Symptoms:**

```
ERROR: Error cloning remote repo 'origin'
fatal: Authentication failed for 'https://github.com/org/repo.git'
```

**Diagnosis:**

```bash
# Test manually on agent
ssh jenkins@agent-host
git clone https://github.com/org/repo.git /tmp/test-clone
# Or with SSH:
git clone git@github.com:org/repo.git /tmp/test-clone
```

**Fix — Wrong credential type:**

```groovy
// For HTTPS repos — use Username/Password credential
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    userRemoteConfigs: [[
        url: 'https://github.com/org/repo.git',
        credentialsId: 'github-pat-credential'  // Personal Access Token
    ]]
])

// For SSH repos — use SSH private key credential
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    userRemoteConfigs: [[
        url: 'git@github.com:org/repo.git',
        credentialsId: 'github-ssh-key'
    ]]
])
```

**Fix — Add SSH key to known_hosts:**

```bash
# On agent machine (as jenkins user)
sudo -u jenkins ssh-keyscan github.com >> ~/.ssh/known_hosts
sudo -u jenkins ssh-keyscan gitlab.com >> ~/.ssh/known_hosts

# Test
sudo -u jenkins ssh -T git@github.com
# Should print: Hi username! You've successfully authenticated
```

---

### 6.2 Checkout Fails — SSL Certificate Error

**Symptoms:**

```
fatal: unable to access 'https://internal-git/':
SSL certificate problem: self signed certificate
```

**Fix — Trust the certificate:**

```bash
# Option 1: Add cert to system trust store (preferred)
sudo cp internal-ca.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

# Option 2: Configure Git to trust it (on agent)
sudo -u jenkins git config --global http.sslCAInfo /path/to/ca.crt

# Option 3: Disable SSL verification (⚠️ not for production)
sudo -u jenkins git config --global http.sslVerify false
```

---

### 6.3 Webhook Not Triggering Builds

**Symptoms:**
- Git push happens but Jenkins doesn't start a build
- Webhook payload is sent but Jenkins doesn't respond

**Diagnosis:**

```bash
# Check webhook delivery in GitHub/GitLab:
# GitHub: Repository → Settings → Webhooks → Recent Deliveries
# Look for failed deliveries (red X)

# Check Jenkins webhook endpoint is reachable
curl -X POST http://your-jenkins-url/github-webhook/ \
  -H "Content-Type: application/json" \
  -d '{}'

# Check Jenkins system log for webhook events
# Dashboard → Manage Jenkins → System Log → All Jenkins Logs
```

**Fix — Webhook URL format:**

```
GitHub webhook URL:
  http://JENKINS_URL/github-webhook/
                    ↑ trailing slash required!

GitLab webhook URL:
  http://JENKINS_URL/project/JOB_NAME

Generic webhook:
  http://JENKINS_URL/generic-webhook-trigger/invoke?token=TOKEN
```

**Fix — Jenkins behind reverse proxy (Nginx):**

```nginx
# /etc/nginx/sites-available/jenkins
server {
    listen 443 ssl;
    server_name jenkins.company.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;  # ← critical for webhooks
    }
}
```

```
Dashboard → Manage Jenkins → System → Jenkins URL
Set to: https://jenkins.company.com/   ← Must match your actual URL
```

---

### 6.4 Submodule Checkout Fails

**Symptoms:**

```
Submodule 'libs/common' could not be checked out
```

**Fix:**

```groovy
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    extensions: [[
        $class: 'SubmoduleOption',
        disableSubmodules: false,
        recursiveSubmodules: true,
        trackingSubmodules: false
    ]],
    userRemoteConfigs: [[
        url: 'git@github.com:org/repo.git',
        credentialsId: 'github-ssh-key'
    ]]
])
```

---

## 7. Plugin Issues

---

### 7.1 Plugin Installation Fails

**Symptoms:**

```
Failed to install plugin: Could not resolve dependencies
Plugin X requires Plugin Y version 2.x but version 1.x is installed
```

**Fix — Install dependencies first:**

```
Manage Jenkins → Plugins → Available
Search and install both the plugin AND its dependencies
Check "Restart Jenkins when installation is complete"
```

**Fix — Install manually via CLI:**

```bash
# Download plugin .hpi file from:
# https://plugins.jenkins.io/<plugin-name>/#releases

# Install via CLI
java -jar jenkins-cli.jar -s http://localhost:8080 \
  install-plugin /path/to/plugin.hpi -restart
```

**Fix — Reset broken plugins:**

```bash
# Disable a broken plugin (rename its directory)
sudo systemctl stop jenkins
sudo mv /var/lib/jenkins/plugins/broken-plugin \
        /var/lib/jenkins/plugins/broken-plugin.bak
sudo systemctl start jenkins
```

---

### 7.2 Plugin Causes Jenkins to Crash on Startup

**Symptoms:**
- Jenkins crashes immediately after update
- Logs show: `Failed to initialize plugin X`

**Fix — Safe restart mode:**

```bash
# Start Jenkins with no plugins
java -jar jenkins.war --pluginroot=/tmp/empty-plugins

# Then go to UI and disable the problematic plugin
# Or delete it from filesystem:
sudo rm -rf /var/lib/jenkins/plugins/problem-plugin*
sudo systemctl restart jenkins
```

---

## 8. Credentials & Security Issues

---

### 8.1 Credential Not Found

**Symptoms:**

```
ERROR: No credential found with id: 'my-docker-cred'
CredentialsUnavailableException
```

**Diagnosis Checklist:**

```
[ ] Does the credential ID match exactly? (case-sensitive)
[ ] Is the credential in the right scope?
      Global = accessible by all jobs
      System = Jenkins internal use only
      Folder = only jobs in that folder
[ ] Is the job in a different folder than the credential?
[ ] Did someone delete or rename the credential?
```

**Fix — Verify credential ID:**

```groovy
// In Jenkinsfile — credential ID must match exactly
withCredentials([usernamePassword(
    credentialsId: 'my-docker-cred',  // ← Must match ID in Jenkins UI
    ...
)])

// List credentials via script console:
// Manage Jenkins → Script Console
import com.cloudbees.plugins.credentials.*
def creds = CredentialsProvider.lookupCredentials(
    com.cloudbees.plugins.credentials.common.StandardCredentials.class,
    Jenkins.instance, null, null
)
creds.each { println it.id + " : " + it.class.simpleName }
```

---

### 8.2 Secret Exposed in Build Log

**Symptoms:**
- Password/token visible in console output
- `echo $MY_SECRET` prints the actual secret

**Fix — Ensure masking is active:**

```groovy
// ✅ withCredentials auto-masks secrets in logs
withCredentials([string(credentialsId: 'api-token', variable: 'TOKEN')]) {
    sh 'curl -H "Authorization: Bearer $TOKEN" https://api.example.com'
    // Logs show: Authorization: Bearer ****
}

// ❌ NEVER do this — secret in plain text
sh "curl -H 'Authorization: Bearer ${SECRET_VAR}' https://api.com"
// Logs show the actual secret!
```

**Fix — Mask additional sensitive values:**

```groovy
// If you need to mask a value not from credentials
wrap([$class: 'MaskPasswordsBuildWrapper',
      varPasswordPairs: [[password: "${env.MY_VALUE}", var: 'MY_VALUE']]]) {
    sh 'echo $MY_VALUE'  // Logs show ****
}
```

**If secret was already exposed:**

```bash
# 1. Revoke the exposed secret IMMEDIATELY
# 2. Rotate: generate new API key/password
# 3. Update the Jenkins credential with new value
# 4. Delete the old build log:
#    Job → Build #N → Delete this build (gear icon)
```

---

### 8.3 CSRF Crumb Issues with API Calls

**Symptoms:**

```
HTTP 403 Forbidden
Error: No valid crumb was included in the request
```

**Fix — Include crumb in API calls:**

```bash
# Step 1: Get crumb
CRUMB=$(curl -s -u admin:token \
  'http://localhost:8080/crumbIssuer/api/json' \
  | python3 -c "import sys,json; d=json.load(sys.stdin); \
    print(d['crumbRequestField']+':'+d['crumb'])")

# Step 2: Use crumb in POST request
curl -X POST -H "$CRUMB" \
  -u admin:token \
  http://localhost:8080/job/my-job/build
```

---

## 9. Performance & Disk Issues

---

### 9.1 Jenkins Running Out of Disk Space

**Symptoms:**

```
java.io.IOException: No space left on device
Build archived artifacts failure
```

**Immediate cleanup:**

```bash
# Check disk usage breakdown
df -h
du -sh /var/lib/jenkins/*
du -sh /var/lib/jenkins/jobs/*/builds | sort -rh | head -20
du -sh /var/lib/jenkins/workspace/* | sort -rh | head -20

# Quick wins:
# 1. Clean workspaces
find /var/lib/jenkins/workspace -maxdepth 1 -mtime +7 -exec rm -rf {} \;

# 2. Clean old build logs
find /var/lib/jenkins/jobs -name "log" -mtime +30 -delete

# 3. Remove unused plugin downloads
find /var/lib/jenkins/plugins -name "*.bak" -delete

# 4. Docker cleanup (if using Docker builds on master)
docker system prune -af --volumes
```

**Permanent fix — Build retention policies:**

```groovy
// In every pipeline:
options {
    buildDiscarder(logRotator(
        numToKeepStr:         '30',   // Keep 30 builds
        daysToKeepStr:        '90',   // Or builds older than 90 days
        artifactNumToKeepStr: '10',   // Keep artifacts for 10 builds
        artifactDaysToKeepStr:'30'    // Or artifact builds older than 30 days
    ))
}

// Always clean workspace:
post {
    always { cleanWs() }
}
```

---

### 9.2 High CPU Usage on Jenkins Master

**Symptoms:**
- Jenkins master at 100% CPU
- Builds are slow or unresponsive
- UI laggy

**Diagnosis:**

```bash
# Check what's using CPU
top -u jenkins -b -n1 | head -30

# Check thread dump for Java
jstack $(pgrep -f jenkins.war) > /tmp/jenkins-threads.txt
grep -A 5 "RUNNABLE" /tmp/jenkins-threads.txt | head -50
```

**Fixes:**

```bash
# Fix 1: Move builds OFF master — add agents
# Set master executors to 0:
# Manage Jenkins → Nodes → Built-In Node → # of executors = 0

# Fix 2: Throttle concurrent builds
# Manage Jenkins → System → # of executors on master = 0
# Add agent with appropriate executor count

# Fix 3: Use G1GC for better garbage collection
JAVA_ARGS="-Xmx4g -Xms2g -XX:+UseG1GC -XX:MaxGCPauseMillis=200"

# Fix 4: Disable unused plugins
# Every plugin adds CPU overhead at startup and runtime
```

---

### 9.3 Build Queue Backed Up

**Symptoms:**
- 50+ builds waiting in queue
- New builds wait 30+ minutes before starting

**Fixes:**

```bash
# Short term: Add temporary agents
# Manage Jenkins → Nodes → New Node

# Medium term: Use cloud agents (auto-scale)
# Install EC2 Plugin, Kubernetes Plugin, or Azure VM Agents

# Pipeline fix: Use lightweight checkout for queue inspection
options {
    skipDefaultCheckout()   // Don't checkout in queue
}
stages {
    stage('Checkout') {
        steps {
            checkout scm   // Only checkout when executor assigned
        }
    }
}

# Also: Reduce build duration so executor turns over faster
# - Parallelize stages
# - Cache dependencies
# - Fail fast on errors
```

---

## 10. Docker-in-Jenkins Issues

---

### 10.1 "docker: command not found" in Pipeline

**Symptoms:**

```
+ docker build -t myapp .
/bin/sh: docker: not found
```

**Fix — Docker not installed on agent:**

```bash
# Install Docker on agent (Ubuntu)
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker

# Add jenkins user to docker group
sudo usermod -aG docker jenkins

# Restart Jenkins agent
sudo systemctl restart jenkins
```

**Fix — Use Docker Pipeline agent instead:**

```groovy
// Run build inside a Docker container that HAS Docker
agent {
    docker {
        image 'docker:dind'
        args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
    }
}
```

---

### 10.2 Docker Permission Denied

**Symptoms:**

```
Got permission denied while trying to connect to the Docker daemon socket
at unix:///var/run/docker.sock
```

**Fix:**

```bash
# Add jenkins user to docker group
sudo usermod -aG docker jenkins

# Fix socket permissions
sudo chmod 666 /var/run/docker.sock

# Restart Jenkins for group change to take effect
sudo systemctl restart jenkins

# Verify jenkins user can run docker
sudo -u jenkins docker ps
```

---

### 10.3 Docker Build Cache Not Working

**Symptoms:**
- Every build downloads all layers from scratch
- Build takes 10 minutes every time (should be 2 minutes with cache)

**Fix — Mount Docker cache:**

```groovy
agent {
    docker {
        image 'maven:3.9-jdk-17'
        // Mount local Maven cache to avoid re-downloading dependencies
        args '-v $HOME/.m2:/root/.m2 -v /tmp/docker-cache:/tmp/docker-cache'
    }
}
```

**Fix — BuildKit cache:**

```groovy
steps {
    sh '''
        export DOCKER_BUILDKIT=1
        docker build \
          --cache-from myapp:cache \
          --build-arg BUILDKIT_INLINE_CACHE=1 \
          -t myapp:latest \
          -t myapp:cache \
          .
        docker push myapp:cache
    '''
}
```

---

### 10.4 Cannot Pull Private Docker Image

**Symptoms:**

```
Error response from daemon: pull access denied for company/myimage
repository does not exist or may require 'docker login'
```

**Fix:**

```groovy
steps {
    withCredentials([usernamePassword(
        credentialsId: 'docker-registry-creds',
        usernameVariable: 'DOCKER_USER',
        passwordVariable: 'DOCKER_PASS'
    )]) {
        sh '''
            echo $DOCKER_PASS | docker login registry.company.com \
              -u $DOCKER_USER --password-stdin
            docker pull registry.company.com/company/myimage:latest
        '''
    }
}
```

---

## 11. Kubernetes Agent Issues

---

### 11.1 Kubernetes Pod Agent Won't Start

**Symptoms:**

```
Unable to connect to Kubernetes: connection refused
Could not allocate node for: Pipeline
```

**Diagnosis:**

```bash
# Check if Jenkins can reach Kubernetes API
curl -k https://kubernetes-api:6443/api

# Check pod creation in Kubernetes
kubectl get pods -n jenkins
kubectl describe pod <jenkins-agent-pod> -n jenkins

# Check Jenkins Kubernetes plugin config
# Manage Jenkins → Clouds → Kubernetes → Test Connection
```

**Fix — Common Kubernetes config issues:**

```
Kubernetes URL:     https://kubernetes.default.svc  (in-cluster)
                    https://<external-api>:6443      (external)

Kubernetes Namespace: jenkins  (must exist!)
Jenkins URL:          http://jenkins.jenkins.svc:8080  (in-cluster)
Jenkins tunnel:       jenkins.jenkins.svc:50000

# Create namespace if missing:
kubectl create namespace jenkins
```

---

### 11.2 Pod Agent Stuck in "Pending"

**Symptoms:**

```
pod stays in Pending state for minutes
0/3 nodes are available: 3 Insufficient memory
```

**Diagnosis:**

```bash
kubectl describe pod <agent-pod> -n jenkins | grep -A 10 Events
kubectl get nodes -o wide
kubectl top nodes
```

**Fix — Reduce resource requests:**

```groovy
agent {
    kubernetes {
        yaml """
spec:
  containers:
  - name: build
    image: maven:3.9-jdk-17
    resources:
      requests:
        cpu: '250m'      # Reduce from default
        memory: '256Mi'  # Reduce from default
      limits:
        cpu: '1'
        memory: '1Gi'
"""
    }
}
```

---

### 11.3 Kubernetes Agent Disconnects Mid-Build

**Symptoms:**
- Build fails with: `Remote call on [agent] failed`
- Pod terminated unexpectedly during build

**Diagnosis:**

```bash
# Check if pod was OOMKilled
kubectl describe pod <agent-pod> -n jenkins | grep -i "oom\|killed\|reason"

# Check pod logs before it died
kubectl logs <agent-pod> -n jenkins --previous
```

**Fix — OOM killed — increase memory:**

```groovy
agent {
    kubernetes {
        yaml """
spec:
  containers:
  - name: build
    image: maven:3.9-jdk-17
    resources:
      limits:
        memory: '4Gi'   # Increase memory limit
    env:
    - name: MAVEN_OPTS
      value: '-Xmx2g'   # Set JVM heap within container memory
"""
    }
}
```

---

## 12. Webhook & Trigger Issues

---

### 12.1 Builds Triggered Multiple Times

**Symptoms:**
- One push triggers 3–4 builds
- Duplicate builds clog the queue

**Causes & Fixes:**

```
CAUSE 1: Both webhook AND poll SCM enabled
FIX: Disable pollSCM, keep only webhook trigger

CAUSE 2: Multiple webhook registrations
FIX: GitHub → Settings → Webhooks → Remove duplicates

CAUSE 3: Multibranch + regular pipeline both configured
FIX: Use only one approach
```

**Fix — Throttle concurrent builds:**

```groovy
options {
    disableConcurrentBuilds()       // Only one build at a time
    // OR:
    throttleJobProperty(
        maxConcurrentPerNode: 1,
        maxConcurrentTotal: 2
    )
}
```

---

### 12.2 Scheduled Jobs Not Running

**Symptoms:**
- `triggers { cron('H 2 * * *') }` never runs
- Scheduled builds missing from history

**Diagnosis:**

```bash
# Check cron syntax is valid
# Use Jenkins cron checker in job config UI

# Check Jenkins system clock
date
timedatectl
```

**Fix — Common cron mistakes:**

```groovy
// ❌ WRONG: Standard cron syntax
triggers { cron('0 2 * * *') }  // Runs at exactly 2:00am (ok but avoid exact times)

// ✅ CORRECT: Use H (hash) to distribute load
triggers { cron('H 2 * * *') }  // Runs sometime between 2:00-2:59am

// ✅ Common patterns:
cron('H/15 * * * *')    // Every 15 minutes
cron('H H * * *')       // Once daily at random time
cron('H H * * 1-5')     // Weekdays only
cron('H 6 * * 1')       // Every Monday at ~6am
cron('@midnight')        // Alias for H H(0-2) * * *
cron('@weekly')          // Once a week
```

---

## 13. Notification Issues

---

### 13.1 Slack Notifications Not Sending

**Symptoms:**
- `slackSend` step runs but no message in Slack
- `Response Code: 400` or `403` in build log

**Diagnosis:**

```groovy
// Add debug output
steps {
    script {
        def response = slackSend(
            channel: '#builds',
            message: 'Test notification',
            tokenCredentialId: 'slack-token'
        )
        echo "Slack response: ${response}"
    }
}
```

**Fix — Common Slack issues:**

```
ISSUE                     FIX
───────────────────────   ──────────────────────────────────────────
Wrong token type          Use "Bot Token" (xoxb-...), not "App Token"
Bot not in channel        /invite @jenkins-bot in Slack channel
Wrong channel format      Use #channel-name, not channel-name
Credential ID mismatch    Verify credentialId matches Jenkins credential
Wrong workspace           Reconfigure Slack workspace in Jenkins global config
```

---

### 13.2 Email Notifications Not Working

**Symptoms:**
- `mail` step runs but email not received
- `AuthenticationFailedException` in logs

**Fix — SMTP configuration:**

```
Manage Jenkins → System → Email Notification

SMTP Server:    smtp.gmail.com
SMTP Port:      587
Use TLS:        ✓
Username:       your-email@gmail.com
Password:       (use App Password, not account password)

Test: Click "Test configuration by sending test e-mail"
```

**Fix — Gmail App Password:**

```
Google Account → Security → 2-Step Verification → App passwords
Create password for "Jenkins"
Use this 16-char password in Jenkins SMTP config
```

**Fix — Email Extension plugin (recommended):**

```groovy
post {
    failure {
        emailext(
            to: 'team@company.com',
            subject: "BUILD FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
Build failed!

Job:    ${env.JOB_NAME}
Build:  #${env.BUILD_NUMBER}
Status: ${currentBuild.currentResult}
URL:    ${env.BUILD_URL}

Console Output:
${currentBuild.rawBuild.getLog(50).join('\n')}
            """,
            attachLog: true
        )
    }
}
```

---

## 14. CI/CD Pipeline Anti-Patterns

These are common bad practices that cause problems — recognize and fix them.

---

### Anti-Pattern 1: Hardcoded Secrets

```groovy
// ❌ NEVER DO THIS
environment {
    DB_PASSWORD = 'SuperSecret123'
    API_KEY     = 'sk-abc123xyz'
}

// ✅ ALWAYS USE CREDENTIALS STORE
withCredentials([
    string(credentialsId: 'db-password', variable: 'DB_PASSWORD'),
    string(credentialsId: 'api-key',     variable: 'API_KEY')
]) {
    sh './deploy.sh'
}
```

---

### Anti-Pattern 2: No Pipeline Timeouts

```groovy
// ❌ Build can run forever
stages {
    stage('Tests') {
        steps { sh 'mvn test' }
    }
}

// ✅ Always set timeouts
options {
    timeout(time: 30, unit: 'MINUTES')
}
stages {
    stage('Tests') {
        options { timeout(time: 10, unit: 'MINUTES') }
        steps { sh 'mvn test' }
    }
}
```

---

### Anti-Pattern 3: Deploy Before Tests Pass

```groovy
// ❌ WRONG ORDER
stage('Deploy') { steps { sh './deploy.sh' } }
stage('Tests')  { steps { sh 'mvn test' } }

// ✅ CORRECT ORDER — fail fast
stage('Build')  { steps { sh 'mvn package' } }
stage('Tests')  { steps { sh 'mvn test' } }
stage('Deploy') { steps { sh './deploy.sh' } }
```

---

### Anti-Pattern 4: Unlimited Build History

```groovy
// ❌ Jenkins fills disk with infinite history
// No options block = keep all builds forever

// ✅ Always set retention
options {
    buildDiscarder(logRotator(numToKeepStr: '30'))
}
```

---

### Anti-Pattern 5: No Workspace Cleanup

```groovy
// ❌ Stale files from previous builds cause issues

// ✅ Always clean up
post {
    always { cleanWs() }
}
```

---

### Anti-Pattern 6: Giant Single Stage

```groovy
// ❌ Everything in one stage — no visibility, no isolation
stage('Everything') {
    steps {
        sh 'mvn package'
        sh 'mvn test'
        sh 'docker build'
        sh 'docker push'
        sh './deploy.sh'
    }
}

// ✅ Break into meaningful stages
stage('Build')       { steps { sh 'mvn package' } }
stage('Test')        { steps { sh 'mvn test' } }
stage('Docker Build'){ steps { sh 'docker build -t myapp .' } }
stage('Docker Push') { steps { sh 'docker push myapp' } }
stage('Deploy')      { steps { sh './deploy.sh' } }
```

---

### Anti-Pattern 7: Swallowing Exceptions

```groovy
// ❌ Error hidden — build shows SUCCESS when it should FAIL
steps {
    script {
        try {
            sh './deploy.sh'
        } catch (e) {
            echo "Deploy failed, ignoring..."
        }
    }
}

// ✅ Handle properly
steps {
    script {
        try {
            sh './deploy.sh'
        } catch (e) {
            currentBuild.result = 'FAILURE'
            error "Deploy failed: ${e.message}"
        }
    }
}
```

---

## 15. Jenkins Recovery & Disaster Scenarios

---

### 15.1 Backup Jenkins

```bash
#!/bin/bash
# jenkins-backup.sh — Run this daily via cron

JENKINS_HOME=/var/lib/jenkins
BACKUP_DIR=/backups/jenkins
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/jenkins-backup-$DATE.tar.gz"

mkdir -p $BACKUP_DIR

# Backup critical files (exclude workspaces and caches)
tar -czf $BACKUP_FILE \
  --exclude="$JENKINS_HOME/workspace" \
  --exclude="$JENKINS_HOME/plugins/*.bak" \
  --exclude="$JENKINS_HOME/caches" \
  --exclude="$JENKINS_HOME/fingerprints" \
  $JENKINS_HOME

echo "Backup saved: $BACKUP_FILE"

# Keep only last 30 backups
find $BACKUP_DIR -name "jenkins-backup-*.tar.gz" -mtime +30 -delete
```

```bash
# Add to cron:
crontab -e
0 2 * * * /usr/local/bin/jenkins-backup.sh >> /var/log/jenkins-backup.log 2>&1
```

---

### 15.2 Restore Jenkins from Backup

```bash
# Step 1: Stop Jenkins
sudo systemctl stop jenkins

# Step 2: Backup current state (just in case)
sudo mv /var/lib/jenkins /var/lib/jenkins.broken

# Step 3: Restore from backup
sudo mkdir /var/lib/jenkins
sudo tar -xzf /backups/jenkins/jenkins-backup-20260501_020000.tar.gz \
  -C /

# Step 4: Fix ownership
sudo chown -R jenkins:jenkins /var/lib/jenkins

# Step 5: Start Jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

---

### 15.3 Jenkins Data Corruption — Config Reset

**Symptoms:**
- Jenkins can't load a specific job config
- XML parsing error in job config
- `Unable to read` error for a job

**Fix — Reset corrupt job config:**

```bash
# Find the corrupt config
sudo ls -la /var/lib/jenkins/jobs/<job-name>/
sudo cat /var/lib/jenkins/jobs/<job-name>/config.xml

# If corrupt, restore from latest backup or recreate:
sudo cp /var/lib/jenkins/jobs/<job-name>/config.xml.bak \
        /var/lib/jenkins/jobs/<job-name>/config.xml

# Or delete job folder and recreate via UI
sudo rm -rf /var/lib/jenkins/jobs/<job-name>
# Dashboard → New Item → Recreate the job
```

---

### 15.4 Emergency: All Builds Failing After Jenkins Upgrade

```bash
# Step 1: Identify what changed
sudo grep "Started by" /var/lib/jenkins/jobs/*/builds/lastSuccessfulBuild/log 2>/dev/null

# Step 2: Rollback Jenkins to previous version
sudo systemctl stop jenkins
wget https://updates.jenkins.io/download/war/<previous-version>/jenkins.war
sudo cp jenkins.war /usr/share/java/jenkins.war
sudo systemctl start jenkins

# Step 3: If plugin compatibility issue — rollback specific plugin
sudo cp /var/lib/jenkins/plugins/<plugin>.hpi.bak \
        /var/lib/jenkins/plugins/<plugin>.hpi
sudo systemctl restart jenkins

# Step 4: Prevent auto-upgrade
sudo apt-mark hold jenkins
```

---

## 16. Useful Commands Cheat Sheet

```bash
# ══════════════════════════════════════════════════════════════
# JENKINS SERVICE
# ══════════════════════════════════════════════════════════════
sudo systemctl start jenkins
sudo systemctl stop jenkins
sudo systemctl restart jenkins
sudo systemctl status jenkins
sudo journalctl -u jenkins -n 100 --no-pager

# ══════════════════════════════════════════════════════════════
# JENKINS LOGS
# ══════════════════════════════════════════════════════════════
sudo tail -100 /var/log/jenkins/jenkins.log
sudo grep -i "error\|exception\|failed" /var/log/jenkins/jenkins.log | tail -50

# ══════════════════════════════════════════════════════════════
# DISK CLEANUP
# ══════════════════════════════════════════════════════════════
df -h /var/lib/jenkins
du -sh /var/lib/jenkins/* | sort -rh
find /var/lib/jenkins/workspace -maxdepth 1 -mtime +7 -exec rm -rf {} \;
find /var/lib/jenkins/jobs -name "log" -mtime +30 -delete

# ══════════════════════════════════════════════════════════════
# JENKINS CLI
# ══════════════════════════════════════════════════════════════
# Download CLI jar
wget http://localhost:8080/jnlpJars/jenkins-cli.jar

# Common commands (use: java -jar jenkins-cli.jar -s URL -auth user:token CMD)
JCLI="java -jar jenkins-cli.jar -s http://localhost:8080 -auth admin:token"

$JCLI list-jobs                           # List all jobs
$JCLI build my-job                        # Trigger build
$JCLI build my-job -p ENV=prod            # Trigger with parameters
$JCLI console my-job 42                   # Get console output of build #42
$JCLI get-job my-job                      # Export job XML config
$JCLI delete-job my-job                   # Delete a job
$JCLI disable-job my-job                  # Disable a job
$JCLI enable-job my-job                   # Enable a job
$JCLI install-plugin git -restart         # Install plugin + restart
$JCLI reload-configuration                # Reload config from disk
$JCLI restart                             # Restart Jenkins
$JCLI safe-restart                        # Restart after current builds finish
$JCLI offline-node agent1 -m "Maintenance"  # Take agent offline
$JCLI online-node agent1                  # Bring agent online

# ══════════════════════════════════════════════════════════════
# REST API
# ══════════════════════════════════════════════════════════════
BASE="http://admin:token@localhost:8080"

# List all jobs
curl -s "$BASE/api/json" | python3 -m json.tool

# Trigger build
curl -X POST "$BASE/job/my-job/build"

# Trigger with parameters
curl -X POST "$BASE/job/my-job/buildWithParameters?BRANCH=main&ENV=prod"

# Get last build info
curl -s "$BASE/job/my-job/lastBuild/api/json" | python3 -m json.tool

# Get build result
curl -s "$BASE/job/my-job/lastBuild/api/json" | \
  python3 -c "import sys,json; print(json.load(sys.stdin)['result'])"

# Get queue info
curl -s "$BASE/queue/api/json" | python3 -m json.tool

# Cancel a build
curl -X POST "$BASE/job/my-job/42/stop"

# ══════════════════════════════════════════════════════════════
# GROOVY SCRIPT CONSOLE (Manage Jenkins → Script Console)
# ══════════════════════════════════════════════════════════════

# List all jobs and their last build result:
Jenkins.instance.getAllItems(Job.class).each {
    def build = it.lastBuild
    println "${it.fullName}: ${build?.result ?: 'NO BUILDS'}"
}

# Cancel all queued builds:
Jenkins.instance.queue.items.each {
    Jenkins.instance.queue.cancel(it.task)
}

# Abort all running builds:
Jenkins.instance.getAllItems(Job.class).each { job ->
    job.builds.findAll { it.inProgress }.each {
        it.doStop()
        println "Stopped: ${job.name} #${it.number}"
    }
}

# List all credentials IDs:
import com.cloudbees.plugins.credentials.*
CredentialsProvider.lookupCredentials(
    com.cloudbees.plugins.credentials.common.StandardCredentials.class,
    Jenkins.instance, null, null
).each { println "${it.id} (${it.class.simpleName})" }

# Free disk by deleting old builds of a job:
def job = Jenkins.instance.getItemByFullName('my-job')
job.builds.findAll { it.number < (job.lastBuild.number - 50) }.each {
    println "Deleting build #${it.number}"
    it.delete()
}

# Reload all job configs from disk:
Jenkins.instance.getAllItems(Job.class).each {
    it.doReload()
}

# ══════════════════════════════════════════════════════════════
# GIT DEBUG IN PIPELINE
# ══════════════════════════════════════════════════════════════
sh 'git log --oneline -5'
sh 'git branch -a'
sh 'git remote -v'
sh 'git status'
sh 'git config --list'

# ══════════════════════════════════════════════════════════════
# DOCKER DEBUG IN PIPELINE
# ══════════════════════════════════════════════════════════════
sh 'docker version'
sh 'docker info'
sh 'docker ps -a'
sh 'docker images | head -20'
sh 'docker system df'
sh 'id && groups'                    # Check user and docker group membership

# ══════════════════════════════════════════════════════════════
# PIPELINE DEBUG HELPERS
# ══════════════════════════════════════════════════════════════
# Print all environment variables (debug builds):
sh 'env | sort'

# Print disk usage in workspace:
sh 'df -h && du -sh * | sort -rh | head -20'

# Check network from agent:
sh 'curl -s --max-time 5 https://registry.npmjs.org || echo "NPM registry unreachable"'
sh 'nslookup github.com || echo "DNS resolution failed"'
```

---

## Error Message → Quick Fix Reference

```
ERROR MESSAGE                              LIKELY FIX
─────────────────────────────────────────  ─────────────────────────────────────────
"No space left on device"                  Clean workspaces, set buildDiscarder
"Connection refused :8080"                 Start Jenkins service
"Connection refused :50000"                Open firewall port 50000
"Waiting for next available executor"      Add agents or increase executor count
"No valid crumb was included"              Add CSRF crumb to API requests
"Scripts not permitted to use method"      Approve in Script Security page
"No credential found with id"             Check credential ID spelling and scope
"Authentication failed"                    Update Git credentials / SSH keys
"SSL certificate problem"                  Add CA cert to trust store
"No such DSL method"                       Install missing plugin
"Address already in use :8080"             Change port or kill conflicting process
"Java heap space"                          Increase JAVA_OPTS -Xmx
"docker: command not found"                Install Docker, add jenkins to docker group
"Permission denied /var/run/docker.sock"   usermod -aG docker jenkins
"Unsupported class file major version"     Wrong Java version — use Java 11 or 17
"Remote call failed"                       Agent disconnected — reconnect or fix network
"OOMKilled" (Kubernetes)                   Increase memory limits in pod spec
"ImagePullBackOff"                         Docker login or fix image name/tag
"Build was aborted"                        Timeout expired or manual stop
"checkout scm: No SCM"                     Job must be backed by SCM (not freestyle)
```

---

*Always check the full console log first — Jenkins error messages are usually specific and actionable. When in doubt: restart, check logs, check disk space.*
