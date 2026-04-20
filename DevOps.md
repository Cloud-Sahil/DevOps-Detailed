# 🚀 Introduction to DevOps
---
## 🔹 What is DevOps?
DevOps is a software development approach that close collaboration & communication between Development `(Dev)` & Operations `(Ops)` teams to deliver software and services

The main goal of DevOps is to:

 - Automate processes
 - Improve collaboration between teams
 - Deliver high-quality software quickly

## 🔹 What is Software & Application?
 - **Software:** A collection of programs, data, and instructions that tell a computer what to do.
 - **Application:** A type of software designed for end users (e.g., mobile apps, web apps).

👉 Example:
 - Software → Operating System
 - Application → Banking App, Instagram
   
## 🔹 What is an Application?
An application is a **user-facing program** that performs specific tasks like:
 - Online shopping
 - Social media
 - Banking

It usually includes:
 - Frontend (UI)
 - Backend (logic)
 - Database

## 🔹 Types of Applications
 - Web Applications (browser-based)
 - Mobile Applications (Android/iOS)
 - Desktop Applications
 - Cloud-native Applications

## 🔹 What is SDLC (Software Development Life Cycle)?

SDLC is a structured process used to develop software step-by-step.

## 📌 SDLC Phases:
1. Requirement Gathering
2. Design
3. Development
4. Testing
5. Deployment
6. Maintenance

## 🔹 SDLC Models
Different approaches to follow SDLC:
 - **Waterfall Model** → Linear, step-by-step
 - **Agile Model** → Iterative and flexible
 - **Spiral Model** → Risk-driven
 - **V-Model** → Testing-focused

## 🔹 Environments in DevOps
Different stages where application runs:
 - **Development (Dev)** → Developers build code
 - **Testing (QA)** → Test application
 - **Staging (UAT)** → Production-like testing
 - **Production (Prod)** → Live users use the app

## 🔹 DevOps Lifecycle
```hcl
Plan → Code → Build → Test → Release → Deploy → Operate → Monitor
```
## 🔹 DevOps Lifecycle Explanation
 - Plan → Define requirements
 - Code → Write application code
 - Build → Compile & package code
 - Test → Run automated tests
 - Release → Prepare for deployment
 - Deploy → Push to servers/cloud
 - Operate → Run application
 - Monitor → Track performance

## 🔹 DevOps Tools (Detailed)

🔧 **Version Control**
 - Git → Track code changes
 - GitHub / GitLab → Store repositories

⚙️ **CI/CD Tools**
 - Jenkins → Automate build & deploy
 - GitHub Actions → Workflow automation
 - GitLab → Built-in CI/CD

🐳 **Containerization**
 - Docker → Package applications

☸️**Orchestration**
 - Kubernetes → Manage containers

🏗️ **Infrastructure as Code**
 - Terraform → Automate infrastructure

🔍 **Code Quality**
 - SonarQube → Analyze code

📊 **Monitoring & Logging**
 - Datadog → Monitor apps
 - Grafana → Dashboards

## 🔹 Types of DevOps Practices
 - Continuous Integration (CI)
 - Continuous Delivery (CD)
 - Continuous Deployment
 - Infrastructure as Code (IaC)
 - Monitoring & Logging
 - Microservices Architecture

---
# 🏗️ Complete DevOps Architecture
## DevOps End-to-End Architecture Diagram
        👨‍💻 Developer
              |
              ▼
        +-------------+
        |   Git Repo  |
        | (GitHub /   |
        |  GitLab)    |
        +------+------+ 
               |
               ▼
        +-------------+
        |   CI/CD     |
        | (Jenkins /  |
        | GitHub CI)  |
        +------+------+ 
               |
     -----------------------
     |         |           |
     ▼         ▼           ▼
+--------+ +--------+ +------------+
| Build  | | Test   | | Code Scan  |
|        | |        | | (SonarQube)|
+--------+ +--------+ +------------+
               |
               ▼
        +-------------+
        | Docker Build|
        |   (Image)   |
        +------+------+ 
               |
               ▼
        +-------------+
        | Push to     |
        | Registry    |
        | (DockerHub) |
        +------+------+ 
               |
               ▼
        +-------------+
        | Kubernetes  |
        | Deployment  |
        +------+------+ 
               |
     -----------------------
     |         |           |
     ▼         ▼           ▼
+--------+ +--------+ +------------+
|  Pods  | | Service| | Ingress    |
+--------+ +--------+ +------------+
               |
               ▼
        +-------------+
        | Monitoring  |
        | (Datadog /  |
        | Grafana)    |
        +------+------+ 
               |
               ▼
        🔁 Feedback Loop
               |
               ▼
        👨‍💻 Developer

