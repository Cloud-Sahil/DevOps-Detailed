
# 🐳 Docker — Complete Guide

---

## 📁 Table of Contents

| # | Section | Topics |
|---|---------|--------|
| 1 | [What is Docker](#1-what-is-docker) | Overview, VM vs Docker, Architecture |
| 2 | [Docker Architecture](#2-docker-architecture) | Client, Daemon, Registry, Objects |
| 3 | [Dockerfile](#3-dockerfile) | Instructions, multi-stage, best practices |
| 4 | [Images](#4-images) | Build, tag, push, layers, registry |
| 5 | [Containers](#5-containers) | Run, manage, lifecycle |
| 6 | [Networking](#6-networking) | Bridge, host, overlay, none |
| 7 | [Volumes & Storage](#7-volumes--storage) | Volumes, bind mounts, tmpfs |
| 8 | [Docker Compose](#8-docker-compose) | Multi-container apps, services, YAML |
| 9 | [Docker Swarm](#9-docker-swarm) | Clustering, services, stacks |
| 10 | [Security](#10-security) | Rootless, secrets, scanning, hardening |
| 11 | [Registry](#11-registry) | Docker Hub, ECR, private registry |
| 12 | [Monitoring & Logging](#12-monitoring--logging) | Stats, logs, Prometheus |
| 13 | [Best Practices](#13-best-practices) | Production-ready Docker |
| 14 | [Quick Reference](#14-quick-reference) | All commands cheatsheet |

---

## 1. What is Docker

### What is Docker?

Docker is an open-source platform that uses **OS-level virtualization** to deliver software in containers. Containers bundle app code + dependencies + runtime into a portable, isolated unit.

```
Without Docker:  "Works on my machine" → broken in production
With Docker:     Same container runs identically everywhere
```

**Why Docker?**

| Benefit | Description |
|---------|-------------|
| **Portability** | Build once, run anywhere (dev, staging, prod) |
| **Isolation** | Each container has its own filesystem, network, process space |
| **Speed** | Starts in milliseconds vs minutes for VMs |
| **Efficiency** | Shares host OS kernel → much lighter than VMs |
| **Reproducibility** | Same image = same environment every time |
| **Scalability** | Easy horizontal scaling with Swarm/Kubernetes |

---

### VM vs Docker vs Bare Metal

```
┌─────────────────────────────────────────────────────────────┐
│  BARE METAL        VM                  DOCKER               │
│                                                             │
│  ┌──────────┐   ┌────────────────┐   ┌──────────────────┐  │
│  │ App A    │   │ App A  │ App B │   │ App A  │ App B    │  │
│  │ App B    │   ├────────┼───────┤   ├────────┼──────────┤  │
│  │ App C    │   │Runtime │Runtime│   │Runtime │ Runtime  │  │
│  ├──────────┤   ├────────┴───────┤   ├────────────────────┤  │
│  │  OS      │   │  Guest OS      │   │  Docker Engine     │  │
│  ├──────────┤   ├────────────────┤   ├────────────────────┤  │
│  │ Hardware │   │  Hypervisor    │   │  Host OS           │  │
│  └──────────┘   ├────────────────┤   ├────────────────────┤  │
│                 │  Host OS       │   │  Hardware          │  │
│                 ├────────────────┤   └────────────────────┘  │
│                 │  Hardware      │                            │
│                 └────────────────┘                           │
└─────────────────────────────────────────────────────────────┘
```

| Feature | Bare Metal | VM | Docker |
|---------|-----------|-----|--------|
| Startup time | Minutes | Minutes | Milliseconds |
| Size | Full OS | GBs | MBs |
| OS | Single | Multiple (separate) | Shared kernel |
| Isolation | None | Strong | Good (process-level) |
| Performance | Best | Good | Near-native |
| Portability | Low | Medium | High |

---

## 2. Docker Architecture

### Client-Server Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    DOCKER ARCHITECTURE                  │
│                                                         │
│  ┌─────────────┐    REST API    ┌──────────────────┐    │
│  │ Docker CLI  │◄──────────────►│  Docker Daemon   │    │
│  │  (Client)   │                │  (dockerd)       │    │
│  └─────────────┘                │                  │    │
│                                 │  ┌────────────┐  │    │
│  ┌─────────────┐                │  │ containerd │  │    │
│  │  Dockerfile │                │  └─────┬──────┘  │    │
│  └─────────────┘                │        │         │    │
│                                 │  ┌─────▼──────┐  │    │
│                                 │  │   runc     │  │    │
│                                 │  └────────────┘  │    │
│                                 └──────────────────┘    │
│                                          │               │
│                                          ▼               │
│                              ┌───────────────────────┐   │
│                              │   Docker Objects       │   │
│                              │  Images │ Containers  │   │
│                              │  Volumes│ Networks    │   │
│                              └───────────────────────┘   │
│                                          │               │
│                                          ▼               │
│                              ┌───────────────────────┐   │
│                              │   Docker Registry     │   │
│                              │   (Docker Hub / ECR)  │   │
│                              └───────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Docker Components

| Component | Description |
|-----------|-------------|
| **Docker Client** | CLI (`docker`) — sends commands to daemon via REST API |
| **Docker Daemon** (`dockerd`) | Background service managing images, containers, networks, volumes |
| **containerd** | Container runtime — manages container lifecycle |
| **runc** | Low-level container runtime — creates containers using Linux namespaces + cgroups |
| **Docker Registry** | Store for Docker images (Docker Hub, ECR, GHCR, private) |

### How `docker run` works internally

```
docker run nginx
     │
     ▼
Docker Client → REST API → Docker Daemon
                                │
                         Image exists locally?
                         ├── YES → use it
                         └── NO → pull from registry
                                │
                         Create container (containerd)
                                │
                         Create process (runc)
                                │
                         Apply Linux namespaces:
                         ├── PID namespace (isolated processes)
                         ├── NET namespace (isolated network)
                         ├── MNT namespace (isolated filesystem)
                         ├── UTS namespace (isolated hostname)
                         └── IPC namespace (isolated IPC)
                                │
                         Apply cgroups:
                         ├── CPU limits
                         ├── Memory limits
                         └── I/O limits
                                │
                         Container running ✅
```

---

## 3. Dockerfile

### All Dockerfile Instructions

```dockerfile
# ============================================================
# COMPLETE DOCKERFILE REFERENCE
# ============================================================

# Base image (ALWAYS first instruction)
FROM ubuntu:22.04

# Set environment variables (available at build + runtime)
ENV APP_HOME=/app \
    PORT=8080 \
    NODE_ENV=production

# Build-time arguments (only available during build)
ARG BUILD_VERSION=1.0.0
ARG GITHUB_TOKEN

# Set working directory (creates if not exists)
WORKDIR $APP_HOME

# Copy files from build context to container
COPY package.json package-lock.json ./
COPY src/ ./src/
COPY --chown=node:node . .     # Copy with ownership

# Add (like COPY but supports URLs and auto-extracts archives)
ADD https://example.com/file.tar.gz /tmp/

# Run commands during build
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      curl \
      ca-certificates && \
    rm -rf /var/lib/apt/lists/*    # Clean up in same layer!

# Create user (security: don't run as root)
RUN groupadd -r appgroup && \
    useradd -r -g appgroup -m appuser

# Switch to non-root user
USER appuser

# Expose port (documentation only — doesn't actually publish)
EXPOSE 8080

# Mount point for volumes
VOLUME ["/app/data", "/app/logs"]

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# Default command (can be overridden by docker run)
CMD ["node", "server.js"]

# OR: Entry point (cannot be overridden without --entrypoint)
ENTRYPOINT ["node"]
CMD ["server.js"]    # Arguments to ENTRYPOINT

# Labels (metadata)
LABEL maintainer="sahil@company.com" \
      version="1.0.0" \
      description="Node.js application"

# Shell form vs Exec form
# Shell form (runs via /bin/sh -c): CMD node server.js
# Exec form (no shell): CMD ["node", "server.js"]  ← PREFERRED
```

---

### Multi-Stage Build (Production Best Practice)

```dockerfile
# ============================================================
# MULTI-STAGE BUILD — Java Spring Boot
# ============================================================

# Stage 1: Build
FROM maven:3.9.6-eclipse-temurin-21 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline    # Cache dependencies
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime (much smaller!)
FROM eclipse-temurin:21-jre-alpine AS runtime
WORKDIR /app

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Copy ONLY the JAR from build stage
COPY --from=builder --chown=appuser:appgroup \
    /app/target/*.jar app.jar

EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=5s \
    CMD wget -qO- http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java"]
CMD ["-jar", "app.jar"]
```

```dockerfile
# ============================================================
# MULTI-STAGE BUILD — Node.js React App
# ============================================================

# Stage 1: Install dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 3: Production server (nginx)
FROM nginx:1.25-alpine AS production
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

### .dockerignore (Always create!)

```
# .dockerignore
node_modules/
npm-debug.log
.git
.gitignore
*.md
*.log
target/
.env
.env.*
Dockerfile*
docker-compose*
.DS_Store
coverage/
.nyc_output/
dist/
build/
*.test.js
__tests__/
```

---

### Image Layer Optimization

```dockerfile
# ❌ BAD — each RUN = new layer, not cached efficiently
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN rm -rf /var/lib/apt/lists/*

# ✅ GOOD — single layer, proper cleanup
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      curl \
      wget && \
    rm -rf /var/lib/apt/lists/*

# ❌ BAD — COPY . . before npm install (cache busted by every change)
COPY . .
RUN npm install

# ✅ GOOD — copy package.json first (npm install cached until package.json changes)
COPY package*.json ./
RUN npm ci
COPY . .
```

---

## 4. Images

### Image Architecture

```
Docker Image = Read-only Stack of Layers

  ┌──────────────────────────────┐
  │ Layer 4: COPY app code       │ ← Your app code
  ├──────────────────────────────┤
  │ Layer 3: RUN npm install     │ ← Dependencies
  ├──────────────────────────────┤
  │ Layer 2: COPY package.json   │ ← Package file
  ├──────────────────────────────┤
  │ Layer 1: RUN apt-get update  │ ← System packages
  ├──────────────────────────────┤
  │ Layer 0: FROM ubuntu:22.04   │ ← Base image
  └──────────────────────────────┘

When you run a container:
  ┌──────────────────────────────┐
  │ Container Layer (R/W)        │ ← Writable layer (lost on stop)
  ├──────────────────────────────┤
  │ Image Layers (Read-Only)     │ ← Shared across containers
  └──────────────────────────────┘
```

### Image Commands

```bash
#!/bin/bash
# ── BUILD ────────────────────────────────────────────────
docker build -t myapp:v1 .
docker build -t myapp:v1 -f Dockerfile.prod .    # Custom Dockerfile
docker build --no-cache -t myapp:latest .         # No cache
docker build --build-arg ENV=prod -t myapp .      # Build args
docker build --target builder -t myapp:builder .  # Specific stage

# ── TAG ──────────────────────────────────────────────────
docker tag myapp:v1 myregistry.io/myapp:v1
docker tag myapp:v1 myapp:latest
docker tag myapp:v1 sahil0117/myapp:v1

# ── PUSH/PULL ────────────────────────────────────────────
docker push sahil0117/myapp:v1
docker pull nginx:1.25-alpine
docker pull --all-tags myapp              # Pull all tags

# ── INSPECT ──────────────────────────────────────────────
docker images                             # List images
docker images -a                          # All including intermediate
docker image ls --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
docker image inspect myapp:v1             # Full metadata
docker image history myapp:v1             # Layer history
docker image inspect --format='{{.Config.Cmd}}' myapp:v1

# ── SAVE/LOAD (offline transfer) ─────────────────────────
docker save myapp:v1 | gzip > myapp.tar.gz
docker load < myapp.tar.gz

# ── CLEANUP ──────────────────────────────────────────────
docker rmi myapp:v1                       # Remove image
docker rmi $(docker images -q)            # Remove all images
docker image prune                        # Remove dangling images
docker image prune -a                     # Remove all unused images
docker system prune -a --volumes          # Full cleanup (CAREFUL!)

# ── SEARCH ───────────────────────────────────────────────
docker search nginx
docker search --filter=stars=1000 nginx
```

---

## 5. Containers

### Container Lifecycle

```
Created → Running → Paused → Running → Stopped → Deleted
   │                                       │
   └─────────── Restarted ─────────────────┘
```

### Container Commands

```bash
#!/bin/bash
# ── RUN ──────────────────────────────────────────────────
docker run nginx                          # Foreground (blocks terminal)
docker run -d nginx                       # Detached (background)
docker run -it ubuntu bash                # Interactive + TTY
docker run --name my-nginx nginx          # Named container
docker run -p 8080:80 nginx               # Port mapping host:container
docker run -p 127.0.0.1:8080:80 nginx    # Bind to specific host IP
docker run -e DB_HOST=localhost nginx     # Environment variable
docker run --env-file .env nginx          # Env from file
docker run -v /host/path:/container/path nginx   # Bind mount
docker run -v myvolume:/data nginx        # Named volume
docker run --rm nginx                     # Auto-remove when stopped
docker run --restart=always nginx         # Restart policy
docker run --restart=on-failure:3 nginx  # Restart on failure max 3 times
docker run --memory=512m nginx            # Memory limit
docker run --cpus=1.5 nginx              # CPU limit
docker run --network my-network nginx     # Connect to network
docker run --user 1000:1000 nginx        # Run as specific user
docker run --read-only nginx             # Read-only filesystem
docker run --security-opt=no-new-privileges nginx  # Security

# ── MANAGE ───────────────────────────────────────────────
docker ps                                 # Running containers
docker ps -a                              # All containers (including stopped)
docker ps -q                              # Only IDs
docker start my-container
docker stop my-container                  # Graceful (SIGTERM + timeout)
docker stop -t 30 my-container           # 30s grace period
docker kill my-container                  # Immediate (SIGKILL)
docker restart my-container
docker pause my-container
docker unpause my-container
docker rm my-container                    # Remove stopped container
docker rm -f my-container                 # Force remove running container
docker rm $(docker ps -aq)               # Remove all containers

# ── INSPECT/DEBUG ────────────────────────────────────────
docker logs my-container
docker logs -f my-container              # Follow logs
docker logs --tail=50 my-container       # Last 50 lines
docker logs --since 1h my-container      # Last 1 hour
docker exec -it my-container bash        # Shell into running container
docker exec my-container env             # Run single command
docker inspect my-container              # Full metadata JSON
docker stats my-container                # Live resource usage
docker stats --no-stream                 # Single snapshot
docker top my-container                  # Running processes
docker port my-container                 # Port mappings
docker cp my-container:/path/to/file ./  # Copy file from container
docker cp ./file my-container:/path/     # Copy file to container
docker diff my-container                 # Changed files
docker events                            # Real-time Docker events

# ── RESTART POLICIES ─────────────────────────────────────
# no          — Never restart (default)
# always      — Always restart
# on-failure  — Restart on non-zero exit
# unless-stopped — Restart unless manually stopped
```

---

## 6. Networking

### Docker Network Types

```
┌─────────────────────────────────────────────────────────┐
│                  DOCKER NETWORKING                      │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │  HOST NETWORK                                    │   │
│  │  Container uses host's network stack directly   │   │
│  │  No isolation — port 80 = host port 80           │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  ┌────────────────────┐  ┌────────────────────────┐     │
│  │  BRIDGE (default)  │  │  CUSTOM BRIDGE         │     │
│  │                    │  │                        │     │
│  │  Container A       │  │  Container A           │     │
│  │  172.17.0.2        │  │  Resolved by name ✅   │     │
│  │      │             │  │      │                 │     │
│  │  docker0 bridge    │  │  my-network bridge     │     │
│  │  172.17.0.1        │  │                        │     │
│  │      │             │  │  Container B           │     │
│  │    HOST            │  │  Resolved by name ✅   │     │
│  └────────────────────┘  └────────────────────────┘     │
│                                                         │
│  ┌────────────────────┐  ┌────────────────────────┐     │
│  │  NONE              │  │  OVERLAY (Swarm/K8s)   │     │
│  │                    │  │                        │     │
│  │  No network access │  │  Multi-host networking │     │
│  │  Fully isolated    │  │  Connects across nodes │     │
│  └────────────────────┘  └────────────────────────┘     │
└─────────────────────────────────────────────────────────┘
```

### Network Types

| Type | Description | Use Case |
|------|-------------|---------|
| `bridge` | Default — isolated network, NAT to host | Single-host apps |
| `host` | Container uses host network directly | High performance, low latency |
| `none` | No network interface | Fully isolated jobs |
| `overlay` | Multi-host networking (Swarm) | Distributed apps |
| `macvlan` | Container gets its own MAC/IP | Legacy app integration |

### Network Commands

```bash
#!/bin/bash
# ── CREATE/MANAGE ────────────────────────────────────────
docker network create my-network
docker network create --driver bridge my-bridge
docker network create --driver overlay --attachable my-overlay
docker network create \
  --subnet 192.168.100.0/24 \
  --gateway 192.168.100.1 \
  my-custom-net

# ── CONNECT/DISCONNECT ───────────────────────────────────
docker run -d --network my-network --name app nginx
docker network connect my-network existing-container
docker network disconnect my-network container-name

# ── INSPECT ──────────────────────────────────────────────
docker network ls                         # List networks
docker network inspect my-network         # Details
docker network inspect bridge             # Default bridge info

# ── DNS RESOLUTION (Custom Bridge only) ──────────────────
# docker run --network my-network --name app1 nginx
# docker run --network my-network --name app2 nginx
# Inside app2: curl http://app1  ← Works! (by container name)
# Default bridge: must use IP, no DNS

# ── CLEANUP ──────────────────────────────────────────────
docker network rm my-network
docker network prune                      # Remove unused networks
```

---

## 7. Volumes & Storage

### Storage Types

```
┌─────────────────────────────────────────────────────────┐
│                  DOCKER STORAGE                         │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │  VOLUMES (Preferred for persistent data)         │   │
│  │  Managed by Docker                               │   │
│  │  Location: /var/lib/docker/volumes/              │   │
│  │  Works on all platforms                          │   │
│  │  Can be shared between containers                │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │  BIND MOUNTS (Dev use / specific host path)      │   │
│  │  Uses exact host filesystem path                 │   │
│  │  Host changes instantly visible in container     │   │
│  │  Less portable                                   │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │  TMPFS MOUNTS (Temporary in-memory storage)      │   │
│  │  Data lost when container stops                  │   │
│  │  Good for sensitive data (passwords, tokens)     │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Volume Commands

```bash
#!/bin/bash
# ── VOLUMES ──────────────────────────────────────────────
docker volume create my-volume
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.1,rw \
  --opt device=:/path/to/dir \
  nfs-volume

docker volume ls
docker volume inspect my-volume
docker volume rm my-volume
docker volume prune                       # Remove unused volumes

# Use in container
docker run -d \
  -v my-volume:/app/data \               # Named volume
  -v /host/path:/container/path \        # Bind mount
  -v /host/path:/container/path:ro \     # Read-only bind mount
  --tmpfs /tmp \                         # tmpfs mount
  my-app

# ── BACKUP VOLUME DATA ───────────────────────────────────
# Backup
docker run --rm \
  -v my-volume:/data \
  -v $(pwd):/backup \
  ubuntu tar czf /backup/volume-backup.tar.gz /data

# Restore
docker run --rm \
  -v my-volume:/data \
  -v $(pwd):/backup \
  ubuntu tar xzf /backup/volume-backup.tar.gz -C /

# ── COMPARISON ───────────────────────────────────────────
# Volume:     docker run -v my-vol:/app/data nginx
# Bind mount: docker run -v /host/dir:/app/data nginx
# Read-only:  docker run -v my-vol:/app/data:ro nginx
```

**Volumes vs Bind Mounts:**

| Feature | Volumes | Bind Mounts |
|---------|---------|-------------|
| Managed by | Docker | OS (user) |
| Location | `/var/lib/docker/volumes/` | Any host path |
| Portability | High | Low |
| Performance | Good | Good (Linux) / Slow (Mac/Win) |
| Use case | Production data | Dev hot-reload |
| Backup | Easy | Manual |

---

## 8. Docker Compose

### What is Docker Compose?

Docker Compose defines and runs **multi-container applications** using a single YAML file. All containers, networks, and volumes declared in one place.

### Complete `docker-compose.yml` Reference

```yaml
# docker-compose.yml — Complete production-ready example
version: "3.9"

services:

  # ── FRONTEND ──────────────────────────────────────────
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      target: production          # Multi-stage target
      args:
        NODE_ENV: production
    image: myapp-frontend:latest
    container_name: myapp-frontend
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    networks:
      - frontend-net
    depends_on:
      backend:
        condition: service_healthy   # Wait for healthy check
    environment:
      - REACT_APP_API_URL=http://backend:8081
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 256M
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.frontend.rule=Host(`myapp.com`)"

  # ── BACKEND ───────────────────────────────────────────
  backend:
    build: ./backend
    image: myapp-backend:latest
    container_name: myapp-backend
    restart: unless-stopped
    ports:
      - "8081:8081"
    networks:
      - frontend-net
      - backend-net
    depends_on:
      database:
        condition: service_healthy
      redis:
        condition: service_started
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://database:3306/mydb
      - SPRING_DATASOURCE_USERNAME=${DB_USER}
      - SPRING_DATASOURCE_PASSWORD=${DB_PASSWORD}
      - REDIS_HOST=redis
    env_file:
      - .env.production
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8081/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 60s
    volumes:
      - logs-volume:/app/logs

  # ── DATABASE ──────────────────────────────────────────
  database:
    image: mysql:8.0
    container_name: myapp-mysql
    restart: unless-stopped
    networks:
      - backend-net
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: mydb
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
    volumes:
      - mysql-data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
    ports:
      - "3306:3306"        # Remove in production!

  # ── REDIS ─────────────────────────────────────────────
  redis:
    image: redis:7-alpine
    container_name: myapp-redis
    restart: unless-stopped
    networks:
      - backend-net
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    ports:
      - "6379:6379"

# ── NETWORKS ──────────────────────────────────────────────
networks:
  frontend-net:
    driver: bridge
  backend-net:
    driver: bridge
    internal: true          # No external access!

# ── VOLUMES ───────────────────────────────────────────────
volumes:
  mysql-data:
    driver: local
  redis-data:
    driver: local
  logs-volume:
    driver: local
```

### Docker Compose Commands

```bash
#!/bin/bash
# ── LIFECYCLE ────────────────────────────────────────────
docker compose up                         # Start all services
docker compose up -d                      # Detached mode
docker compose up --build                 # Rebuild images first
docker compose up --build --no-deps frontend  # Rebuild specific service
docker compose down                       # Stop + remove containers/networks
docker compose down -v                    # Also remove volumes (CAREFUL!)
docker compose down --rmi all             # Also remove images
docker compose start                      # Start stopped services
docker compose stop                       # Stop without removing
docker compose restart backend            # Restart specific service

# ── SCALE ────────────────────────────────────────────────
docker compose up --scale backend=3       # Run 3 instances of backend

# ── INSPECT ──────────────────────────────────────────────
docker compose ps                         # Service status
docker compose logs                       # All service logs
docker compose logs -f backend            # Follow backend logs
docker compose logs --tail=50 backend     # Last 50 lines
docker compose top                        # Running processes

# ── EXEC/RUN ─────────────────────────────────────────────
docker compose exec backend bash          # Shell into running service
docker compose run --rm backend pytest    # One-off command

# ── BUILD/PULL ───────────────────────────────────────────
docker compose build                      # Build all services
docker compose build --no-cache backend   # Rebuild specific
docker compose pull                       # Pull latest images

# ── CONFIG ───────────────────────────────────────────────
docker compose config                     # Validate + show merged config
docker compose --env-file .env.prod up    # Use specific env file
docker compose -f docker-compose.yml -f docker-compose.prod.yml up

# ── PROFILES (selective services) ────────────────────────
# In compose file: profiles: ["debug"]
docker compose --profile debug up         # Include debug services
```

### Environment Variable Handling

```bash
# .env file (auto-loaded by docker compose)
DB_USER=admin
DB_PASSWORD=secretpass
REDIS_PASSWORD=redispass
DB_ROOT_PASSWORD=rootpass
```

```yaml
# Override per environment
# docker-compose.override.yml (auto-merged in dev)
services:
  backend:
    volumes:
      - ./backend/src:/app/src    # Hot-reload in dev
    environment:
      - SPRING_PROFILES_ACTIVE=dev

# docker-compose.prod.yml (explicitly merged in prod)
services:
  backend:
    deploy:
      replicas: 3
    environment:
      - SPRING_PROFILES_ACTIVE=prod
```

---

## 9. Docker Swarm

### Swarm Architecture

```
┌──────────────────────────────────────────────────────────┐
│                   DOCKER SWARM CLUSTER                   │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │           MANAGER NODES (Raft Consensus)           │  │
│  │                                                    │  │
│  │  Manager 1 (Leader)                               │  │
│  │  Manager 2 (Follower)                             │  │
│  │  Manager 3 (Follower)                             │  │
│  │                                                    │  │
│  │  • Maintain cluster state                         │  │
│  │  • Schedule services                              │  │
│  │  • Serve Swarm API                                │  │
│  └──────────────────────┬─────────────────────────────┘  │
│                         │                                │
│           ┌─────────────┼─────────────┐                  │
│           ▼             ▼             ▼                  │
│     Worker Node 1  Worker Node 2  Worker Node 3          │
│     (Tasks/Containers)                                   │
└──────────────────────────────────────────────────────────┘
```

### Swarm Commands

```bash
#!/bin/bash
# ── INIT/JOIN ────────────────────────────────────────────
docker swarm init --advertise-addr 192.168.1.10
docker swarm join --token <token> 192.168.1.10:2377
docker swarm join-token worker        # Get worker token
docker swarm join-token manager       # Get manager token
docker swarm leave --force            # Leave swarm

# ── NODE MANAGEMENT ──────────────────────────────────────
docker node ls                        # List nodes
docker node inspect <node>            # Node details
docker node update --availability=drain <node>  # Drain node
docker node update --availability=active <node>
docker node promote <node>            # Worker → Manager
docker node demote <node>             # Manager → Worker
docker node rm <node>                 # Remove node

# ── SERVICES ─────────────────────────────────────────────
docker service create \
  --name my-web \
  --replicas 3 \
  --publish 80:80 \
  --network my-overlay \
  nginx:latest

docker service ls                     # List services
docker service ps my-web              # Tasks (containers) per service
docker service inspect my-web
docker service logs my-web -f         # Follow logs
docker service scale my-web=5         # Scale up/down
docker service update --image nginx:1.25 my-web  # Rolling update
docker service update --rollback my-web           # Rollback
docker service rm my-web              # Delete service

# ── STACKS (Compose in Swarm) ────────────────────────────
docker stack deploy -c docker-compose.yml myapp
docker stack ls
docker stack services myapp
docker stack ps myapp
docker stack rm myapp

# ── SECRETS ──────────────────────────────────────────────
echo "mysecretpassword" | docker secret create db_password -
docker secret ls
docker secret inspect db_password
docker secret rm db_password

# Use in service
docker service create \
  --secret db_password \
  --env DB_PASSWORD_FILE=/run/secrets/db_password \
  myapp
```

**Swarm vs Kubernetes:**

| Feature | Docker Swarm | Kubernetes |
|---------|-------------|-----------|
| Setup | Easy (5 min) | Complex (hours) |
| Learning curve | Low | High |
| Scaling | Good | Excellent |
| Auto-healing | Yes | Yes (more advanced) |
| Load balancing | Built-in (VIP) | Advanced (Ingress) |
| Secrets | Built-in | Encrypted (etcd) |
| Rolling updates | Yes | Yes (+ canary, blue-green) |
| Best for | Small-medium apps | Enterprise/large scale |

---

## 10. Security

### Docker Security Layers

```
┌─────────────────────────────────────────────────────┐
│              DOCKER SECURITY LAYERS                 │
│                                                     │
│  1. Build-time security                             │
│     ├── Use official base images                   │
│     ├── Scan images (Trivy, Snyk)                  │
│     ├── Non-root USER                               │
│     └── Minimal base (alpine, distroless)           │
│                                                     │
│  2. Runtime security                                │
│     ├── Read-only filesystem                        │
│     ├── Resource limits (--memory, --cpus)         │
│     ├── Drop capabilities                           │
│     ├── No privileged mode                         │
│     └── Security profiles (seccomp, AppArmor)      │
│                                                     │
│  3. Network security                                │
│     ├── Custom networks (not default bridge)        │
│     ├── Minimize exposed ports                      │
│     └── Use encrypted overlay networks              │
│                                                     │
│  4. Secrets management                              │
│     ├── Docker Swarm secrets                        │
│     ├── HashiCorp Vault                             │
│     └── Never in Dockerfiles or env vars           │
└─────────────────────────────────────────────────────┘
```

### Secure Dockerfile

```dockerfile
# ── SECURE DOCKERFILE ────────────────────────────────────
# Use specific version tags (not :latest)
FROM node:20.11.0-alpine3.19

# Install security updates
RUN apk update && apk upgrade && rm -rf /var/cache/apk/*

WORKDIR /app

# Copy and install dependencies first (better caching)
COPY package*.json ./
RUN npm ci --only=production

# Copy source
COPY --chown=node:node . .

# Remove unnecessary tools
RUN rm -rf /usr/local/lib/node_modules/npm

# Use non-root user
USER node

# Read-only hint
VOLUME ["/tmp"]

EXPOSE 8080

CMD ["node", "server.js"]
```

### Secure Container Run

```bash
#!/bin/bash
docker run \
  --read-only \                              # Read-only filesystem
  --tmpfs /tmp:size=100m \                  # Writable temp storage
  --user 1000:1000 \                        # Non-root user
  --cap-drop ALL \                          # Drop all capabilities
  --cap-add NET_BIND_SERVICE \              # Add only needed caps
  --security-opt=no-new-privileges \        # No privilege escalation
  --security-opt seccomp=my-profile.json \ # Seccomp profile
  --memory 512m \                           # Memory limit
  --memory-swap 512m \                      # Disable swap
  --cpus 1.0 \                             # CPU limit
  --pids-limit 100 \                       # Process limit
  --network my-secure-net \                # Custom network
  --no-healthcheck \                       # Or use custom healthcheck
  myapp:v1
```

### Image Scanning

```bash
#!/bin/bash
# Install Trivy
# curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh

# Scan image
trivy image myapp:v1
trivy image --severity HIGH,CRITICAL myapp:v1
trivy image --exit-code 1 --severity CRITICAL myapp:v1  # Fail on critical

# Scan in CI/CD
trivy image --format json --output results.json myapp:v1
trivy fs .                                              # Scan local directory
trivy config ./                                         # Scan Dockerfile
```

---

## 11. Registry

### Registry Types

| Registry | Type | Use Case |
|----------|------|---------|
| Docker Hub | Public/Private cloud | Default, open source projects |
| AWS ECR | Private cloud | AWS workloads |
| Google GCR/Artifact Registry | Private cloud | GCP workloads |
| GitHub GHCR | Private/Public | GitHub CI/CD integration |
| Harbor | Self-hosted | Enterprise, air-gapped |
| Nexus/JFrog | Self-hosted | Enterprise artifact management |

### Registry Commands

```bash
#!/bin/bash
# ── DOCKER HUB ───────────────────────────────────────────
docker login
docker login -u sahil0117 -p <token>    # Use token not password!
docker logout

docker push sahil0117/myapp:v1
docker pull sahil0117/myapp:v1

# ── AWS ECR ──────────────────────────────────────────────
# Authenticate
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS \
  --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com

# Create repo
aws ecr create-repository --repository-name myapp --region us-east-1

# Tag and push
docker tag myapp:v1 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:v1
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:v1

# ── PRIVATE REGISTRY ─────────────────────────────────────
# Run private registry
docker run -d \
  -p 5000:5000 \
  --name registry \
  -v registry-data:/var/lib/registry \
  registry:2

# Push to private registry
docker tag myapp:v1 localhost:5000/myapp:v1
docker push localhost:5000/myapp:v1
docker pull localhost:5000/myapp:v1
```

---

## 12. Monitoring & Logging

### Container Statistics

```bash
#!/bin/bash
# Real-time stats
docker stats
docker stats --no-stream                  # One-time snapshot
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Logs
docker logs my-container
docker logs -f my-container               # Follow
docker logs --tail 100 my-container       # Last 100 lines
docker logs --since 1h my-container       # Since 1 hour ago
docker logs --until 2026-05-24T10:00 my-container
docker logs my-container 2>&1 | grep ERROR

# System info
docker system df                          # Disk usage
docker system events                      # Real-time events
docker system info                        # System information
```

### Logging Drivers

```bash
#!/bin/bash
# Default: json-file (stored in /var/lib/docker/containers/)
docker run --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  nginx

# Syslog
docker run --log-driver=syslog nginx

# AWS CloudWatch
docker run \
  --log-driver=awslogs \
  --log-opt awslogs-group=/docker/myapp \
  --log-opt awslogs-region=us-east-1 \
  nginx

# Fluentd
docker run \
  --log-driver=fluentd \
  --log-opt fluentd-address=localhost:24224 \
  nginx
```

### Prometheus Monitoring Setup

```yaml
# docker-compose.monitoring.yml
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"
```

---

## 13. Best Practices

### Production Docker Best Practices

```
✅ DO:
├── Use specific image tags (nginx:1.25.3, not nginx:latest)
├── Multi-stage builds to minimize image size
├── Non-root USER in Dockerfile
├── Add .dockerignore file
├── Use official/verified base images
├── One process per container
├── Set resource limits (--memory, --cpus)
├── Use health checks
├── Use named volumes for persistent data
├── Scan images before production (Trivy)
├── Use --read-only flag where possible
├── Use secrets management (Swarm secrets, Vault)
├── Tag images with git commit hash or version
└── Use Docker Content Trust for image signing

❌ DON'T:
├── Use :latest tag in production
├── Run as root
├── Store secrets in Dockerfiles or env vars in compose
├── Use --privileged unless absolutely necessary
├── Put everything in one layer
├── Ignore health check configuration
├── Leave debug tools in production images
└── Expose unnecessary ports
```

### Image Size Optimization

```dockerfile
# Use alpine base
FROM python:3.11-alpine        # vs python:3.11 (900MB smaller!)

# Use distroless (Google)
FROM gcr.io/distroless/python3:latest   # No shell, minimal attack surface

# Multi-stage (already covered)
# Copy only needed files
# Clean package manager cache in same layer
RUN apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# Use --squash (experimental) to merge layers
docker build --squash -t myapp .
```

---

## 14. Quick Reference

```bash
#!/bin/bash
# =============================================
# DOCKER COMPLETE COMMANDS CHEATSHEET
# =============================================

# ── IMAGES ───────────────────────────────────
docker build -t name:tag .               # Build image
docker build --no-cache -t name .        # Build without cache
docker images                            # List images
docker pull nginx:alpine                 # Pull image
docker push user/repo:tag                # Push to registry
docker rmi image:tag                     # Remove image
docker image prune -a                    # Remove unused images
docker image history image:tag           # Layer history
docker image inspect image:tag           # Full metadata
docker save image:tag | gzip > file.tar  # Export image
docker load < file.tar.gz                # Import image

# ── CONTAINERS ───────────────────────────────
docker run -d -p 8080:80 --name web nginx  # Run detached
docker run -it ubuntu bash               # Interactive
docker run --rm alpine echo hello        # Auto-remove
docker ps                                # Running containers
docker ps -a                             # All containers
docker stop container                    # Graceful stop
docker kill container                    # Force stop
docker rm container                      # Remove stopped
docker rm -f container                   # Force remove
docker logs -f container                 # Follow logs
docker exec -it container bash           # Shell into container
docker stats container                   # Resource usage
docker inspect container                 # Full details
docker cp file container:/path           # Copy to container
docker cp container:/path file           # Copy from container
docker top container                     # Processes in container

# ── NETWORKS ─────────────────────────────────
docker network create mynet              # Create network
docker network ls                        # List networks
docker network inspect mynet             # Network details
docker network connect mynet container   # Connect container
docker network disconnect mynet cont     # Disconnect
docker network rm mynet                  # Remove network
docker network prune                     # Remove unused

# ── VOLUMES ──────────────────────────────────
docker volume create myvol               # Create volume
docker volume ls                         # List volumes
docker volume inspect myvol             # Volume details
docker volume rm myvol                  # Remove volume
docker volume prune                     # Remove unused

# ── COMPOSE ──────────────────────────────────
docker compose up -d                     # Start all services
docker compose down                      # Stop all
docker compose down -v                   # Stop + remove volumes
docker compose up --build -d             # Rebuild + start
docker compose ps                        # Service status
docker compose logs -f service           # Follow logs
docker compose exec service bash        # Shell into service
docker compose scale service=3          # Scale service
docker compose pull                      # Update images
docker compose config                    # Validate config

# ── SWARM ────────────────────────────────────
docker swarm init                        # Initialize swarm
docker swarm join --token <token> host  # Join swarm
docker node ls                           # List nodes
docker service create --replicas 3 ...  # Create service
docker service ls                        # List services
docker service scale svc=5              # Scale service
docker service update --image img svc   # Rolling update
docker service rollback svc             # Rollback
docker stack deploy -c compose.yml app  # Deploy stack

# ── SYSTEM ───────────────────────────────────
docker system info                       # System info
docker system df                         # Disk usage
docker system prune                      # Remove unused
docker system prune -a --volumes         # Full cleanup ⚠️
docker system events                     # Real-time events
docker version                           # Docker version
docker info                              # Detailed info

# ── REGISTRY ─────────────────────────────────
docker login                             # Login to Docker Hub
docker login registry.example.com       # Login to private
docker logout                            # Logout
docker search nginx                      # Search images
docker tag img:v1 user/img:v1           # Tag for push
```

---
