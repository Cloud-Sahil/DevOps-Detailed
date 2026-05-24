# Docker Interview Questions and Answers

# Table of Contents
1. Basic Questions
2. Intermediate Questions
3. Advanced Questions
4. Scenario-Based Questions
5. Dockerfile Questions
6. Docker Compose Questions
7. Networking Questions
8. Security Questions
9. Troubleshooting Questions
10. Architecture Questions
11. Real-Time Production Questions

---

# 1. Basic Questions

## What is Docker?
Docker is a containerization platform used to package applications and dependencies together.

## Difference Between Container and VM

| Container | VM |
|---|---|
| Lightweight | Heavy |
| Shares host OS | Separate OS |
| Fast startup | Slow startup |
| Less resource usage | More resource usage |

## What is Docker Image?
A read-only template used to create containers.

## What is Docker Container?
A running instance of a Docker image.

## What is Docker Hub?
A public registry for Docker images.

---

# 2. Intermediate Questions

## Difference Between CMD and ENTRYPOINT

| CMD | ENTRYPOINT |
|---|---|
| Default command | Main executable |
| Can be overridden | Harder to override |

## What is Docker Volume?
Used for persistent storage.

## What is Docker Compose?
Tool for managing multi-container applications.

## Explain Docker Layers
Each Dockerfile instruction creates a layer.

---

# 3. Advanced Questions

## Explain Docker Architecture

```text
Docker CLI
    ↓
Docker Daemon
    ↓
Containers / Images / Networks
```

## What Happens When You Run `docker run nginx`?

1. Checks image locally
2. Pulls from registry if missing
3. Creates container
4. Starts container
5. Attaches networking

## Explain Multi-Stage Build

Used to reduce image size.

Example:

```dockerfile
FROM maven AS build
COPY . .
RUN mvn package

FROM openjdk
COPY --from=build target/app.jar app.jar
```

---

# 4. Scenario-Based Questions

## Scenario 1

### Question
Container is exiting immediately after startup. What will you do?

### Answer
- Check logs
- Verify CMD/ENTRYPOINT
- Ensure main process is running

Commands:

```bash
docker logs container
docker inspect container
```

---

## Scenario 2

### Question
Application works locally but not inside Docker container.

### Answer
Possible issues:
- Wrong port mapping
- Missing dependencies
- Incorrect environment variables
- Network issue

---

## Scenario 3

### Question
How will you reduce Docker image size?

### Answer
- Use Alpine image
- Multi-stage builds
- Remove unnecessary packages
- Use .dockerignore

---

## Scenario 4

### Question
How will you connect frontend and backend containers?

### Answer

```bash
docker network create appnet
```

Run containers in same network.

---

## Scenario 5

### Question
Database data is lost after container restart.

### Answer
Use Docker volumes.

```bash
docker volume create mysql-data
```

---

# 5. Dockerfile Questions

## Difference Between COPY and ADD

| COPY | ADD |
|---|---|
| Simple copy | Copy + extract tar |
| Preferred | Less preferred |

## Difference Between RUN, CMD, ENTRYPOINT

| RUN | CMD | ENTRYPOINT |
|---|---|---|
| Build time | Runtime | Runtime executable |

## Why Use `.dockerignore`?
Reduces build context and improves performance.

---

# 6. Docker Compose Questions

## Why Docker Compose?
Manage multiple containers easily.

## Command to Start Services

```bash
docker compose up -d
```

## Command to Stop Services

```bash
docker compose down
```

---

# 7. Networking Questions

## Types of Docker Networks

- Bridge
- Host
- None
- Overlay

## Difference Between EXPOSE and -p

| EXPOSE | -p |
|---|---|
| Documentation | Port mapping |

---

# 8. Security Questions

## How Will You Secure Docker Containers?

- Use minimal images
- Scan vulnerabilities
- Avoid root user
- Limit privileges
- Use secrets management

## What is Docker Content Trust?
Ensures image authenticity.

---

# 9. Troubleshooting Questions

## How to Check Container Logs?

```bash
docker logs container
```

## How to Enter Running Container?

```bash
docker exec -it container bash
```

## Port Already in Use Error

```bash
sudo lsof -i :80
```

---

# 10. Architecture Questions

## Docker Architecture Diagram

```text
+-------------------+
| Docker Client     |
+---------+---------+
          |
          v
+-------------------+
| Docker Daemon     |
+---------+---------+
          |
    -----------------
    |       |       |
    v       v       v
 Images Containers Networks
```

## Explain Docker Engine
Core component responsible for building and running containers.

---

# 11. Real-Time Production Questions

## Question
How do companies deploy applications using Docker?

### Answer
- Build image
- Push to registry
- Pull image in server
- Deploy using Kubernetes or Docker Swarm

---

## Question
How do you monitor Docker containers?

### Answer
Tools:
- Prometheus
- Grafana
- cAdvisor
- ELK Stack

---

## Question
How will you perform zero downtime deployment?

### Answer
- Blue-Green deployment
- Rolling updates
- Load balancer switching

---

## Question
What is the difference between Docker Swarm and Kubernetes?

| Docker Swarm | Kubernetes |
|---|---|
| Simple | Complex |
| Easy setup | Enterprise-grade |
| Less features | More features |

---

# Important Docker Commands for Interviews

```bash
docker ps
docker images
docker logs
docker inspect
docker stats
docker exec
docker build
docker pull
docker push
docker network ls
docker volume ls
```

---

# Final Tips for Docker Interviews

- Practice Docker commands daily
- Build projects
- Understand networking and volumes
- Learn Docker Compose properly
- Know troubleshooting steps
- Understand architecture diagrams
- Learn real-world scenarios
