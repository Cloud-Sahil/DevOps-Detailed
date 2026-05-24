# <img src="https://cdn.simpleicons.org/docker" width="40" /> Introduction to Docker
# Docker Complete Notes

# Table of Contents
1. Introduction to Docker
2. Why Docker?
3. Docker Architecture
4. Docker Components
5. Docker Installation
6. Docker Images
7. Docker Containers
8. Dockerfile
9. Docker Networking
10. Docker Volumes
11. Docker Compose
12. Docker Registry
13. Docker Commands
14. Docker Swarm
15. Docker Security
16. Docker Best Practices
17. Docker Lifecycle
18. Docker Troubleshooting
19. Real-Time Docker Scenarios
20. Useful Diagrams

---

# 1. Introduction to Docker

Docker is an open-source containerization platform used to build, package, deploy, and run applications inside lightweight containers.

## Key Features
- Lightweight
- Portable
- Fast deployment
- Consistent environments
- Isolation
- Scalability

---

# 2. Why Docker?

## Problems Before Docker
- "Works on my machine" issue
- Dependency conflicts
- Slow deployment
- Environment mismatch

## Benefits
- Same environment everywhere
- Faster CI/CD
- Better resource utilization
- Easy rollback
- Easy scaling

---

# 3. Docker Architecture

```text
+-----------------------+
|      Docker CLI       |
+-----------+-----------+
            |
            v
+-----------------------+
|    Docker Daemon      |
|      (dockerd)        |
+-----------+-----------+
            |
   -------------------
   |        |        |
   v        v        v
Images   Containers Volumes
            |
            v
      Docker Engine
```

## Components
- Docker Client
- Docker Daemon
- Docker Registry
- Docker Engine
- Images
- Containers

---

# 4. Docker Components

## Docker Client
Used to interact with Docker.

```bash
docker run nginx
```

## Docker Daemon
Runs in background and manages containers.

## Docker Registry
Stores Docker images.

Examples:
- Docker Hub
- AWS ECR
- GCR

## Docker Images
Blueprint/template for containers.

## Docker Containers
Running instance of Docker image.

---

# 5. Docker Installation

## Ubuntu

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

## Verify Installation

```bash
docker --version
docker run hello-world
```

---

# 6. Docker Images

## Pull Image

```bash
docker pull nginx
```

## List Images

```bash
docker images
```

## Remove Image

```bash
docker rmi nginx
```

## Build Image

```bash
docker build -t myapp .
```

---

# 7. Docker Containers

## Run Container

```bash
docker run -d -p 80:80 nginx
```

## List Containers

```bash
docker ps
docker ps -a
```

## Stop Container

```bash
docker stop <container_id>
```

## Remove Container

```bash
docker rm <container_id>
```

---

# 8. Dockerfile

Dockerfile is used to create custom Docker images.

## Example Dockerfile

```dockerfile
FROM ubuntu:22.04

RUN apt update && apt install nginx -y

COPY . /app

WORKDIR /app

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

## Important Instructions

| Instruction | Purpose |
|---|---|
| FROM | Base image |
| RUN | Execute command |
| COPY | Copy files |
| ADD | Copy + extract |
| WORKDIR | Set working directory |
| CMD | Default command |
| ENTRYPOINT | Main executable |
| ENV | Environment variables |
| EXPOSE | Open port |

---

# 9. Docker Networking

## Network Types

### Bridge Network
Default network.

### Host Network
Container shares host network.

### None Network
No networking.

### Overlay Network
Used in Swarm.

## Commands

```bash
docker network ls
docker network create mynet
docker network inspect mynet
```

---

# 10. Docker Volumes

Used for persistent storage.

## Create Volume

```bash
docker volume create myvolume
```

## Attach Volume

```bash
docker run -v myvolume:/data nginx
```

## Bind Mount

```bash
docker run -v /host/path:/container/path nginx
```

---

# 11. Docker Compose

Used to manage multi-container applications.

## Example docker-compose.yml

```yaml
version: '3'

services:
  web:
    image: nginx
    ports:
      - "80:80"

  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
```

## Commands

```bash
docker compose up -d
docker compose down
docker compose ps
```

---

# 12. Docker Registry

## Push Image

```bash
docker login
docker tag myapp username/myapp:v1
docker push username/myapp:v1
```

## Pull Image

```bash
docker pull username/myapp:v1
```

---

# 13. Docker Commands

## Important Commands

```bash
docker images
docker ps
docker exec -it container bash
docker logs container
docker inspect container
docker stats
docker top container
docker cp
docker history
docker info
```

---

# 14. Docker Swarm

Docker's native orchestration tool.

## Initialize Swarm

```bash
docker swarm init
```

## Create Service

```bash
docker service create --name web nginx
```

## Scale Service

```bash
docker service scale web=5
```

---

# 15. Docker Security

## Best Practices
- Use official images
- Avoid root user
- Scan images
- Use minimal images
- Keep images updated
- Limit container privileges

## Image Scanning

```bash
docker scan nginx
```

---

# 16. Docker Best Practices

- Use multi-stage builds
- Reduce image size
- Use .dockerignore
- Avoid latest tag
- One process per container
- Use health checks
- Store secrets securely

---

# 17. Docker Lifecycle

```text
Dockerfile
    ↓
Docker Image
    ↓
Docker Container
    ↓
Run / Stop / Restart
```

---

# 18. Docker Troubleshooting

## Container Exits Immediately
Reason:
- Main process finished

Solution:
```bash
docker logs container
```

## Port Already Used
Solution:
```bash
sudo netstat -tulnp
```

## Permission Denied

```bash
sudo usermod -aG docker $USER
```

---

# 19. Real-Time Docker Scenarios

## Scenario 1: Deploy Nginx Website

```bash
docker run -d -p 80:80 nginx
```

## Scenario 2: Run MySQL Database

```bash
docker run -d \
--name mysql \
-e MYSQL_ROOT_PASSWORD=root \
-p 3306:3306 mysql
```

## Scenario 3: Connect Two Containers

```bash
docker network create appnet

docker run -d --name db --network appnet mysql

docker run -d --name backend --network appnet node
```

---

# 20. Useful Diagrams

## Docker Workflow

```text
Developer
   ↓
Dockerfile
   ↓
Docker Build
   ↓
Docker Image
   ↓
Docker Registry
   ↓
Docker Run
   ↓
Container
```

## Container vs VM

```text
Virtual Machine:
App + OS + Hypervisor

Container:
App + Libraries + Docker Engine
```

---

# Conclusion

Docker simplifies application deployment using containerization. It is widely used in DevOps, Cloud, Kubernetes, CI/CD, and Microservices architecture.



