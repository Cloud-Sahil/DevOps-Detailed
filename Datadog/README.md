# <img src="https://cdn.simpleicons.org/datadog" height="40" /> Datadog
 - (Monitoring & Observability Platform)
---

## 🔹 What is Datadog?

Datadog is a cloud-based monitoring and observability platform used to monitor:

- Infrastructure
- Applications
- Logs
- Containers
- Kubernetes
- Cloud services
- CI/CD pipelines

👉 It provides real-time monitoring, alerting, dashboards, and analytics.

Company:
- Developed by Datadog Inc.

---

# 🔹 Why Datadog is Used?

Datadog helps DevOps and SRE teams to:

- Monitor servers
- Detect application issues
- Track logs
- Analyze performance
- Monitor Kubernetes clusters
- Create alerts
- Improve troubleshooting

---

# 🔹 Datadog Features

| Feature | Purpose |
|---|---|
| Infrastructure Monitoring | Monitor servers & cloud |
| Log Management | Collect & analyze logs |
| APM | Application performance monitoring |
| Dashboard | Visualize metrics |
| Alerting | Send alerts automatically |
| Kubernetes Monitoring | Monitor K8s clusters |
| Container Monitoring | Monitor Docker containers |
| Cloud Monitoring | AWS, Azure, GCP monitoring |
| Network Monitoring | Analyze network traffic |
| Security Monitoring | Detect threats |

---

# 🔹 Types of Monitoring in Datadog

## 1. Infrastructure Monitoring
Monitors:
- CPU
- Memory
- Disk
- Network
- Server health

---

## 2. Application Performance Monitoring (APM)

Tracks:
- Application latency
- Errors
- Response time
- API performance

---

## 3. Log Monitoring

Collects:
- Application logs
- System logs
- Kubernetes logs
- Docker logs

---

## 4. Container Monitoring

Supports:
- Docker
- Kubernetes
- ECS
- EKS

Tracks:
- Container health
- Pod metrics
- Node metrics

---

## 5. Cloud Monitoring

Supports:
- AWS
- Azure
- GCP

Monitors:
- EC2
- RDS
- Load Balancers
- Lambda

---

# 🔹 Datadog Architecture

```hcl
Application / Server / Kubernetes
                │
                ▼
        Datadog Agent
                │
                ▼
        Datadog Cloud Platform
                │
 ┌──────────────┼──────────────┐
 ▼              ▼              ▼
Metrics       Logs          Traces
 │              │              │
 ▼              ▼              ▼
Dashboard     Alerts       Analytics
```

---

# 🔹 Datadog Components

| Component | Purpose |
|---|---|
| Datadog Agent | Collect metrics & logs |
| Dashboard | Visual monitoring |
| Alert Manager | Send notifications |
| APM | Trace applications |
| Log Explorer | Analyze logs |
| Integrations | Connect cloud services |

---

# 🔹 How Datadog Works

1. Install Datadog Agent
2. Agent collects metrics/logs
3. Sends data to Datadog Cloud
4. Datadog analyzes data
5. Dashboards & alerts generated

---

# 🔹 Datadog Agent

Agent runs on:
- Linux servers
- Windows servers
- Kubernetes nodes
- Docker containers

Responsibilities:
- Collect metrics
- Forward logs
- Monitor processes
- Send traces

---

# 🔹 Datadog Integrations

| Platform | Supported |
|---|---|
| AWS | Yes |
| Azure | Yes |
| GCP | Yes |
| Kubernetes | Yes |
| Docker | Yes |
| Jenkins | Yes |
| Terraform | Yes |
| Nginx | Yes |

---

# 🔹 Datadog Dashboard

Dashboards help visualize:
- CPU usage
- Memory usage
- Network traffic
- Pod health
- Application latency

Benefits:
- Easy troubleshooting
- Real-time monitoring
- Better visibility

---

# 🔹 Datadog Alerts

Alerts can trigger when:
- CPU high
- Pod down
- Disk full
- API failing
- Kubernetes node unhealthy

Notification Channels:
- Email
- Slack
- PagerDuty
- Microsoft Teams

---

# 🔹 Datadog with Kubernetes

Datadog monitors:
- Pods
- Nodes
- Containers
- Deployments
- Cluster health

Common Metrics:
- Pod restarts
- CPU usage
- Memory usage
- Node pressure

---

# 🔹 Datadog with AWS

Datadog integrates with:
- EC2
- RDS
- Lambda
- CloudWatch
- EKS
- S3

---

# 🔹 Datadog vs Prometheus

| Feature | Datadog | Prometheus |
|---|---|---|
| Type | SaaS | Open-source |
| Setup | Easy | Complex |
| UI | Advanced | Basic |
| Alerting | Built-in | Needs Alertmanager |
| Scalability | High | Medium |
| Cost | Paid | Free |

---

# 🔹 Advantages of Datadog

- Easy setup
- Real-time monitoring
- Multi-cloud support
- Powerful dashboards
- Kubernetes integration
- Centralized observability

---

# 🔹 Limitations of Datadog

- Paid service
- Can become expensive
- Internet dependency
- Learning curve for advanced features

---

# 🔹 Common Datadog Commands

## Kubernetes Agent
```bash
kubectl get pods -n datadog
```

## Docker Agent
```bash
docker ps
```

## Linux Agent Status
```bash
sudo datadog-agent status
```

---

# 🔹 Real-Time DevOps Usage

Datadog is used for:
- Infrastructure monitoring
- Kubernetes monitoring
- CI/CD monitoring
- Cloud monitoring
- Incident management
- Application troubleshooting

---

# 🔹 Interview Questions

1. What is Datadog?
2. Explain Datadog architecture.
3. Difference between Datadog and Prometheus?
4. What is Datadog Agent?
5. How does Datadog monitor Kubernetes?
6. What are Datadog integrations?
7. Explain Datadog APM.
8. How are alerts configured in Datadog?
9. How does Datadog collect logs?
10. Why is Datadog used in DevOps?

---

# 🔹 Quick Memory Trick

👉 “Collect → Analyze → Visualize → Alert”

#Datadog #Monitoring #DevOps #Kubernetes #CloudComputing #Observability #APM #AWS
