
# 🐳 Docker — Complete Interview Guide

> **Simple → Advanced → Scenario-Based**

---

## 📁 Table of Contents

| # | Section | Questions |
|---|---------|-----------|
| 1 | [Basics & Fundamentals](#1-basics--fundamentals) | Q1–Q20 — Simple questions |
| 2 | [Images & Dockerfile](#2-images--dockerfile) | Q21–Q35 — Build, layers, multi-stage |
| 3 | [Containers & Runtime](#3-containers--runtime) | Q36–Q50 — Lifecycle, resources |
| 4 | [Networking](#4-networking) | Q51–Q60 — Bridge, overlay, DNS |
| 5 | [Volumes & Storage](#5-volumes--storage) | Q61–Q70 — Persistence, mounts |
| 6 | [Docker Compose](#6-docker-compose) | Q71–Q80 — Multi-container apps |
| 7 | [Docker Swarm](#7-docker-swarm) | Q81–Q88 — Clustering |
| 8 | [Security](#8-security) | Q89–Q98 — Hardening, secrets |
| 9 | [Advanced Concepts](#9-advanced-concepts) | Q99–Q115 — Internals, optimization |
| 10 | [Scenario-Based Questions](#10-scenario-based-questions) | 60 real-world scenarios |

---

## 1. Basics & Fundamentals

### Q1. What is Docker?

Docker is an open-source **containerization platform** that packages applications and their dependencies into lightweight, portable **containers**. Containers run consistently across any environment — development, staging, and production.

```
Developer builds image → Push to registry → Pull anywhere → Run identically
```

---

### Q2. What is the difference between a Container and a Virtual Machine?

| Feature | Container | Virtual Machine |
|---------|-----------|----------------|
| OS | Shares host kernel | Separate guest OS |
| Size | MBs | GBs |
| Startup | Milliseconds | Minutes |
| Isolation | Process-level | Hardware-level |
| Performance | Near-native | Overhead |
| Portability | Very high | Medium |
| Resource use | Low | High |

```
VM:
┌──────────┬──────────┐
│  App A   │  App B   │
│  Guest OS│  Guest OS│
├──────────┴──────────┤
│      Hypervisor     │
│      Host OS        │
│      Hardware       │
└─────────────────────┘

Container:
┌──────────┬──────────┐
│  App A   │  App B   │
├──────────┴──────────┤
│    Docker Engine    │
│      Host OS        │
│      Hardware       │
└─────────────────────┘
```

---

### Q3. What are the main components of Docker architecture?

| Component | Description |
|-----------|-------------|
| **Docker Client** | CLI (`docker`) — sends commands to daemon |
| **Docker Daemon** (`dockerd`) | Background service managing everything |
| **Docker Registry** | Store for images (Docker Hub, ECR) |
| **Docker Objects** | Images, containers, networks, volumes |
| **containerd** | Container runtime (manages lifecycle) |
| **runc** | Low-level runtime (creates containers) |

---

### Q4. What is a Docker Image?

A Docker Image is a **read-only template** used to create containers. It consists of stacked layers — each `RUN`, `COPY`, `ADD` instruction in a Dockerfile creates a new layer.

```
Image = Layer 0 (base OS)
      + Layer 1 (RUN apt-get install)
      + Layer 2 (COPY package.json)
      + Layer 3 (RUN npm install)
      + Layer 4 (COPY app code)
```

Images are **immutable** — you never modify an image. You build a new one.

---

### Q5. What is a Docker Container?

A container is a **running instance of an image**. It adds a writable layer on top of the read-only image layers.

```
Container = Image (read-only layers)
          + Container Layer (writable, lost on stop)
          + Running process
          + Isolated network, filesystem, PID
```

Multiple containers can run from the same image — they share image layers but have separate writable layers.

---

### Q6. What is a Dockerfile?

A Dockerfile is a text file containing **instructions** to build a Docker image. Each instruction creates a layer.

```dockerfile
FROM node:20-alpine          # Base image
WORKDIR /app                 # Set working directory
COPY package*.json ./        # Copy files
RUN npm install              # Execute command (creates layer)
COPY . .                     # Copy remaining files
EXPOSE 3000                  # Document port
CMD ["node", "server.js"]    # Default command
```

---

### Q7. What is Docker Hub?

Docker Hub is Docker's official **cloud-based registry** for storing and sharing Docker images.

- **Public images** — free, accessible to everyone
- **Private images** — requires paid plan
- **Official images** — verified by Docker (nginx, postgres, node, etc.)
- **Verified publisher** images — from trusted organizations

```bash
docker pull nginx:alpine              # Pull from Docker Hub
docker push username/myapp:v1         # Push to Docker Hub
docker search nginx                   # Search images
```

---

### Q8. What is the difference between `docker run` and `docker start`?

| Command | When to use |
|---------|------------|
| `docker run` | Create AND start a **new** container from an image |
| `docker start` | Start an **existing stopped** container |

```bash
docker run -d --name web nginx        # Creates new container + starts it
docker stop web                       # Stop it
docker start web                      # Start the SAME container again
```

---

### Q9. What is the difference between `COPY` and `ADD` in Dockerfile?

| Feature | COPY | ADD |
|---------|------|-----|
| Local files | ✅ Yes | ✅ Yes |
| URLs | ❌ No | ✅ Yes |
| Auto-extract archives (.tar) | ❌ No | ✅ Yes |
| Recommended for | Most cases | Only when URL/tar needed |

```dockerfile
COPY ./app /app            # Simple, predictable
ADD https://url/file /tmp  # Downloads file
ADD archive.tar.gz /app    # Extracts automatically
```

**Best practice:** Use `COPY` — it's explicit and predictable. Use `ADD` only when you need URL download or tar extraction.

---

### Q10. What is the difference between `CMD` and `ENTRYPOINT`?

| | CMD | ENTRYPOINT |
|--|-----|-----------|
| Purpose | Default arguments | Fixed executable |
| Overridable | ✅ Yes (docker run args) | ❌ No (need --entrypoint) |
| Use case | Default command for container | Define container behavior |

```dockerfile
# CMD only
CMD ["nginx", "-g", "daemon off;"]
# docker run image         → runs nginx
# docker run image bash    → runs bash (overrides CMD)

# ENTRYPOINT only
ENTRYPOINT ["nginx"]
# docker run image                → runs nginx
# docker run image -g "daemon off;" → runs nginx -g "daemon off;"

# Both (recommended pattern)
ENTRYPOINT ["node"]
CMD ["server.js"]
# docker run image          → runs: node server.js
# docker run image debug.js → runs: node debug.js
```

---

### Q11. What is the difference between `docker stop` and `docker kill`?

| Command | Signal | Behavior |
|---------|--------|---------|
| `docker stop` | SIGTERM → SIGKILL | Graceful shutdown (app can clean up) |
| `docker kill` | SIGKILL | Immediate termination |

```bash
docker stop -t 30 my-container   # Give 30 seconds for graceful shutdown
docker kill my-container          # Immediate — use only if stop hangs
```

**Best practice:** Always use `docker stop` in production. Applications should handle SIGTERM for graceful shutdown.

---

### Q12. What are Docker layers and how do they work?

Each instruction in a Dockerfile creates a read-only layer. Layers are **cached** and **shared** between images.

```
docker build → processes each instruction
               checks cache for each layer
               if cached → reuse (fast!)
               if not cached → execute + create new layer

docker run  → adds writable container layer on top
              all layers below = shared, read-only
```

**Union filesystem (OverlayFS)** makes multiple layers appear as one unified filesystem.

---

### Q13. What is a Docker Registry?

A registry is a **server-side storage and distribution system** for Docker images.

| Registry | Type | Use case |
|----------|------|---------|
| Docker Hub | Public/Private | Default registry |
| AWS ECR | Private | AWS deployments |
| Google GCR | Private | GCP deployments |
| GitHub GHCR | Public/Private | GitHub CI/CD |
| Harbor | Self-hosted | Enterprise |
| JFrog Artifactory | Self-hosted | Enterprise |

```bash
docker login registry.example.com
docker tag myapp:v1 registry.example.com/myapp:v1
docker push registry.example.com/myapp:v1
```

---

### Q14. What is a Docker Volume?

A volume is Docker-managed **persistent storage** that exists outside the container lifecycle. Data survives container restarts and deletions.

```bash
docker volume create my-data
docker run -v my-data:/app/data nginx
docker volume ls
docker volume inspect my-data
docker volume rm my-data
```

---

### Q15. What is Docker Compose?

Docker Compose is a tool for defining and running **multi-container applications** using a YAML file. One command starts all services together.

```bash
docker compose up -d       # Start all services defined in docker-compose.yml
docker compose down        # Stop and remove everything
docker compose ps          # Service status
```

---

### Q16. What are Docker namespaces?

Linux namespaces provide **isolation** for containers — each container thinks it's the only process on the system.

| Namespace | Isolates |
|-----------|---------|
| `PID` | Process IDs |
| `NET` | Network interfaces, ports |
| `MNT` | Filesystems, mount points |
| `UTS` | Hostname, domain name |
| `IPC` | Inter-process communication |
| `USER` | User and group IDs |

---

### Q17. What are cgroups in Docker?

**Control groups (cgroups)** limit and monitor resources a container can use:

```bash
docker run \
  --memory=512m \       # Max 512MB RAM
  --memory-swap=512m \  # No swap
  --cpus=1.5 \          # Max 1.5 CPU cores
  --pids-limit=100 \    # Max 100 processes
  myapp
```

---

### Q18. What is Docker Content Trust (DCT)?

DCT enables **image signing and verification** — ensures images come from trusted sources and haven't been tampered with.

```bash
export DOCKER_CONTENT_TRUST=1      # Enable DCT
docker push myapp:v1               # Automatically signed
docker pull myapp:v1               # Verified before pull
```

---

### Q19. What is the difference between `docker images` and `docker image ls`?

They are **identical** — `docker image ls` is the newer, more explicit syntax. Both list all locally cached images.

```bash
docker images                      # Old syntax
docker image ls                    # New syntax (preferred)
docker image ls --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

---

### Q20. What does `docker system prune` do?

Removes **all unused Docker objects** — stopped containers, dangling images, unused networks, and optionally volumes.

```bash
docker system prune            # Remove stopped containers + dangling images + unused networks
docker system prune -a         # Also remove unused images (not just dangling)
docker system prune -a --volumes  # Also remove volumes ⚠️ CAREFUL!
docker system df               # Show disk usage before pruning
```

---

## 2. Images & Dockerfile

### Q21. What is a multi-stage build and why use it?

Multi-stage builds use **multiple FROM statements** in one Dockerfile — each stage can copy artifacts from previous stages. Result: small production image without build tools.

```dockerfile
# Stage 1: Build (large — includes compiler, build tools)
FROM maven:3.9-eclipse-temurin-21 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime (small — only JRE, no Maven)
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]

# Result: Build image = ~700MB, Runtime image = ~200MB
```

**Benefits:**
- Drastically smaller production images
- No build tools in production (security)
- Build dependencies don't leak into production

---

### Q22. What is the build cache and how does it work?

Docker caches each layer during build. On rebuild, Docker checks if the instruction and its inputs changed:

```dockerfile
FROM node:20-alpine                # Layer 1: cached (same base)
WORKDIR /app                       # Layer 2: cached
COPY package*.json ./              # Layer 3: cached IF package.json unchanged
RUN npm install                    # Layer 4: cached IF layer 3 cached
COPY . .                           # Layer 5: ALWAYS changes (source changes)
```

**Cache busting strategies:**
```bash
docker build --no-cache -t myapp . # Ignore all cache
docker build --build-arg CACHEBUST=$(date +%s) -t myapp .  # Selective bust
```

---

### Q23. What is a dangling image?

A dangling image is an image with **no tag** — created when you rebuild an image with the same tag. The old image loses its tag but remains on disk.

```bash
docker images -f "dangling=true"   # List dangling images
docker image prune                 # Remove dangling images
```

```
# Before rebuild:
myapp:latest   → sha256:abc123

# After rebuild with same tag:
myapp:latest   → sha256:def456   (new)
<none>:<none>  → sha256:abc123   (dangling — old version)
```

---

### Q24. What is the difference between `docker build` and `docker commit`?

| | `docker build` | `docker commit` |
|--|---------------|----------------|
| Source | Dockerfile | Running container |
| Reproducible | ✅ Yes | ❌ No |
| Version controlled | ✅ Yes (Dockerfile in Git) | ❌ No |
| Recommended | ✅ Always | ❌ Only for debugging |

```bash
# docker commit (NOT recommended for production)
docker exec -it mycontainer bash
# ... make changes ...
docker commit mycontainer myapp:debug-version
```

**Best practice:** Always use Dockerfiles. `docker commit` is for debugging only.

---

### Q25. How do you reduce Docker image size?

```dockerfile
# 1. Use minimal base image
FROM alpine:3.19              # ~5MB vs ubuntu ~70MB
FROM node:20-alpine           # Alpine variant
FROM gcr.io/distroless/java21 # Google Distroless — no shell!

# 2. Multi-stage builds
FROM node:20 AS builder
RUN npm run build
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html

# 3. Clean package manager cache in SAME layer
RUN apt-get update && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*

# 4. .dockerignore file
# node_modules/, .git/, *.log, dist/, coverage/

# 5. Combine RUN commands
RUN cmd1 && cmd2 && cmd3   # 1 layer vs 3 layers

# 6. Use --squash (merge all layers)
docker build --squash -t myapp .
```

---

### Q26. What is the difference between `ARG` and `ENV`?

| Feature | `ARG` | `ENV` |
|---------|-------|-------|
| Available during build | ✅ Yes | ✅ Yes |
| Available at runtime | ❌ No | ✅ Yes |
| Visible in `docker inspect` | ❌ No | ✅ Yes |
| Override from CLI | `--build-arg` | `-e` or `--env` |
| Use for secrets | ❌ No (visible in history!) | ❌ No |

```dockerfile
ARG NODE_VERSION=20          # Build-time only
ENV APP_PORT=8080            # Runtime + build-time

# Build with custom arg
# docker build --build-arg NODE_VERSION=18 .
# docker run -e APP_PORT=9090 myapp
```

---

### Q27. What is a base image and how do you choose one?

A base image is the starting `FROM` layer. Choosing the right one affects:

| Criteria | Recommendation |
|----------|---------------|
| Size | Alpine > Slim > Full |
| Security | Smaller = fewer vulnerabilities |
| Compatibility | Ensure all dependencies available |
| Official | Use Docker Official Images when possible |

```dockerfile
# Options for Node.js (in order of size, smallest first):
FROM node:20-alpine        # ~40MB — most apps
FROM node:20-slim          # ~70MB — if Alpine lacks deps
FROM node:20               # ~1GB — avoid in production
FROM gcr.io/distroless/nodejs20-debian12  # Most secure
```

---

### Q28. What is `EXPOSE` and does it actually publish the port?

`EXPOSE` is **documentation only** — it declares which port the container listens on but does NOT publish it to the host.

```dockerfile
EXPOSE 8080       # Documents that container uses port 8080
```

To actually publish:
```bash
docker run -p 8080:8080 myapp    # Host:Container port mapping
docker run -P myapp              # Auto-map EXPOSE'd ports to random host ports
```

---

### Q29. What is `HEALTHCHECK` in Dockerfile?

`HEALTHCHECK` tells Docker how to test if a container is healthy.

```dockerfile
HEALTHCHECK --interval=30s \
            --timeout=10s \
            --start-period=30s \
            --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1
```

Status values:
- `0` — healthy
- `1` — unhealthy
- `2` — reserved

```bash
docker ps           # Shows health status
docker inspect --format='{{.State.Health.Status}}' mycontainer
```

---

### Q30. What is the difference between `RUN`, `CMD`, and `ENTRYPOINT`?

| Instruction | When runs | Can override | Use for |
|-------------|----------|--------------|---------|
| `RUN` | During **build** | N/A | Install packages, build steps |
| `CMD` | At **container start** | Yes (docker run args) | Default arguments |
| `ENTRYPOINT` | At **container start** | Only with `--entrypoint` | Define container's main executable |

```dockerfile
RUN npm install              # Build time — installs packages
CMD ["node", "server.js"]    # Runtime — default command
ENTRYPOINT ["node"]          # Runtime — fixed executable
```

---

### Q31. How do you pass secrets to Docker build without exposing them in layers?

```dockerfile
# Using BuildKit secrets (recommended)
# syntax=docker/dockerfile:1
FROM node:20-alpine

# Mount secret during build — NOT stored in image layers!
RUN --mount=type=secret,id=npm_token \
    NPM_TOKEN=$(cat /run/secrets/npm_token) npm install

# Build command
# DOCKER_BUILDKIT=1 docker build --secret id=npm_token,src=.npmrc .
```

```bash
# Alternative: multi-stage (delete secret in final stage)
FROM node:20 AS builder
ARG GITHUB_TOKEN                    # ⚠️ Visible in history — avoid for real secrets
RUN git clone https://${GITHUB_TOKEN}@github.com/org/private-repo .
```

---

### Q32. What is `docker image inspect` used for?

Provides **complete metadata** about an image in JSON format.

```bash
docker image inspect nginx:alpine

# Specific fields
docker image inspect --format='{{.Config.Cmd}}' nginx
docker image inspect --format='{{.Config.Env}}' nginx
docker image inspect --format='{{.RootFS.Layers}}' nginx
docker image inspect --format='{{.RepoDigests}}' nginx   # Content hash
```

---

### Q33. What is a Docker manifest?

A manifest describes an image that supports **multiple architectures** (amd64, arm64, etc.).

```bash
docker manifest inspect nginx:alpine           # See all supported platforms
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:multi .
docker buildx build --push --platform linux/amd64,linux/arm64 -t user/myapp .
```

---

### Q34. How does Docker handle layer caching in CI/CD?

```yaml
# GitHub Actions — cache Docker layers
- name: Cache Docker layers
  uses: actions/cache@v3
  with:
    path: /tmp/.buildx-cache
    key: ${{ runner.os }}-buildx-${{ hashFiles('Dockerfile') }}

- name: Build with cache
  run: |
    docker buildx build \
      --cache-from type=local,src=/tmp/.buildx-cache \
      --cache-to type=local,dest=/tmp/.buildx-cache-new \
      --push \
      -t myapp:${{ github.sha }} .
```

---

### Q35. What is `docker save` vs `docker export`?

| Command | Saves | Use case |
|---------|-------|---------|
| `docker save` | Image (with layers + metadata) | Transfer image between hosts |
| `docker export` | Container filesystem (flat) | Extract files, no layer info |

```bash
# Save image (preserves layers, tags, history)
docker save myapp:v1 | gzip > myapp-v1.tar.gz
docker load < myapp-v1.tar.gz

# Export container (flat filesystem, no history)
docker export mycontainer | gzip > container.tar.gz
docker import container.tar.gz myimage:imported
```

---

## 3. Containers & Runtime

### Q36. What is the container lifecycle?

```
Image
  │
  ▼
Created ──► Running ──► Paused ──► Running
  │             │
  │             ▼
  │          Stopped ──► Deleted
  │             │
  └─────────────┘ (restart)
```

```bash
docker create myapp        # Created (not running)
docker start mycontainer   # Created → Running
docker pause mycontainer   # Running → Paused (SIGSTOP)
docker unpause mycontainer # Paused → Running
docker stop mycontainer    # Running → Stopped (SIGTERM + SIGKILL)
docker rm mycontainer      # Stopped → Deleted
```

---

### Q37. What are restart policies?

| Policy | Behavior |
|--------|---------|
| `no` | Never restart (default) |
| `always` | Always restart when stopped |
| `on-failure` | Restart only on non-zero exit code |
| `unless-stopped` | Always restart unless manually stopped |

```bash
docker run --restart=always nginx
docker run --restart=on-failure:5 myapp   # Max 5 retries
docker update --restart=always mycontainer  # Update running container
```

---

### Q38. How do you limit container resources?

```bash
# Memory
docker run --memory=512m myapp              # Hard limit
docker run --memory=512m --memory-swap=512m myapp  # Disable swap
docker run --memory-reservation=256m myapp  # Soft limit

# CPU
docker run --cpus=1.5 myapp               # 1.5 CPU cores
docker run --cpu-shares=512 myapp          # Relative weight (default 1024)
docker run --cpuset-cpus="0,1" myapp      # Only use CPU 0 and 1

# I/O
docker run --blkio-weight=300 myapp        # I/O weight (default 500)
docker run --device-read-bps /dev/sda:1mb myapp

# PIDs
docker run --pids-limit=100 myapp          # Max 100 processes
```

---

### Q39. What is an ephemeral container?

Containers are **ephemeral by design** — they can be stopped, deleted, and recreated at any time without data loss (data goes in volumes, config in environment).

This design principle means:
- No storing application state inside container
- No storing logs inside container (use logging drivers)
- Configuration via environment variables or mounted configs
- Data via named volumes or external databases

---

### Q40. How do you run multiple processes in a Docker container?

```dockerfile
# Method 1: Use supervisord (init system)
FROM ubuntu
RUN apt-get install -y supervisor
COPY supervisord.conf /etc/supervisor/conf.d/
CMD ["/usr/bin/supervisord"]

# Method 2: Use tini (lightweight init)
FROM node:20-alpine
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "server.js"]

# Method 3: Shell script as entrypoint
COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

**Best practice:** One main process per container. Use multiple containers + Compose instead.

---

### Q41. What is `docker exec` vs `docker attach`?

| | `docker exec` | `docker attach` |
|--|--------------|----------------|
| Creates | New process in container | Connects to main process |
| On exit | Container keeps running | Container may stop |
| Use case | Debugging, inspection | Watch main process output |

```bash
docker exec -it mycontainer bash      # New shell (preferred)
docker exec mycontainer ls /app       # Run single command
docker attach mycontainer             # Connect to PID 1 (risky!)
```

---

### Q42. How do you copy files between host and container?

```bash
# Host → Container
docker cp ./config.json mycontainer:/app/config.json
docker cp ./uploads/ mycontainer:/var/www/uploads/

# Container → Host
docker cp mycontainer:/app/logs/error.log ./error.log
docker cp mycontainer:/var/www/ ./backup/

# Note: Copying doesn't require container to be running for files
# But container must exist (can be stopped)
```

---

### Q43. What is the difference between `docker rm` and `docker rmi`?

```bash
docker rm mycontainer     # Remove container (must be stopped first)
docker rm -f mycontainer  # Force remove running container
docker rmi myimage:tag    # Remove image (no containers using it)
docker rmi -f myimage     # Force remove image

# Bulk cleanup
docker rm $(docker ps -aq)     # Remove all stopped containers
docker rmi $(docker images -q) # Remove all images
```

---

### Q44. What is `docker inspect`?

Returns **low-level JSON information** about Docker objects.

```bash
docker inspect mycontainer           # Container details
docker inspect --format='{{.NetworkSettings.IPAddress}}' mycontainer
docker inspect --format='{{.State.Status}}' mycontainer
docker inspect --format='{{json .Mounts}}' mycontainer
docker inspect mynetwork             # Network details
docker inspect myvolume              # Volume details
```

---

### Q45. What is `docker stats`?

Live stream of **resource usage** for running containers.

```bash
docker stats                          # All containers
docker stats mycontainer              # Specific container
docker stats --no-stream              # Single snapshot (non-live)
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"
```

Output columns: CPU%, MEM USAGE/LIMIT, MEM%, NET I/O, BLOCK I/O, PIDS

---

### Q46. How do you run a Docker container in read-only mode?

```bash
docker run \
  --read-only \                    # Root filesystem read-only
  --tmpfs /tmp \                   # Writable temp storage (in memory)
  --tmpfs /var/run \               # Writable runtime directory
  -v logs-volume:/app/logs \       # Writable via volume
  myapp

# In docker-compose.yml
services:
  app:
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
```

---

### Q47. What is the difference between `SIGTERM` and `SIGKILL` in Docker?

| Signal | Docker command | Container behavior |
|--------|---------------|-------------------|
| `SIGTERM` | `docker stop` | Application can catch and clean up |
| `SIGKILL` | `docker kill` | Immediate termination, no cleanup |

```bash
# docker stop sends SIGTERM, waits 10s, then sends SIGKILL
docker stop -t 30 mycontainer   # Give 30s for graceful shutdown

# Application should handle SIGTERM:
process.on('SIGTERM', () => {
  server.close(() => {
    db.disconnect();
    process.exit(0);
  });
});
```

---

### Q48. What is a scratch image?

`scratch` is a special empty base image — contains nothing. Used for building **truly minimal images** (static binaries).

```dockerfile
# Build static binary in one stage
FROM golang:1.21 AS builder
RUN CGO_ENABLED=0 go build -o myapp .

# Use scratch as final base (0MB base!)
FROM scratch
COPY --from=builder /build/myapp /myapp
EXPOSE 8080
ENTRYPOINT ["/myapp"]
```

Final image = only your binary (a few MBs). No shell, no tools, no vulnerabilities from OS packages.

---

### Q49. What is `docker top`?

Displays **running processes** inside a container — like `ps` but for containers.

```bash
docker top mycontainer
docker top mycontainer aux        # ps aux format
docker top mycontainer -eo pid,ppid,cmd  # Custom columns
```

---

### Q50. How do you update a running container's configuration?

```bash
# Update resource limits without restart
docker update --memory=1g --cpus=2 mycontainer
docker update --restart=always mycontainer

# Update environment variables
# ❌ NOT possible without recreation
docker stop mycontainer
docker rm mycontainer
docker run -d -e NEW_VAR=value --name mycontainer myimage
```

---

## 4. Networking

### Q51. What are the Docker network drivers?

| Driver | Use case | Multi-host |
|--------|---------|-----------|
| `bridge` | Default, single host | ❌ |
| `host` | High performance, no isolation | ❌ |
| `overlay` | Docker Swarm, multi-host | ✅ |
| `macvlan` | Legacy app, needs MAC address | ✅ |
| `none` | Complete isolation | ❌ |
| `ipvlan` | Low-level network control | ✅ |

---

### Q52. What is the default `bridge` network and its limitation?

The default `bridge` network connects containers but **does NOT provide DNS-based service discovery** — containers must communicate via IP addresses.

```bash
# Default bridge — NO name resolution!
docker run -d --name app1 nginx
docker run -d --name app2 nginx
# app2 CANNOT reach app1 by name "app1" — must use IP

# Custom bridge — name resolution works!
docker network create mynet
docker run -d --network mynet --name app1 nginx
docker run -d --network mynet --name app2 nginx
# app2 CAN reach app1 by name: curl http://app1
```

**Always use custom bridge networks in production!**

---

### Q53. How does container DNS resolution work?

```
Custom bridge network:
  Container A (name: backend) → IP: 172.20.0.2
  Container B (name: frontend)
    → can reach backend via: http://backend:8080
    → Docker's embedded DNS resolves "backend" → 172.20.0.2

DNS server: 127.0.0.11 (Docker's embedded DNS)
```

```bash
# Test DNS inside container
docker exec -it frontend nslookup backend
docker exec -it frontend cat /etc/resolv.conf
# nameserver 127.0.0.11  ← Docker's DNS
```

---

### Q54. What is `host` network mode?

Container shares the **host's network stack** — no network isolation. Container port 80 = host port 80.

```bash
docker run --network host nginx
# nginx binds directly to host port 80 — no -p needed
```

**Use cases:** High performance networking, when container needs access to host network interfaces.
**Avoid for:** Production web apps (security risk).

---

### Q55. How do you connect a container to multiple networks?

```bash
docker network create frontend-net
docker network create backend-net

# Backend container connected to both networks
docker run -d --name backend --network backend-net myapp
docker network connect frontend-net backend   # Add second network

# Now backend can talk to both frontend AND database services
docker network disconnect backend-net backend  # Remove network
```

---

### Q56. What is an overlay network?

Overlay networks enable **multi-host container communication** — containers on different Docker hosts can talk to each other.

```bash
# Requires Docker Swarm
docker swarm init
docker network create --driver overlay --attachable my-overlay

# Containers on any swarm node can communicate
docker run --network my-overlay myapp
```

Used by: Docker Swarm services, Kubernetes (with CNI plugins).

---

### Q57. How do you expose a container's port?

```bash
# Map specific ports
docker run -p 8080:80 nginx           # Host 8080 → Container 80
docker run -p 127.0.0.1:8080:80 nginx # Bind to specific host IP
docker run -p 8080:80/tcp nginx       # Explicit TCP
docker run -p 53:53/udp dns-server    # UDP port

# Map all EXPOSE'd ports to random host ports
docker run -P nginx
docker port nginx-container           # See all port mappings
```

---

### Q58. How do containers communicate with each other?

```
Same custom network:
  Container A → http://container-b:8080  (by name)
  Container A → http://172.20.0.3:8080   (by IP)

Different networks:
  Connect container to both networks → can communicate
  OR use external service discovery

Container → Host:
  docker.host.internal  (Mac/Windows)
  172.17.0.1            (Linux default gateway)

Container → Internet:
  Docker's NAT masquerade → host's network
```

---

### Q59. What is `docker network inspect`?

```bash
docker network inspect bridge          # Default bridge details
docker network inspect mynet           # Custom network
# Shows: containers connected, IP assignments, subnet, gateway
docker network inspect --format='{{json .Containers}}' mynet
```

---

### Q60. How do you implement network segmentation in Docker?

```yaml
# docker-compose.yml — secure network segmentation
services:
  frontend:
    networks:
      - public-net    # Only on public network
  backend:
    networks:
      - public-net    # Can receive from frontend
      - private-net   # Can reach database
  database:
    networks:
      - private-net   # Only on private, internal network
    # database CANNOT be reached from frontend directly!

networks:
  public-net:
    driver: bridge
  private-net:
    driver: bridge
    internal: true    # No external internet access!
```

---

## 5. Volumes & Storage

### Q61. What are the three storage mount types?

| Type | Command | Managed by | Persists |
|------|---------|-----------|---------|
| **Volume** | `-v myvolume:/data` | Docker | ✅ Yes |
| **Bind mount** | `-v /host/path:/data` | User/OS | ✅ Yes (host) |
| **tmpfs** | `--tmpfs /data` | Memory | ❌ No |

---

### Q62. When to use volumes vs bind mounts?

| Scenario | Use |
|----------|-----|
| Database data | Volume |
| Log files | Volume |
| Dev hot-reload (code changes) | Bind mount |
| Config files | Bind mount (read-only) |
| Sensitive temp data | tmpfs |
| Sharing between containers | Volume |
| Sharing with host | Bind mount |

---

### Q63. How do you backup a Docker volume?

```bash
# Backup volume data
docker run --rm \
  -v my-database-volume:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar czf /backup/db-backup-$(date +%Y%m%d).tar.gz /data

# Restore volume data
docker run --rm \
  -v my-database-volume:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar xzf /backup/db-backup-20260524.tar.gz -C /

# Copy volume between hosts
docker save $(docker volume inspect --format='{{.Name}}' my-volume) | \
  ssh user@remote-host docker load
```

---

### Q64. How do you share data between containers?

```bash
# Method 1: Named volume
docker run -d -v shared-data:/shared app1
docker run -d -v shared-data:/shared app2
# Both see same data in /shared

# Method 2: Data container pattern (old method)
docker create -v /shared --name data-container busybox
docker run --volumes-from data-container app1
docker run --volumes-from data-container app2

# Method 3: Bind mount to same host path
docker run -v /host/shared:/shared app1
docker run -v /host/shared:/shared app2
```

---

### Q65. What is a volume driver?

Volume drivers enable **external storage backends** — NFS, AWS EBS, Azure Disk, etc.

```bash
# NFS volume
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/exports/data \
  nfs-volume

# AWS EBS (using rexray driver)
docker volume create \
  --driver rexray/ebs \
  --opt size=20 \
  my-ebs-volume
```

---

### Q66. How do you make a volume read-only?

```bash
# Read-only volume mount
docker run -v my-config:/etc/config:ro nginx

# In docker-compose.yml
volumes:
  - my-config:/etc/config:ro
  - ./nginx.conf:/etc/nginx/nginx.conf:ro
```

---

### Q67. What is `--volumes-from`?

Mounts volumes from another container (named or container ID).

```bash
# Create container with volumes
docker run -d -v /data --name source-container myapp

# Mount volumes from source container
docker run --volumes-from source-container ubuntu ls /data
```

---

### Q68. What happens to volume data when container is deleted?

```bash
# Named volume: data PERSISTS even after container deleted
docker run -v my-data:/app/data myapp
docker rm myapp                        # Container gone
docker volume ls                       # my-data still there!
docker run -v my-data:/app/data myapp2 # Data accessible in new container

# Anonymous volume: removed with --rm flag
docker run --rm -v /app/data myapp     # Volume deleted when container stops
docker run -v /app/data myapp          # Anonymous volume, manually cleanup needed

# Cleanup
docker volume prune                    # Remove unused volumes
docker rm -v mycontainer               # Remove container AND its volumes
```

---

### Q69. How do you find where a volume is stored on host?

```bash
docker volume inspect my-volume
# Output:
# "Mountpoint": "/var/lib/docker/volumes/my-volume/_data"

# List contents
ls /var/lib/docker/volumes/my-volume/_data

# All volume data location
ls /var/lib/docker/volumes/
```

---

### Q70. What is the difference between anonymous and named volumes?

| | Named Volume | Anonymous Volume |
|--|-------------|-----------------|
| Definition | `-v my-vol:/data` | `-v /data` |
| Name | User-defined | Random hash |
| Persistence | ✅ Survives `rm` | ❌ Removed with `--rm` |
| Reusability | ✅ Easy to reuse | ❌ Hard to reference |
| Best for | Production data | Temporary data |

---

## 6. Docker Compose

### Q71. What is `depends_on` in Docker Compose?

Controls **service startup order** — but does NOT wait for service to be ready, only for container to start.

```yaml
services:
  backend:
    depends_on:
      - database       # Starts database before backend
      # OR with health check (compose v2.1+):
      database:
        condition: service_healthy   # Wait until healthy

  database:
    image: postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
```

---

### Q72. What is the difference between `docker-compose up` and `docker-compose run`?

| Command | Purpose |
|---------|---------|
| `up` | Start all services defined in compose file |
| `run` | Start a ONE-OFF command in a new container |

```bash
docker compose up -d                   # Start all services
docker compose run --rm backend pytest # Run tests (one-off)
docker compose run --rm app sh         # Interactive shell
```

---

### Q73. How do you override docker-compose configuration?

```bash
# Docker Compose automatically merges:
# docker-compose.yml (base)
# docker-compose.override.yml (local overrides — auto-loaded in dev)

# Explicit merge
docker compose -f docker-compose.yml -f docker-compose.prod.yml up

# Environment-specific
docker compose --env-file .env.production up
```

```yaml
# docker-compose.yml (base)
services:
  app:
    image: myapp:latest
    environment:
      - LOG_LEVEL=info

# docker-compose.override.yml (dev overrides — auto-loaded)
services:
  app:
    volumes:
      - ./src:/app/src     # Hot reload
    environment:
      - LOG_LEVEL=debug    # Overrides base
```

---

### Q74. What is a Compose profile?

Profiles allow **selectively starting services** — some services only run in specific environments.

```yaml
services:
  app:
    image: myapp

  debug-tools:
    image: debug-utils
    profiles: ["debug"]     # Only starts with --profile debug

  monitoring:
    image: prometheus
    profiles: ["monitoring"]
```

```bash
docker compose up                          # Only starts app
docker compose --profile debug up          # Starts app + debug-tools
docker compose --profile monitoring up     # Starts app + monitoring
```

---

### Q75. How do you scale services in Docker Compose?

```bash
# Scale specific service
docker compose up --scale backend=3 -d
docker compose up --scale worker=5 -d

# Check scaled services
docker compose ps

# Note: Scaling services that expose fixed ports will fail!
# Use dynamic port mapping:
services:
  backend:
    ports:
      - "8080"    # NOT "8080:8080" — Docker assigns random host ports
```

---

### Q76. What is `docker compose config`?

Validates the compose file and **shows the merged configuration** (with all overrides applied).

```bash
docker compose config                    # Show merged config
docker compose config --quiet            # Only validate, no output
docker compose --env-file .env config    # With specific env file
```

---

### Q77. How do you use secrets in Docker Compose?

```yaml
# docker-compose.yml (Swarm mode)
services:
  app:
    secrets:
      - db_password
    environment:
      - DB_PASSWORD_FILE=/run/secrets/db_password

secrets:
  db_password:
    external: true     # Already created with docker secret create

# For local dev (file-based):
secrets:
  db_password:
    file: ./secrets/db_password.txt
```

```bash
echo "mypassword" | docker secret create db_password -
docker secret ls
```

---

### Q78. What is the difference between `restart: always` and `restart: unless-stopped`?

| Policy | After manual stop | After Docker restart | After OS reboot |
|--------|-----------------|---------------------|----------------|
| `always` | Restarts | Restarts | Restarts |
| `unless-stopped` | ❌ Stays stopped | ❌ Stays stopped | Restarts |
| `on-failure` | ❌ Only on error | ❌ Only on error | ❌ Only on error |

```yaml
services:
  app:
    restart: unless-stopped    # Recommended for most services
  db:
    restart: always            # Critical service — always restart
```

---

### Q79. How do you handle environment variables in Docker Compose?

```bash
# 1. .env file (auto-loaded — same directory as compose file)
DB_PASSWORD=secret
REDIS_PORT=6379

# 2. env_file in compose (load multiple env files)
services:
  app:
    env_file:
      - .env
      - .env.local    # Overrides .env

# 3. environment in compose
services:
  app:
    environment:
      - DB_HOST=database
      - DB_PORT=${DB_PORT:-5432}    # With default fallback

# 4. docker compose --env-file custom.env up
```

---

### Q80. What is `docker compose exec` vs `docker compose run`?

| Command | Container | New container | Use |
|---------|-----------|--------------|-----|
| `exec` | Existing running | ❌ | Debug running service |
| `run` | New (one-off) | ✅ | Run tests, migrations |

```bash
docker compose exec backend bash           # Shell in running backend
docker compose exec database psql -U admin # DB console in running db
docker compose run --rm backend pytest     # Run tests (new container)
docker compose run --rm app python manage.py migrate
```

---

## 7. Docker Swarm

### Q81. What is Docker Swarm?

Docker Swarm is Docker's native **container orchestration** system — turns multiple Docker hosts into a single virtual cluster.

```
Swarm cluster:
  Manager nodes: maintain cluster state, schedule services
  Worker nodes:  run containers (tasks)

Features:
  ✅ Service discovery
  ✅ Load balancing (VIP routing mesh)
  ✅ Rolling updates + rollback
  ✅ Secrets management
  ✅ Health monitoring + auto-restart
```

---

### Q82. What is the difference between a service and a task in Swarm?

```
Service = desired state definition (what you want)
  "Run 3 replicas of nginx"

Task = individual container instance (what runs)
  Task 1: nginx on node-1
  Task 2: nginx on node-2
  Task 3: nginx on node-3

Service → manages → Tasks (containers)
```

```bash
docker service create --replicas 3 --name web nginx
docker service ps web     # Shows all 3 tasks + which node
```

---

### Q83. What is a Swarm routing mesh?

The routing mesh ensures any request to any Swarm node on the published port is routed to a healthy container — regardless of which node the container is on.

```
Node-1 (has nginx task)  ← Request to node-1:80 → served ✅
Node-2 (no nginx task)   ← Request to node-2:80 → routed to node-1 ✅
Node-3 (no nginx task)   ← Request to node-3:80 → routed to node-1 ✅
```

---

### Q84. What are manager node requirements for Swarm HA?

Swarm uses Raft consensus — needs majority (quorum) of managers available.

| Manager nodes | Fault tolerance |
|--------------|----------------|
| 1 | 0 (no HA) |
| 3 | 1 node can fail |
| 5 | 2 nodes can fail |
| 7 | 3 nodes can fail |

**Recommended:** 3 or 5 manager nodes. More is NOT always better (slower consensus).

---

### Q85. How do rolling updates work in Swarm?

```bash
docker service update \
  --image nginx:1.25 \
  --update-parallelism 2 \     # Update 2 tasks at a time
  --update-delay 10s \         # Wait 10s between batches
  --update-failure-action rollback \  # Auto-rollback on failure
  my-service

# Monitor
docker service ps my-service   # Watch task states

# Manual rollback
docker service rollback my-service
```

---

### Q86. How are Swarm secrets different from environment variables?

| | Swarm Secrets | Environment Variables |
|--|--------------|----------------------|
| Encrypted at rest | ✅ Yes | ❌ No |
| Encrypted in transit | ✅ Yes | ❌ No |
| Visible in `docker inspect` | ❌ No | ✅ Yes |
| Available as file | ✅ `/run/secrets/name` | ❌ |
| Swarm required | ✅ Yes | ❌ No |

```bash
echo "secretpassword" | docker secret create db_pass -
docker service create \
  --secret db_pass \
  --env DB_PASS_FILE=/run/secrets/db_pass \
  myapp
# Container reads: cat /run/secrets/db_pass
```

---

### Q87. What is a global service vs replicated service?

| Type | Replicas | Use case |
|------|---------|---------|
| `replicated` | Specified number | Web servers, APIs |
| `global` | One per node | Log agents, monitoring |

```bash
docker service create --mode global fluentd      # One per node
docker service create --replicas 3 --mode replicated nginx  # 3 total
```

---

### Q88. How do you drain a Swarm node for maintenance?

```bash
# Drain: move all tasks off node, don't schedule new ones
docker node update --availability drain my-node-1

# Verify tasks moved
docker node ps my-node-1   # Should show Shutdown state

# Perform maintenance...

# Re-enable node
docker node update --availability active my-node-1
```

---

## 8. Security

### Q89. What are Docker capabilities?

Linux capabilities allow **fine-grained privilege control** — instead of all-or-nothing root.

```bash
# Drop ALL capabilities (most secure)
docker run --cap-drop ALL myapp

# Add back only what's needed
docker run \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \    # Bind to ports < 1024
  --cap-add CHOWN \               # Change file ownership
  myapp

# Common capabilities to drop in production
--cap-drop SYS_ADMIN
--cap-drop NET_ADMIN
--cap-drop SYS_PTRACE
```

---

### Q90. What is `--privileged` and when should you use it?

`--privileged` gives container **nearly full host root access** — all capabilities, all devices.

```bash
docker run --privileged myapp   # ⚠️ AVOID in production!
```

**Legitimate use cases:**
- Docker-in-Docker (DinD) — running Docker inside Docker
- Low-level system tools
- Hardware access

**Never use for:** web apps, APIs, databases.

---

### Q91. How do you prevent privilege escalation?

```bash
docker run \
  --security-opt=no-new-privileges \   # Process can't gain new privileges
  --user 1000:1000 \                   # Non-root user
  myapp
```

```dockerfile
# In Dockerfile — create and switch to non-root user
RUN useradd -r -u 1001 -g app appuser
USER appuser
```

---

### Q92. What is seccomp in Docker?

**Secure Computing Mode** — restricts which system calls a container can make.

```bash
# Docker's default seccomp profile blocks ~44 of ~300+ syscalls
docker run --security-opt seccomp=/path/to/profile.json myapp

# Disable seccomp (not recommended!)
docker run --security-opt seccomp=unconfined myapp
```

---

### Q93. How do you scan Docker images for vulnerabilities?

```bash
# Trivy (most popular)
trivy image myapp:latest
trivy image --severity HIGH,CRITICAL myapp:latest
trivy image --exit-code 1 --severity CRITICAL myapp:latest  # Fail CI

# Docker Scout (built into Docker)
docker scout cves myapp:latest
docker scout recommendations myapp:latest

# Snyk
snyk container test myapp:latest
```

---

### Q94. What are the security best practices for Dockerfiles?

```dockerfile
# ✅ Use specific version tags
FROM node:20.11.0-alpine3.19    # Not FROM node:latest

# ✅ Run as non-root
RUN adduser -D -u 1001 appuser
USER appuser

# ✅ Minimal base image
FROM alpine:3.19

# ✅ No secrets in layers
# ❌ WRONG:
RUN curl -H "Authorization: Bearer $TOKEN" https://api.example.com
# ✅ CORRECT:
RUN --mount=type=secret,id=token \
    TOKEN=$(cat /run/secrets/token) curl ...

# ✅ Read-only hint
VOLUME ["/tmp"]

# ✅ Remove unnecessary tools
RUN apt-get install -y curl && \
    apt-get purge -y curl && \     # Remove after use
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/*
```

---

### Q95. How does Docker handle user namespaces?

User namespaces map container UID/GID to different host UID/GID — so root inside container ≠ root on host.

```bash
# Enable user namespace remapping in Docker daemon
# /etc/docker/daemon.json
{
  "userns-remap": "default"
}

# Now container root (UID 0) maps to unprivileged user on host
docker run --rm -it ubuntu id
# uid=0(root) in container = uid=100000 on host
```

---

### Q96. What is Docker Bench for Security?

Open-source tool that checks Docker deployment against **CIS Docker Benchmark**.

```bash
docker run --rm -it \
  --net host \
  --pid host \
  --userns host \
  --cap-add audit_control \
  -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
  -v /etc:/etc:ro \
  -v /lib/systemd/system:/lib/systemd/system:ro \
  -v /usr/bin/containerd:/usr/bin/containerd:ro \
  -v /usr/bin/runc:/usr/bin/runc:ro \
  -v /usr/lib/systemd:/usr/lib/systemd:ro \
  -v /var/lib:/var/lib:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  docker/docker-bench-security
```

---

### Q97. How do you protect the Docker daemon socket?

```bash
# Docker socket = full root access to host!
# /var/run/docker.sock should NEVER be mounted in production containers

# If needed, use socket proxy (Tecnativa)
docker run \
  -d \
  --name socket-proxy \
  -v /var/run/docker.sock:/var/run/docker.sock \
  tecnativa/docker-socket-proxy

# Configure allowed operations only
-e CONTAINERS=1      # Allow container list/inspect
-e IMAGES=0          # Deny image operations
-e NETWORKS=0        # Deny network operations
```

---

### Q98. What is Docker Content Trust?

DCT ensures images are **signed and verified** before use — prevents tampered images.

```bash
# Enable globally
export DOCKER_CONTENT_TRUST=1

# Sign image on push (requires Notary)
docker push myapp:v1    # Automatically signed

# Verify on pull
docker pull myapp:v1    # Verified before pulling

# Disable for specific command
DOCKER_CONTENT_TRUST=0 docker pull unsigned-image:latest
```

---

## 9. Advanced Concepts

### Q99. What is BuildKit and how is it different from legacy builder?

BuildKit is Docker's **next-generation build engine** — faster, more secure, more features.

```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1
docker build .

# Or permanently in daemon.json
{ "features": { "buildkit": true } }

# BuildKit features:
# ✅ Parallel stage execution
# ✅ Better caching
# ✅ Secret mounts (--mount=type=secret)
# ✅ SSH agent forwarding
# ✅ Inline cache
# ✅ Better output/progress
```

---

### Q100. What is `docker buildx`?

Extended build capabilities — multi-platform builds, advanced caching.

```bash
# Build for multiple architectures
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myapp:multi \
  --push .

# Create builder with more resources
docker buildx create --name mybuilder --use
docker buildx inspect --bootstrap
```

---

### Q101. What is the Union Filesystem (OverlayFS)?

Docker uses **OverlayFS** to present multiple read-only layers + one writable layer as a single unified filesystem.

```
UpperDir (container writable layer):  file3.txt (new/modified)
LowerDir (image read-only layers):    file1.txt, file2.txt
Result (merged view):                 file1.txt, file2.txt, file3.txt

When you modify file1.txt:
  Copy-on-Write: file1.txt copied to UpperDir, then modified
  LowerDir: original file1.txt unchanged
  Container sees: modified file1.txt from UpperDir
```

---

### Q102. How does Docker handle logging?

```bash
# Logging drivers
docker run --log-driver=json-file \       # Default (stored on host)
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  nginx

# Available drivers:
# json-file    → Default, local JSON files
# syslog       → System syslog
# journald     → systemd journal
# gelf         → Graylog
# fluentd      → Fluentd
# awslogs      → AWS CloudWatch
# splunk       → Splunk
# none         → Disable logging

# Daemon-level default (daemon.json)
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

---

### Q103. What is Docker-in-Docker (DinD)?

Running Docker inside a Docker container — used for CI/CD pipelines.

```bash
# Method 1: True DinD (--privileged)
docker run --privileged -d --name dind docker:dind

# Method 2: Mount Docker socket (safer for CI)
docker run \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(which docker):/usr/bin/docker \
  jenkins-agent

# Method 3: Kaniko (no Docker daemon needed)
docker run --rm \
  -v $(pwd):/workspace \
  gcr.io/kaniko-project/executor:latest \
  --context /workspace \
  --destination myapp:latest
```

---

### Q104. What is image digest?

A **content-addressable hash** — uniquely identifies an exact image version regardless of tag.

```bash
# Pull by digest (immutable — always the same image)
docker pull nginx@sha256:abc123def456...

# Get digest of an image
docker image inspect --format='{{.RepoDigests}}' nginx:alpine

# Tags can be reassigned; digests never change
```

---

### Q105. What is `docker system df`?

Shows **disk usage** by Docker components.

```bash
docker system df
# TYPE            TOTAL  ACTIVE  SIZE    RECLAIMABLE
# Images          15     3       4.2GB   3.1GB (74%)
# Containers      8      3       250MB   180MB (72%)
# Volumes         12     5       5.6GB   2.1GB (37%)
# Build Cache     -      -       1.2GB   1.2GB

docker system df -v     # Verbose — list each item
```

---

### Q106. What is `docker events`?

Real-time stream of Docker daemon events — container lifecycle, network changes, volume operations.

```bash
docker events                                 # All events
docker events --filter type=container         # Container events only
docker events --filter event=start            # Start events only
docker events --filter container=myapp        # Specific container
docker events --since 1h                      # Last hour
docker events --format '{{json .}}'           # JSON format
```

---

### Q107. How does Docker container networking work internally?

```
Container network (172.17.0.0/16):
  Container A: 172.17.0.2
  Container B: 172.17.0.3
  
  veth pair: container network interface ↔ host veth
  
  docker0 bridge: 172.17.0.1 (default gateway for containers)
  
  Outbound traffic: Container → docker0 → NAT (iptables) → host → internet
  Inbound traffic (port mapped): host:8080 → iptables DNAT → 172.17.0.2:80
```

---

### Q108. What is `docker checkpoint`?

Experimental feature — **save container state** (CRIU) and restore it later (Checkpoint/Restore).

```bash
docker checkpoint create mycontainer checkpoint1
docker start --checkpoint checkpoint1 mycontainer
# Container resumes exactly where it paused!
```

---

### Q109. How do you monitor Docker with Prometheus?

```yaml
# docker-compose.monitoring.yml
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
```

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
```

---

### Q110. What is `docker manifest`?

Multi-platform image manifests — one tag that works on multiple architectures.

```bash
docker manifest inspect nginx:alpine
# Shows: linux/amd64, linux/arm64, linux/arm/v7 variants

# Create multi-platform manifest
docker manifest create myapp:v1 \
  myapp:v1-amd64 \
  myapp:v1-arm64

docker manifest push myapp:v1
```

---

### Q111. How does Docker implement container isolation?

```
Container isolation stack:
  namespaces:  PID, NET, MNT, UTS, IPC, USER
  cgroups:     CPU, memory, I/O, network limits
  seccomp:     System call filtering
  AppArmor:    Mandatory access control profiles
  SELinux:     Mandatory access control (RHEL-based)
  Capabilities: Fine-grained privilege control
  OverlayFS:   Filesystem isolation
```

---

### Q112. What is the OCI (Open Container Initiative)?

OCI defines **industry standards** for containers:
- **OCI Image Spec** — standard format for container images
- **OCI Runtime Spec** — standard interface for container runtimes

This means Docker images can run with `runc`, `kata-containers`, `gVisor`, etc.

```bash
# Docker uses runc by default
docker info | grep "Default Runtime"

# Change runtime to gVisor (stronger isolation)
docker run --runtime=runsc myapp      # gVisor (Google)
docker run --runtime=kata myapp       # Kata containers (VM-based)
```

---

### Q113. What is `docker plugin`?

Docker plugins extend functionality — volume drivers, network plugins, log drivers.

```bash
docker plugin install <plugin-name>   # Install plugin
docker plugin ls                       # List plugins
docker plugin enable <plugin>          # Enable
docker plugin disable <plugin>         # Disable
docker plugin rm <plugin>              # Remove

# Example: REX-Ray for cloud volumes
docker plugin install rexray/ebs
docker volume create --driver rexray/ebs --opt size=20 myvol
```

---

### Q114. How do you implement health checks for dependencies in Compose?

```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy

  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s

  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 3
```

---

### Q115. What is the difference between `docker compose up` and `docker stack deploy`?

| Feature | `docker compose up` | `docker stack deploy` |
|---------|--------------------|-----------------------|
| Requires Swarm | ❌ No | ✅ Yes |
| Secrets | File-based | Swarm secrets |
| Scaling | `--scale` flag | `deploy.replicas` |
| Rolling updates | ❌ Manual | ✅ Automatic |
| Production use | Dev/testing | Production |

```bash
# Compose (dev)
docker compose up -d

# Swarm stack (production)
docker stack deploy -c docker-compose.yml myapp
```

---

## 10. Scenario-Based Questions

### Scenario 1: Container exits immediately after `docker run`

**Problem:** Container starts and stops within seconds. Exit code 1.

```bash
#!/bin/bash
# Step 1: Check exit code and last state
docker ps -a | grep mycontainer
docker inspect mycontainer --format='{{.State.ExitCode}}'

# Step 2: Check logs
docker logs mycontainer

# Step 3: Run interactively to debug
docker run -it --entrypoint /bin/sh myapp     # Override entrypoint
docker run -it myapp sh                       # If entrypoint is exec

# Step 4: Override CMD for debugging
docker run -it myapp sleep 3600               # Keep alive for inspection

# Common causes:
# - Application crashing (check logs)
# - Missing environment variable
# - Config file not found
# - Database connection refused on startup
```

---

### Scenario 2: Container running but app not accessible

**Problem:** `docker run -p 8080:80 nginx` — browser shows "connection refused".

```bash
#!/bin/bash
# Step 1: Verify container is running
docker ps | grep nginx

# Step 2: Check port mapping
docker port mycontainer

# Step 3: Check if app is listening inside container
docker exec -it mycontainer ss -tlnp
docker exec -it mycontainer curl localhost:80

# Step 4: Check host firewall
sudo iptables -L -n | grep 8080
sudo ufw status

# Step 5: Check if binding to 127.0.0.1 (not accessible from outside)
docker run -p 0.0.0.0:8080:80 nginx    # Explicitly bind to all interfaces
```

---

### Scenario 3: Docker build failing — no space left on device

```bash
#!/bin/bash
# Step 1: Check disk usage
df -h
docker system df

# Step 2: Clean up
docker system prune -a            # Remove all unused objects
docker volume prune               # Remove unused volumes (careful!)
docker image prune -a             # Remove all unused images

# Step 3: Identify large layers
docker image history myapp:latest

# Step 4: Long-term fix
# - Increase disk space
# - Add .dockerignore
# - Use multi-stage builds
# - Clean apt cache in Dockerfile
```

---

### Scenario 4: Two containers can't communicate

**Problem:** frontend container can't reach backend container.

```bash
#!/bin/bash
# Step 1: Check if on same network
docker inspect frontend --format='{{json .NetworkSettings.Networks}}'
docker inspect backend --format='{{json .NetworkSettings.Networks}}'

# Step 2: Test connectivity
docker exec -it frontend ping backend        # Name resolution
docker exec -it frontend ping 172.20.0.3    # Direct IP

# Step 3: Check if on default bridge (no DNS!)
# Fix: Use custom network
docker network create app-net
docker network connect app-net frontend
docker network connect app-net backend
docker exec -it frontend curl http://backend:8080  # Now works!

# Step 4: In docker-compose — ensure same network
services:
  frontend:
    networks: [app-net]
  backend:
    networks: [app-net]
networks:
  app-net:
```

---

### Scenario 5: Container using too much memory — getting OOMKilled

**Problem:** Container killed with exit code 137 (SIGKILL from OOM).

```bash
#!/bin/bash
# Step 1: Verify OOM kill
docker inspect mycontainer --format='{{.State.OOMKilled}}'
# true = OOM killed

# Step 2: Check current limits
docker inspect mycontainer --format='{{.HostConfig.Memory}}'

# Step 3: Check actual usage
docker stats --no-stream mycontainer

# Step 4: Fix
# Option A: Increase memory limit
docker run --memory=2g myapp

# Option B: Fix memory leak in application
# Profile app with: docker exec -it mycontainer <profiler>

# Option C: Set swap (use carefully)
docker run --memory=512m --memory-swap=1g myapp

# Step 5: Monitor
docker stats --format "table {{.Name}}\t{{.MemUsage}}\t{{.MemPerc}}"
```

---

### Scenario 6: Docker image taking too long to build

**Problem:** `docker build` takes 10+ minutes every time.

```bash
#!/bin/bash
# Step 1: Check what's invalidating cache
# Most common: COPY . . too early

# Fix: Restructure Dockerfile for better caching
# ❌ BAD (cache busted on every code change):
COPY . .
RUN npm install

# ✅ GOOD (npm install cached unless package.json changes):
COPY package*.json ./
RUN npm install
COPY . .

# Step 2: Use BuildKit
DOCKER_BUILDKIT=1 docker build .

# Step 3: Use remote cache
docker build \
  --cache-from myapp:latest \    # Use prod image as cache
  -t myapp:new .

# Step 4: Check .dockerignore (large files in build context?)
ls -la .dockerignore
du -sh .                        # Total context size
```

---

### Scenario 7: Production container shows stale data after image update

**Problem:** Updated image, restarted container, but old data still showing.

```bash
#!/bin/bash
# Step 1: Verify new image is actually being used
docker inspect mycontainer --format='{{.Image}}'
docker images myapp --format='{{.ID}} {{.Tag}}'

# Step 2: Check if old container is running (not new one)
docker ps | grep myapp
# Is it using the NEW image?

# Step 3: Pull latest image
docker pull myapp:latest

# Step 4: Remove old container and start fresh
docker stop myapp-container
docker rm myapp-container
docker run -d --name myapp-container myapp:latest

# Step 5: In docker-compose
docker compose pull          # Pull latest images
docker compose up -d         # Recreate with new images

# Root cause: docker compose up only recreates if image hash changed!
# Force recreate:
docker compose up -d --force-recreate
```

---

### Scenario 8: `docker-compose up` failing with dependency not ready

**Problem:** Backend starts before database is ready → connection refused.

```yaml
# Fix: Use health checks with condition
services:
  backend:
    depends_on:
      database:
        condition: service_healthy    # Wait for healthy!

  database:
    image: mysql:8.0
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 5s
      retries: 10
      start_period: 30s
```

```bash
#!/bin/bash
# Application-level retry (belt and suspenders approach)
# In your app startup code, retry connection with backoff
# retries=5, wait=2s between attempts

# Or use wait-for-it script
COPY wait-for-it.sh /wait-for-it.sh
RUN chmod +x /wait-for-it.sh
CMD ["/wait-for-it.sh", "database:3306", "--", "java", "-jar", "app.jar"]
```

---

### Scenario 9: Volume data lost after container restart

**Problem:** MySQL data gone after `docker restart`.

```bash
#!/bin/bash
# Step 1: Check if volume was mounted
docker inspect mycontainer | grep -A 10 "Mounts"
# If empty or bind mount to /tmp → data loss expected!

# Step 2: Use named volume (ALWAYS for databases!)
# ❌ WRONG:
docker run -d mysql:8.0

# ✅ CORRECT:
docker run -d \
  -v mysql-data:/var/lib/mysql \    # Named volume persists!
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0

# Step 3: Verify volume exists
docker volume ls | grep mysql-data
docker volume inspect mysql-data

# Step 4: docker-compose.yml
services:
  database:
    image: mysql:8.0
    volumes:
      - mysql-data:/var/lib/mysql    # Named volume

volumes:
  mysql-data:                        # Declare it!
```

---

### Scenario 10: Dockerfile secret accidentally committed to Git

**Problem:** `docker build --build-arg API_KEY=secret .` and `ARG API_KEY` visible in `docker history`.

```bash
#!/bin/bash
# Step 1: Verify the exposure
docker history myapp:latest --no-trunc | grep API_KEY
# If visible → rebuild WITHOUT the secret in layers!

# Step 2: Fix using BuildKit secrets (no layer exposure)
# syntax=docker/dockerfile:1
FROM node:20-alpine

RUN --mount=type=secret,id=api_key \
    API_KEY=$(cat /run/secrets/api_key) \
    npm config set ...

# Build command:
DOCKER_BUILDKIT=1 docker build \
  --secret id=api_key,src=./api_key.txt \
  -t myapp:clean .

# Step 3: Rotate the exposed secret immediately!
# Step 4: Rebuild and repush ALL affected image tags
# Step 5: Audit registry — who pulled the old image?
```

---

### Scenario 11: Container CPU at 100% — service degraded

```bash
#!/bin/bash
# Step 1: Identify the container
docker stats --no-stream

# Step 2: Check processes inside
docker exec -it mycontainer top
docker top mycontainer

# Step 3: Limit CPU to protect other containers
docker update --cpus=2.0 mycontainer     # Limit to 2 cores

# Step 4: Investigate the cause
docker exec -it mycontainer sh
# Check for infinite loops, deadlocks, runaway processes

# Step 5: Set CPU limits at startup
docker run --cpus=1.5 --cpu-shares=512 myapp

# Step 6: In docker-compose.yml
deploy:
  resources:
    limits:
      cpus: "1.5"
      memory: 512M
```

---

### Scenario 12: "Image not found" error in CI/CD pipeline

**Problem:** `docker pull myapp:latest` fails with "manifest not found".

```bash
#!/bin/bash
# Step 1: Verify image exists in registry
docker search myregistry.com/myapp
docker manifest inspect myregistry.com/myapp:latest

# Step 2: Check authentication
docker login myregistry.com
echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin

# Step 3: Check tag name (typo? wrong case?)
docker images myapp --format "{{.Tag}}"

# Step 4: Check if tag was pushed
docker pull myapp:latest   # explicit pull after push

# Step 5: In CI/CD — use specific tag not latest
docker build -t myapp:$GIT_SHA .
docker push myapp:$GIT_SHA
# Then deploy with: docker run myapp:$GIT_SHA
```

---

### Scenario 13: Docker Compose network conflict — subnet overlap

**Problem:** Error: "Pool overlaps with other one on this address space".

```bash
#!/bin/bash
# Step 1: Check existing networks
docker network ls
docker network inspect bridge | grep Subnet

# Step 2: Remove conflicting network
docker network rm conflicting-network

# Step 3: Or specify custom subnet in compose
networks:
  app-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.25.0.0/16   # Explicitly set non-conflicting subnet

# Step 4: Clean all unused networks
docker network prune
```

---

### Scenario 14: Container log files filling up disk

```bash
#!/bin/bash
# Step 1: Check log size
du -sh /var/lib/docker/containers/*/
ls -lh /var/lib/docker/containers/<container-id>/*.log

# Step 2: Configure log rotation (per container)
docker run \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myapp

# Step 3: Set globally in daemon.json
# /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
sudo systemctl restart docker

# Step 4: In docker-compose.yml
services:
  app:
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

---

### Scenario 15: Slow `docker push` / `docker pull`

```bash
#!/bin/bash
# Step 1: Check image size (smaller = faster)
docker images myapp --format "{{.Size}}"

# Step 2: Check which layers are already cached on registry
# Only new/changed layers are pushed

# Step 3: Optimize image to reduce push size
# - Multi-stage build
# - Use alpine base
# - Better .dockerignore

# Step 4: Use registry closer to build machine (same region)
# - Build in us-east-1 → push to ECR us-east-1

# Step 5: Use registry mirrors
# daemon.json:
{
  "registry-mirrors": ["https://mirror.gcr.io"]
}

# Step 6: Parallel push with BuildKit
DOCKER_BUILDKIT=1 docker build --push .
```

---

### Scenario 16: Rolling update causing downtime in production

```bash
#!/bin/bash
# Problem: During docker service update, service is briefly unavailable

# Fix 1: Ensure replicas > 1
docker service scale myapp=3          # Need multiple replicas

# Fix 2: Configure proper update settings
docker service update \
  --update-parallelism 1 \            # Update 1 at a time
  --update-delay 30s \                # Wait 30s between updates
  --update-failure-action rollback \  # Auto-rollback on failure
  --update-monitor 60s \              # Monitor for 60s after update
  --image myapp:v2 \
  myapp

# Fix 3: Readiness probe (health check)
docker service create \
  --health-cmd "curl -f http://localhost/health || exit 1" \
  --health-interval 10s \
  --health-retries 3 \
  --health-start-period 30s \         # Give app time to start
  myapp

# Fix 4: Set minReplicas during update (Swarm --update-max-failure-ratio)
docker service update \
  --update-max-failure-ratio 0.1 \    # 10% failure = rollback
  myapp
```

---

### Scenario 17: Debugging a container that crashes immediately

```bash
#!/bin/bash
# Container exits before you can exec into it
# Method 1: Override entrypoint
docker run -it --entrypoint /bin/sh myapp
docker run -it --entrypoint bash myapp

# Method 2: Override CMD
docker run -it myapp sh

# Method 3: Check previous logs
docker logs myapp-container             # Even stopped container has logs
docker inspect myapp-container --format='{{.State.ExitCode}}'

# Method 4: Start with sleep, then investigate
docker run -d --name debug-app \
  --entrypoint sleep myapp 3600         # Keep alive for 1 hour
docker exec -it debug-app sh            # Investigate

# Method 5: Share volumes to inspect files
docker run -d --name debug-app myapp
docker cp debug-app:/app/config.json .  # Extract config
```

---

### Scenario 18: Multi-environment deployment with Docker Compose

**Problem:** Same compose file needs to work for dev, staging, prod.

```yaml
# docker-compose.yml (base — same for all envs)
services:
  app:
    image: myapp:${APP_VERSION:-latest}
    environment:
      - DB_HOST=database
      - LOG_LEVEL=${LOG_LEVEL:-info}
  database:
    image: mysql:8.0
    volumes:
      - db-data:/var/lib/mysql

volumes:
  db-data:
```

```yaml
# docker-compose.dev.yml (dev overrides)
services:
  app:
    build: .              # Build locally in dev
    volumes:
      - ./src:/app/src    # Hot-reload
    environment:
      - LOG_LEVEL=debug
  database:
    ports:
      - "3306:3306"       # Expose in dev only
```

```yaml
# docker-compose.prod.yml (prod overrides)
services:
  app:
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
```

```bash
# Dev
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

### Scenario 19: Container accessing host services (database on host)

```bash
#!/bin/bash
# Problem: Container needs to reach service running on host

# Mac/Windows: use host.docker.internal
docker run -e DB_HOST=host.docker.internal myapp

# Linux: use host network or host-gateway
docker run --network host myapp          # Container on host network

# OR: Add host-gateway
docker run --add-host=host.docker.internal:host-gateway myapp

# In docker-compose.yml
services:
  app:
    extra_hosts:
      - "host.docker.internal:host-gateway"
    environment:
      - DB_HOST=host.docker.internal
```

---

### Scenario 20: Image works locally but fails in CI pipeline

```bash
#!/bin/bash
# Common causes:
# 1. Different architecture (amd64 vs arm64)
docker buildx build --platform linux/amd64 -t myapp .   # Force amd64

# 2. Different Docker version
docker version                              # Check local version
# Pin Docker version in CI/CD

# 3. Missing environment variables
docker run -e REQUIRED_VAR=value myapp      # Add missing vars

# 4. Registry authentication
echo $CI_REGISTRY_PASSWORD | docker login \
  -u $CI_REGISTRY_USER \
  --password-stdin $CI_REGISTRY

# 5. Build context different
# Add comprehensive .dockerignore
# Check what's in build context:
docker build --progress=plain . 2>&1 | head -50
```

---

### Scenario 21: Data volume accidentally deleted

```bash
#!/bin/bash
# Prevention is key — implement backup strategy!

# If deleted accidentally:
# Named volumes: check if still present
docker volume ls | grep my-important-data

# If truly gone — restore from backup
docker run --rm \
  -v my-data-volume:/data \
  -v /backup:/backup \
  ubuntu tar xzf /backup/latest.tar.gz -C /

# Prevention:
# 1. Regular automated backups
# 2. Never use docker system prune -a --volumes without confirmation
# 3. Use external storage (AWS EBS, NFS)
# 4. Mount volumes as read-only where possible
# 5. Monitor volume usage
```

---

### Scenario 22: Container needs to access GPU

```bash
#!/bin/bash
# Install NVIDIA Container Toolkit first
# Then:
docker run \
  --gpus all \                          # All GPUs
  --gpus '"device=0"' \                # Specific GPU
  --gpus "all,capabilities=compute,utility" \
  nvidia/cuda:12.0-base \
  nvidia-smi

# In docker-compose.yml
services:
  ml-training:
    image: my-ml-app
    deploy:
      resources:
        reservations:
          devices:
          - driver: nvidia
            count: 1
            capabilities: [gpu]
```

---

### Scenario 23: Docker Swarm service not starting

```bash
#!/bin/bash
docker service ls                         # Service listed?
docker service ps my-service              # Task state?
docker service ps --no-trunc my-service   # Full error message!

# Common states:
# Rejected  = node doesn't meet constraint (taint, resource)
# Failed    = container exited with error
# Shutdown  = intentional (drain, update)

# Debug specific task
docker inspect <task-id>

# Check node resources
docker node ls
docker node inspect <node> --format='{{json .Status}}'

# Check if image is accessible from all nodes
# Worker nodes need access to registry!
```

---

### Scenario 24: Implementing blue-green deployment with Docker

```bash
#!/bin/bash
# Step 1: Deploy green (new version)
docker run -d \
  --name app-green \
  --network app-net \
  myapp:v2

# Step 2: Verify green is healthy
sleep 30
docker exec app-green curl localhost:8080/health

# Step 3: Switch nginx/load balancer to green
# nginx upstream:
docker exec nginx-lb nginx -s reload    # After updating upstream config

# Step 4: Monitor for errors
docker logs -f app-green --tail 100

# Step 5: Remove blue if green is good
docker stop app-blue && docker rm app-blue

# Step 6: If green has issues — switch back (instant rollback)
docker exec nginx-lb nginx -s reload    # Point back to blue
docker stop app-green && docker rm app-green
```

---

### Scenario 25: Docker container consuming too much disk space

```bash
#!/bin/bash
# Step 1: Find space usage
docker system df
docker system df -v           # Verbose

# Step 2: Largest images
docker images --format "{{.Size}}\t{{.Repository}}:{{.Tag}}" | sort -rh | head -10

# Step 3: Largest containers (writable layer)
docker ps -a --format "{{.Names}}" | \
  xargs -I {} docker inspect {} --format='{{.Name}} {{.SizeRootFs}}'

# Step 4: Clean up strategically
docker container prune        # Remove stopped containers
docker image prune            # Remove dangling images
docker image prune -a         # Remove all unused images
docker volume prune           # Remove unused volumes
docker network prune          # Remove unused networks

# Step 5: Full cleanup (use carefully!)
docker system prune -a --volumes

# Step 6: Limit writable layer size with storage limits
docker run --storage-opt size=10G myapp   # Limit container storage
```

---

### Scenario 26: Running database migrations before app start

```bash
#!/bin/bash
# Option 1: Init container (Kubernetes) / entrypoint script (Docker)

# entrypoint.sh
#!/bin/bash
set -e
echo "Running database migrations..."
python manage.py migrate

echo "Starting application..."
exec "$@"

# Dockerfile
COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["gunicorn", "app:app"]
```

```yaml
# Option 2: docker-compose — separate migration service
services:
  migrations:
    image: myapp:latest
    command: python manage.py migrate
    depends_on:
      database:
        condition: service_healthy
    restart: "no"           # Run once and exit

  app:
    image: myapp:latest
    depends_on:
      migrations:
        condition: service_completed_successfully
```

---

### Scenario 27: Handling secrets in Docker production deployment

```bash
#!/bin/bash
# ❌ BAD — visible in process list, history, logs
docker run -e DB_PASSWORD=secret myapp

# ✅ Option 1: Docker Swarm secrets
echo "secretpass" | docker secret create db_password -
docker service create \
  --secret db_password \
  --env DB_PASSWORD_FILE=/run/secrets/db_password \
  myapp

# ✅ Option 2: External secrets manager (HashiCorp Vault)
# App reads secret at runtime from Vault
docker run \
  -e VAULT_ADDR=http://vault:8200 \
  -e VAULT_TOKEN=${VAULT_TOKEN} \
  myapp

# ✅ Option 3: AWS Secrets Manager + IAM role
# Container uses IAM role → reads secret from Secrets Manager at runtime
# No secrets in environment at all!

# ✅ Option 4: Kubernetes secrets (if using K8s)
kubectl create secret generic db-secret --from-literal=password=secret
# Mount as volume or env var in pod spec
```

---

### Scenario 28: Docker build fails — package installation error

```bash
#!/bin/bash
# Common: apt-get fails because of outdated package list cache

# Fix: Always update before install
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      curl wget && \
    rm -rf /var/lib/apt/lists/*    # MUST be in same RUN!

# If package not found:
RUN apt-cache search <package>     # Find correct package name

# Network issues during build:
docker build --network host .      # Use host networking

# Retry logic for flaky downloads:
RUN for i in 1 2 3; do \
      apt-get install -y mypackage && break || sleep 5; \
    done
```

---

### Scenario 29: Multi-container app — one service restarting loop

```bash
#!/bin/bash
# Step 1: Identify the crashing service
docker compose ps
docker compose logs crashing-service --tail 50

# Step 2: Check if dependency is ready
docker compose logs database | grep "ready for connections"

# Step 3: Temporarily disable restart to debug
services:
  crashing-service:
    restart: "no"         # Disable restart temporarily

docker compose up -d crashing-service
docker logs crashing-service   # Read error before it restarts

# Step 4: Common causes
# - DB not ready (add healthcheck + condition)
# - Missing env var
# - Port already in use: docker compose ps && netstat -tlnp
# - Volume permission issue: docker exec -it service ls -la /app
```

---

### Scenario 30: Implement Docker layer caching in GitHub Actions

```yaml
# .github/workflows/docker.yml
name: Docker Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push with cache
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            myuser/myapp:latest
            myuser/myapp:${{ github.sha }}
          cache-from: type=gha              # GitHub Actions cache
          cache-to: type=gha,mode=max       # Store in GHA cache
          # OR use registry cache:
          # cache-from: type=registry,ref=myuser/myapp:buildcache
          # cache-to: type=registry,ref=myuser/myapp:buildcache,mode=max
```

---

### Scenarios 31–60: Quick Scenario Reference

**31. Container can't resolve external DNS:**
```bash
docker exec mycontainer nslookup google.com
# Fix: Check host DNS, or add --dns flag
docker run --dns=8.8.8.8 myapp
```

**32. `docker exec` — "executable not found":**
```bash
# Alpine image doesn't have bash!
docker exec -it mycontainer /bin/sh    # Use sh instead of bash
docker exec -it mycontainer ash        # Alpine's shell
```

**33. Container restart loop — liveness probe failing:**
```dockerfile
# Probe too aggressive — app starting slowly
HEALTHCHECK --start-period=60s \    # Give 60s before health checks start
            --interval=30s \
            --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1
```

**34. "Permission denied" when writing to mounted volume:**
```bash
# Fix: Match UID/GID between container and host
docker run --user $(id -u):$(id -g) -v /host/path:/data myapp
# OR in Dockerfile:
RUN chown -R 1001:1001 /app/data
USER 1001
```

**35. `docker compose` changes not applying:**
```bash
docker compose up -d --force-recreate    # Force container recreation
docker compose down && docker compose up -d  # Full restart
docker compose config                     # Verify config is correct
```

**36. Container taking long to stop (SIGTERM ignored):**
```dockerfile
# App not handling SIGTERM
# Fix: Use exec form CMD (receives signals directly)
# ❌ CMD node server.js          (shell form — node doesn't get signal)
# ✅ CMD ["node", "server.js"]   (exec form — node receives SIGTERM)
```

**37. Old Docker Compose syntax warning:**
```yaml
# Remove obsolete 'version' field
# ❌ version: "3.9"    ← deprecated in newer compose
# ✅ (no version field) ← just remove it
services:
  app:
    image: nginx
```

**38. Image always tagged as latest (bad practice):**
```bash
# Use git commit hash or semantic version
docker build -t myapp:$(git rev-parse --short HEAD) .
docker build -t myapp:1.2.3 .
# Never deploy with :latest in production!
```

**39. Container network performance issue:**
```bash
# Option 1: Use host network (no NAT overhead)
docker run --network host myapp

# Option 2: Use macvlan (own IP on LAN)
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 mynet
```

**40. Docker Compose v1 vs v2 command difference:**
```bash
# Old (docker-compose — deprecated)
docker-compose up

# New (docker compose — current)
docker compose up
```

**41. Concurrent builds causing conflicts:**
```bash
# Tag with unique ID for concurrent builds
docker build -t myapp:${BUILD_NUMBER}-${GIT_SHA} .
# Never push :latest from parallel builds simultaneously
```

**42. Container clock drift:**
```bash
# Containers inherit host clock
# If clock is wrong — fix host: chronyc makestep
# Or mount host time:
docker run -v /etc/localtime:/etc/localtime:ro myapp
```

**43. "Too many open files" error:**
```bash
docker run --ulimit nofile=65536:65536 myapp
# Or in daemon.json:
{ "default-ulimits": { "nofile": { "Hard": 64000, "Name": "nofile", "Soft": 64000 } } }
```

**44. Image tag not found after push:**
```bash
docker push myapp:v1
docker pull myapp:v1    # Always verify after push!
# Check if login is to correct registry
docker info | grep "Registry"
```

**45. Docker daemon not starting:**
```bash
sudo systemctl status docker
sudo journalctl -u docker -n 50
# Check /var/lib/docker — disk full?
# Check daemon.json for syntax errors
sudo dockerd --validate-config
```

**46. Container timezone wrong:**
```bash
docker run \
  -e TZ=Asia/Kolkata \
  -v /etc/localtime:/etc/localtime:ro \
  myapp
```

**47. Large build context (slow build start):**
```bash
# Check context size
du -sh .
# Fix: Comprehensive .dockerignore
echo "node_modules" >> .dockerignore
echo ".git" >> .dockerignore
echo "*.log" >> .dockerignore
```

**48. Docker Compose service name conflict:**
```bash
# Services with same name in different compose files
# Use project name to isolate:
docker compose -p project1 up
docker compose -p project2 -f other-compose.yml up
```

**49. Troubleshoot network latency between containers:**
```bash
docker exec app1 ping -c 10 app2      # Measure latency
docker exec app1 iperf3 -c app2       # Bandwidth test
# Use host network for lowest latency
```

**50. Container environment variable not set:**
```bash
docker exec mycontainer env | grep MY_VAR   # Check if set
docker inspect mycontainer | grep -A 20 "Env"
# Fix: Verify -e flag or env_file
docker run -e MY_VAR=value myapp
```

**51. Build fails on M1/M2 Mac (ARM):**
```bash
# Force AMD64 build
docker buildx build --platform linux/amd64 -t myapp .
# Or multi-platform:
docker buildx build --platform linux/amd64,linux/arm64 -t myapp --push .
```

**52. Container keeps getting killed by OOM:**
```bash
dmesg | grep -i "oom-killer"        # Check host OOM killer
docker stats --no-stream            # Check memory usage
docker run --memory=2g myapp        # Increase limit
# Fix memory leak in application
```

**53. Pushing to AWS ECR fails:**
```bash
# Re-authenticate (ECR tokens expire in 12 hours!)
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS \
  --password-stdin \
  123456789.dkr.ecr.us-east-1.amazonaws.com

# Check IAM permissions (need ecr:GetAuthorizationToken + push permissions)
```

**54. Container IP keeps changing:**
```bash
# Use container name (DNS) instead of IP
# IP changes on restart! Name resolution is stable.
docker run --name mydb mysql
docker run -e DB_HOST=mydb myapp    # Use name, not IP
```

**55. Diff between two Docker images:**
```bash
# Option 1: Container diff
docker run --rm alpine cat /etc/os-release > old.txt
docker run --rm alpine:edge cat /etc/os-release > new.txt
diff old.txt new.txt

# Option 2: dive tool (layer inspection)
dive myapp:v1
dive myapp:v2
```

**56. Container auto-starting on boot:**
```bash
# Enable: restart policy
docker run --restart=always myapp
# Verify Docker service auto-starts
sudo systemctl enable docker
```

**57. Nginx inside container can't find upstream:**
```bash
# DNS timing issue — nginx caches DNS at start
# Fix: Use resolver in nginx config
resolver 127.0.0.11 valid=30s;    # Docker's DNS with 30s refresh
set $upstream http://backend:8080;
proxy_pass $upstream;
```

**58. Docker image size after removing file still large:**
```bash
# Removing file in new layer doesn't reduce size!
# ❌ WRONG:
RUN apt-get install -y curl    # Layer 1: +50MB
RUN rm -rf /var/lib/apt/lists/ # Layer 2: removes metadata but layer 1 still large!

# ✅ CORRECT: Same layer
RUN apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*    # Cleanup in SAME layer
```

**59. Service mesh (sidecars) in Docker Compose:**
```yaml
# Add Envoy sidecar alongside app container (shared network)
services:
  app:
    image: myapp
    network_mode: "service:envoy"   # Share network with envoy

  envoy:
    image: envoyproxy/envoy
    ports:
      - "8080:8080"
    volumes:
      - ./envoy.yaml:/etc/envoy/envoy.yaml
```

**60. Debugging intermittent container failures:**
```bash
# Enable automatic restart + logging
docker run \
  --restart=on-failure:10 \
  --log-driver=json-file \
  --log-opt max-size=50m \
  myapp

# Watch for restart events
docker events --filter container=myapp --filter event=restart

# Capture all logs including crashes
docker logs myapp 2>&1 | tee app.log

# Monitor health over time
watch -n 5 docker stats --no-stream myapp
```

---

## 📊 Quick Reference Tables

### Exit Codes

| Code | Meaning | Typical Cause |
|------|---------|---------------|
| `0` | Success / normal exit | App completed its work |
| `1` | General error | App crash, missing config |
| `127` | Command not found | Wrong CMD/ENTRYPOINT |
| `130` | SIGINT (Ctrl+C) | Manual interruption |
| `137` | SIGKILL / OOMKilled | Memory limit exceeded |
| `139` | Segmentation fault | Application bug |
| `143` | SIGTERM | Graceful shutdown signal |

### Docker vs Docker Compose vs Swarm

| Feature | Docker CLI | Docker Compose | Docker Swarm |
|---------|-----------|----------------|-------------|
| Single container | ✅ | ✅ | ✅ |
| Multi-container | Manual | ✅ (one YAML) | ✅ (stack) |
| Multi-host | ❌ | ❌ | ✅ |
| Auto-scaling | ❌ | Manual | ✅ |
| Rolling updates | ❌ | ❌ | ✅ |
| Secrets | ❌ | Limited | ✅ Encrypted |
| Load balancing | ❌ | ❌ | ✅ Built-in |

### Dockerfile Best Practices Summary

| Rule | Reason |
|------|--------|
| Use specific image tags | Reproducibility |
| Copy deps files before source | Better cache |
| Combine RUN commands | Fewer layers |
| Clean up in same RUN | Smaller image |
| Use non-root USER | Security |
| Add HEALTHCHECK | Container health monitoring |
| Use multi-stage builds | Smaller production image |
| Add .dockerignore | Faster builds |
| Use exec form CMD | Signals work correctly |
| Minimize COPY | Build context efficiency |

---
