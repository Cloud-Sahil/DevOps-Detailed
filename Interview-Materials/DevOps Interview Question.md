# DevOps Interview Questions & Answers Guide

## Table of Contents
1. Terraform Fundamentals  
2. Docker & Containerization  
3. Kubernetes  
4. CI/CD & Jenkins  
5. AWS & Cloud  
6. Linux & Networking  
7. Monitoring & Observability  
8. Scenario-Based Questions  
9. Architecture Diagrams  
10. Useful Commands & Syntax  

---

# 1. Terraform Fundamentals

## Q1. What is Infrastructure as Code (IaC)?
Infrastructure as Code (IaC) is the process of managing infrastructure using configuration files instead of manual setup.

### Benefits
| Feature | Benefit |
|---|---|
| Automation | Reduces manual work |
| Consistency | Same setup every time |
| Version Control | Track infrastructure changes |
| Scalability | Easy to scale resources |

---

## Q2. What is Terraform?
Terraform is an open-source IaC tool used to provision and manage cloud infrastructure.

### Terraform Workflow
```bash
terraform init
terraform plan
terraform apply
terraform destroy
```

---

## Q3. Explain Terraform State File.
Terraform stores infrastructure information inside:

```bash
terraform.tfstate
```

### Best Practice
- Store remotely using S3
- Enable state locking using DynamoDB

---

## Q4. Difference Between Count and for_each
| count | for_each |
|---|---|
| Uses index | Uses key-value |
| Good for identical resources | Good for unique resources |

### Example
```hcl
resource "aws_instance" "web" {
  count = 2
}
```

---

## Q5. What are Terraform Modules?
Modules are reusable Terraform configurations.

### Example
```hcl
module "vpc" {
  source = "./modules/vpc"
}
```

---

# 2. Docker & Containerization

## Q6. Difference Between Virtualization and Containerization
| Virtualization | Containerization |
|---|---|
| Uses Hypervisor | Uses Container Engine |
| Heavyweight | Lightweight |
| Separate OS | Shared Kernel |

---

## Q7. Explain Docker Architecture.
Docker architecture contains:
- Docker Client
- Docker Daemon
- Docker Registry
- Docker Images
- Docker Containers

---

## Q8. What is a Docker Image?
A Docker image is a read-only template used to create containers.

### Example
```bash
docker pull nginx
```

---

## Q9. What is the Difference Between CMD and ENTRYPOINT?
| CMD | ENTRYPOINT |
|---|---|
| Default command | Main executable |
| Easily overridden | Harder to override |

---

## Q10. Explain Docker Volumes.
Docker volumes are used for persistent storage.

```bash
docker volume create myvolume
```

---

## Q11. What is Docker Networking?
Docker networking allows communication between containers.

### Types
- Bridge
- Host
- Overlay
- None

---

## Q12. Explain Multi-stage Docker Build.
Used to reduce image size.

```dockerfile
FROM maven AS build
FROM openjdk
```

---

# 3. Kubernetes

## Q13. What is Kubernetes?
Kubernetes is a container orchestration platform.

---

## Q14. Explain Kubernetes Architecture.
### Components
- Master Node
- Worker Node
- API Server
- Scheduler
- etcd
- kubelet
- kube-proxy

---

## Q15. Difference Between Deployment and StatefulSet
| Deployment | StatefulSet |
|---|---|
| Stateless Apps | Stateful Apps |
| Random Pod Names | Fixed Pod Identity |

---

## Q16. What is a Pod?
Pod is the smallest deployable unit in Kubernetes.

---

## Q17. Explain Services in Kubernetes.
### Types
- ClusterIP
- NodePort
- LoadBalancer

---

## Q18. What is an Ingress Controller?
Ingress manages external HTTP/HTTPS access to services.

---

## Q19. What Happens When a Pod Fails?
Kubernetes automatically recreates the pod using ReplicaSet.

---

## Q20. Explain ConfigMap and Secret.
| ConfigMap | Secret |
|---|---|
| Non-sensitive data | Sensitive data |
| Plain text | Base64 encoded |

---

# 4. CI/CD & Jenkins

## Q21. What is CI/CD?
CI/CD automates software build, test, and deployment.

---

## Q22. Explain Jenkins Pipeline.
### Declarative Pipeline Example
```groovy
pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
    }
  }
}
```

---

## Q23. Difference Between Blue-Green and Canary Deployment
| Blue-Green | Canary |
|---|---|
| Full environment switch | Gradual traffic shift |
| Faster rollback | Lower risk |

---

## Q24. What is GitOps?
GitOps uses Git as the source of truth for deployments.

---

## Q25. Explain Webhooks in CI/CD.
Webhooks automatically trigger Jenkins pipelines when code changes happen in GitHub.

---

# 5. AWS & Cloud

## Q26. Difference Between EC2 and Lambda
| EC2 | Lambda |
|---|---|
| Server-based | Serverless |
| Manual scaling | Auto scaling |

---

## Q27. Explain VPC.
VPC is a logically isolated virtual network in AWS.

### Components
- Subnet
- Route Table
- Internet Gateway
- NAT Gateway

---

## Q28. Difference Between ALB and NLB
| ALB | NLB |
|---|---|
| Layer 7 | Layer 4 |
| HTTP/HTTPS | TCP/UDP |

---

## Q29. What is Auto Scaling?
Automatically increases or decreases resources based on traffic.

---

## Q30. Explain IAM Roles and Policies.
IAM controls AWS access permissions.

---

# 6. Linux & Networking

## Q31. Useful Linux Commands

```bash
top
htop
df -h
free -m
netstat -tulpn
ss -tulpn
ps aux
```

---

## Q32. What is DNS?
DNS converts domain names into IP addresses.

---

## Q33. Difference Between TCP and UDP
| TCP | UDP |
|---|---|
| Reliable | Faster |
| Connection-oriented | Connectionless |

---

## Q34. Explain Load Balancer.
A load balancer distributes traffic across servers.

---

# 7. Monitoring & Observability

## Q35. Difference Between Monitoring and Observability
| Monitoring | Observability |
|---|---|
| Detects problems | Explains problems |

---

## Q36. Tools Used for Monitoring
| Tool | Purpose |
|---|---|
| Prometheus | Metrics |
| Grafana | Visualization |
| ELK Stack | Logging |

---

## Q37. What is Alerting?
Alerting notifies teams when thresholds are crossed.

---

# 8. Scenario-Based Questions

## Q38. Application Suddenly Goes Down in Kubernetes. What Will You Check?
### Steps
1. Check pod status
2. View logs
3. Check events
4. Verify node health
5. Check resource usage

```bash
kubectl get pods
kubectl logs pod-name
kubectl describe pod pod-name
```

---

## Q39. Jenkins Pipeline Fails After Deployment. How Will You Troubleshoot?
### Troubleshooting
- Check Jenkins logs
- Verify credentials
- Check artifact version
- Verify deployment server connectivity

---

## Q40. Website is Slow in Production. What Will You Do?
### Investigation
- CPU & Memory usage
- Database latency
- Network bottlenecks
- Load balancer health
- Application logs

---

## Q41. Terraform Apply Failed Midway. What Happens?
Terraform state may become partially updated.  
Use:
```bash
terraform refresh
terraform plan
```

---

## Q42. Docker Container Keeps Restarting. What Could Be the Reason?
### Possible Reasons
- Application crash
- Port conflict
- Missing environment variables
- Insufficient memory

---

## Q43. How Will You Secure Kubernetes Cluster?
### Security Best Practices
- RBAC
- Network Policies
- Pod Security Policies
- Secrets Management
- Image Scanning

---

## Q44. A Production Server Disk is Full. What Will You Do?
```bash
df -h
du -sh /*
journalctl --vacuum-time=7d
```

---

## Q45. How Will You Design Highly Available Architecture?
### Components
- Load Balancer
- Multiple Availability Zones
- Auto Scaling
- Database Replication
- Monitoring

---

# 9. Architecture Diagrams

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/2a18c5d3-9e79-4436-9dc3-fdee74fe39a5" />

---

# 10. Useful Commands & Syntax

## Docker Commands
```bash
docker ps
docker images
docker logs container-id
docker exec -it container-id bash
```

---

## Kubernetes Commands
```bash
kubectl get pods
kubectl get svc
kubectl describe pod pod-name
kubectl logs pod-name
```

---

## Terraform Commands
```bash
terraform init
terraform validate
terraform plan
terraform apply
terraform destroy
```

---

## Jenkins Commands
```bash
systemctl status jenkins
journalctl -u jenkins
```

---

# Final Tips

## Important Interview Topics
- Kubernetes Troubleshooting
- CI/CD Pipelines
- Terraform State Management
- Docker Networking
- AWS Architecture
- Monitoring & Logging

## Interview Preparation Tips
1. Practice hands-on labs
2. Create projects
3. Learn troubleshooting
4. Understand architecture
5. Practice scenario-based questions

---
