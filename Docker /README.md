# 🐳 Docker — Complete Guide

> A complete, modular reference for Docker — from fundamentals to production best practices.

---

## 📋 Table of Contents

| # | Module | Topics |
|---|--------|--------|
| 1 | [What is Docker?](#module-1--what-is-docker) | Problem it solves, Analogy, Benefits |
| 2 | [VM vs Container vs Bare Metal](#module-2--vm-vs-container-vs-bare-metal) | Comparison, Architecture diagram |
| 3 | [Docker Architecture](#module-3--docker-architecture) | Client, Daemon, containerd, runc |
| 4 | [Core Concepts](#module-4--core-concepts) | Image, Container, Registry, Layers |
| 5 | [Dockerfile](#module-5--dockerfile) | All instructions, multi-stage, optimization |
| 6 | [Docker Flags](#module-6--docker-flags) | build, run, ps, logs, prune flags |
| 7 | [Images — Commands](#module-7--images--commands) | Build, tag, push, pull, inspect |
| 8 | [Containers — Commands](#module-8--containers--commands) | Run, manage, lifecycle, exec |
| 9 | [Networking](#module-9--networking) | Bridge, host, overlay, none, DNS |
| 10 | [Volumes & Storage](#module-10--volumes--storage) | Named, bind mount, tmpfs, backup |
| 11 | [Docker Compose](#module-11--docker-compose) | YAML reference, commands, env vars |
| 12 | [Docker Swarm](#module-12--docker-swarm) | Cluster, services, stacks, secrets |
| 13 | [Security](#module-13--security) | Rootless, scanning, hardening |
| 14 | [Registry](#module-14--registry) | Docker Hub, ECR, private registry |
| 15 | [Monitoring & Logging](#module-15--monitoring--logging) | Stats, logs, drivers, Prometheus |
| 16 | [Troubleshooting](#module-16--troubleshooting) | Common errors and fixes |
| 17 | [Best Practices](#module-17--best-practices) | Production checklist |
| 18 | [Quick Reference Cheatsheet](#module-18--quick-reference-cheatsheet) | All commands in one place |

---

## Module 1 — What is Docker?

### The Shipping Container Analogy

Before standardized shipping containers, transporting goods was chaotic — every package was a different size and shape, making loading and transport unpredictable.

The shipping container solved this by creating a **standardized box**. Any container could be loaded onto any ship, train, or truck — making transport predictable, universal, and efficient.

**Docker does exactly this for software.**

It puts your application and everything it needs to run — code, libraries, settings, runtime — into a standardized, lightweight unit called a **container**.

### The Problem Docker Solves

The classic developer nightmare: **"But it works on my machine!"**

```
Developer machine:   Python 3.10 → App works ✅
Production server:   Python 3.7  → App crashes ❌
Teammate's machine:  Different OS → App breaks ❌
```

Docker packages the app **with its entire environment**. The container carries Python 3.10, your libraries, your config — all of it. Wherever Docker runs, your app runs exactly the same.

### What is Docker?

Docker is an open-source platform that uses **OS-level virtualization** to deliver software in containers. It allows you to:

- **Develop** — build your app inside a container
- **Ship** — push that container image to a registry
- **Run** — pull and run the same image anywhere

### Key Benefits

| Benefit | Description |
|---------|-------------|
| **Portability** | Build once, run anywhere — dev, staging, prod |
| **Isolation** | Each container has its own filesystem, network, processes |
| **Speed** | Starts in milliseconds vs minutes for VMs |
| **Efficiency** | Shares host OS kernel — much lighter than VMs |
| **Reproducibility** | Same image = same environment every time |
| **Scalability** | Easy horizontal scaling with Swarm or Kubernetes |

---

## Module 2 — VM vs Container vs Bare Metal

### Architecture Comparison

```
┌────────────────┬───────────────────────┬──────────────────────────┐
│  BARE METAL    │  VIRTUAL MACHINE (VM) │  DOCKER (Container)      │
│                │                       │                          │
│  App A         │  App A    │ App B     │  App A  │  App B         │
│  App B         │  Runtime  │ Runtime   │  Runtime│  Runtime       │
│  App C         │  Guest OS │ Guest OS  │  ─────────────────────   │
│  ─────────     │  ─────────────────    │  Docker Engine           │
│  Host OS       │  Hypervisor           │  Host OS                 │
│  ─────────     │  ─────────────────    │  ─────────────────────   │
│  Hardware      │  Host OS + Hardware   │  Hardware                │
└────────────────┴───────────────────────┴──────────────────────────┘
```

### Feature Comparison

| Feature | Bare Metal | Virtual Machine | Docker Container |
|---------|-----------|-----------------|-----------------|
| Startup time | Minutes | Minutes | **Milliseconds** |
| Size | Full OS | GBs | **MBs** |
| OS | Single | Multiple (full) | Shared kernel |
| Isolation | None | Strong (hardware) | Good (process-level) |
| Performance | Best | Good | **Near-native** |
| Portability | Low | Medium | **High** |
| Use case | Raw performance | Full OS isolation | Apps, microservices |

> **Key insight:** VMs virtualize **hardware**. Containers virtualize the **OS**. That's why containers are faster and lighter — they skip the full OS boot and share the host kernel.

---

## Module 3 — Docker Architecture

### How Docker Works Internally

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
│                                ┌─────────▼──────────┐   │
│                                │    Docker Objects   │   │
│                                │  Images │ Containers│   │
│                                │  Volumes│ Networks  │   │
│                                └─────────────────────┘   │
│                                          │               │
│                                ┌─────────▼──────────┐   │
│                                │   Docker Registry   │   │
│                                │  Docker Hub / ECR   │   │
│                                └─────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Components

| Component | What it does |
|-----------|-------------|
| **Docker Client (CLI)** | You type `docker` commands here. Sends instructions to the daemon via REST API |
| **Docker Daemon (`dockerd`)** | Background service that manages images, containers, networks, volumes |
| **containerd** | Container runtime — manages container lifecycle (create, start, stop) |
| **runc** | Low-level runtime — creates containers using Linux namespaces + cgroups |
| **Docker Registry** | Storage for images (Docker Hub, ECR, GHCR, private) |

### What Happens Internally When You Run `docker run nginx`

```
docker run nginx
      │
      ▼
Docker CLI → REST API → Docker Daemon
                              │
                       Image exists locally?
                       ├── YES → use cached image
                       └── NO  → pull from Docker Hub
                              │
                       Create container (containerd)
                              │
                       Create process (runc)
                              │
                       Apply Linux Namespaces:
                       ├── PID   → isolated processes
                       ├── NET   → isolated network
                       ├── MNT   → isolated filesystem
                       ├── UTS   → isolated hostname
                       └── IPC   → isolated inter-process communication
                              │
                       Apply cgroups:
                       ├── CPU limits
                       ├── Memory limits
                       └── I/O limits
                              │
                       Container running ✅
```

---

## Module 4 — Core Concepts

### Image

A Docker **image** is a **read-only template** — like a blueprint or class in programming. It defines:

- The OS base layer (e.g., ubuntu, alpine)
- Installed software and libraries
- Environment variables
- Files copied into it
- The startup command

```
Image  = Blueprint  (static, stored in registry)
Container = Running instance of an image (dynamic, alive)
```

Images are built in **layers**. Each instruction in a Dockerfile creates a new layer on top of the previous one:

```
  ┌──────────────────────────────┐
  │ Layer 4: COPY app code       │  ← your code
  ├──────────────────────────────┤
  │ Layer 3: RUN npm install     │  ← dependencies
  ├──────────────────────────────┤
  │ Layer 2: COPY package.json   │  ← package file
  ├──────────────────────────────┤
  │ Layer 1: RUN apt-get update  │  ← system packages
  ├──────────────────────────────┤
  │ Layer 0: FROM ubuntu:22.04   │  ← base image
  └──────────────────────────────┘
```

When you run a container, Docker adds a **thin writable layer** on top. All read layers are shared between containers — saving disk space.

### Container

A **container** is a running instance of an image. It is:

- **Isolated** — has its own filesystem, network, and processes
- **Lightweight** — shares the host OS kernel (unlike VMs)
- **Ephemeral** — data is lost when deleted (unless volumes are used)

You can run 10 containers from the same image, each doing different things. Stop a container — it pauses. Delete it — the image still exists, ready to make a fresh one.

### Registry

A **registry** stores and distributes images. Docker Hub is the default. When you run `docker pull nginx`, Docker fetches the `nginx` image from Docker Hub.

| Registry | Type | Use Case |
|----------|------|---------|
| Docker Hub | Public/Private cloud | Default, open source |
| AWS ECR | Private cloud | AWS workloads |
| GitHub GHCR | Public/Private | GitHub CI/CD |
| Harbor | Self-hosted | Enterprise, air-gapped |

### Container Lifecycle

```
Created → Running → Paused → Running → Stopped → Deleted
   │                                       │
   └─────────── Restarted ─────────────────┘
```

---

## Module 5 — Dockerfile

### All Dockerfile Instructions

```dockerfile
# ============================================================
#  COMPLETE DOCKERFILE REFERENCE
# ============================================================

# FROM — base image (ALWAYS the first instruction)
FROM ubuntu:22.04

# ENV — environment variables (available at build + runtime)
ENV APP_HOME=/app \
    PORT=8080 \
    NODE_ENV=production

# ARG — build-time arguments (only at build, NOT at runtime)
ARG BUILD_VERSION=1.0.0
ARG GITHUB_TOKEN

# WORKDIR — set working directory (creates it if it doesn't exist)
WORKDIR $APP_HOME

# COPY — copy files from build context into the image
COPY package.json package-lock.json ./
COPY src/ ./src/
COPY --chown=node:node . .     # copy with file ownership

# ADD — like COPY but also extracts tarballs and supports URLs
ADD https://example.com/file.tar.gz /tmp/

# RUN — run a shell command DURING the build (creates a new layer)
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl ca-certificates && \
    rm -rf /var/lib/apt/lists/*    # clean up in the SAME layer!

# Create a non-root user (security best practice)
RUN groupadd -r appgroup && useradd -r -g appgroup -m appuser

# USER — switch to non-root user
USER appuser

# EXPOSE — document which port the app uses (does NOT publish it)
EXPOSE 8080

# VOLUME — create a mount point for persistent data
VOLUME ["/app/data", "/app/logs"]

# HEALTHCHECK — tells Docker how to test if the container is healthy
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# CMD — default command when the container starts (can be overridden)
CMD ["node", "server.js"]

# ENTRYPOINT — main process (harder to override than CMD)
ENTRYPOINT ["node"]
CMD ["server.js"]    # these become arguments to ENTRYPOINT

# LABEL — add metadata to the image
LABEL maintainer="sahil@company.com" \
      version="1.0.0"
```

### Key Instruction Differences

| Instruction | Runs When | Can Override? | Use For |
|-------------|-----------|---------------|---------|
| `RUN` | Build time | N/A | Install packages, compile code |
| `CMD` | Container start | Yes (`docker run image cmd`) | Default startup command |
| `ENTRYPOINT` | Container start | Only with `--entrypoint` | Main executable process |
| `ENV` | Build + Runtime | Yes (`-e` flag) | App configuration |
| `ARG` | Build time only | Yes (`--build-arg`) | Build secrets, versions |
| `COPY` | Build time | N/A | Add local files |
| `ADD` | Build time | N/A | Files + URLs + archives |

### Shell Form vs Exec Form

```dockerfile
# Shell form — runs via /bin/sh -c (spawns a shell process)
CMD node server.js
RUN apt-get update

# Exec form — no shell, direct process (PREFERRED for CMD/ENTRYPOINT)
CMD ["node", "server.js"]
RUN ["apt-get", "update"]
```

> Use **exec form** for `CMD` and `ENTRYPOINT` — signals are passed directly to the process, not a shell.

---

### Multi-Stage Build (Production Must-Have)

**Why?** Your build tools (compiler, npm, SDK) bloat the final image. Multi-stage builds let you build in a heavy image and ship in a tiny one.

```dockerfile
# ── Node.js React App ──────────────────────────────────────────
# Stage 1: Install all deps and build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci                         # clean install
COPY . .
RUN npm run build                  # produces /app/build or /app/dist

# Stage 2: Serve with nginx (tiny final image ~25 MB)
FROM nginx:alpine AS production
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```dockerfile
# ── Java Spring Boot ───────────────────────────────────────────
# Stage 1: Build the JAR
FROM maven:3.9.6-eclipse-temurin-21 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline      # cache dependencies layer
COPY src ./src
RUN mvn clean package -DskipTests  # produces target/*.jar

# Stage 2: Run the JAR (small JRE only)
FROM eclipse-temurin:21-jre-alpine AS runtime
WORKDIR /app
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
COPY --from=builder --chown=appuser:appgroup /app/target/*.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

The final image **only** has the built output + runtime — no source code, no dev tools, no Node.js build environment.

---

### Layer Caching — The Most Important Optimization

Docker caches each layer. If nothing changed in that layer, it reuses the cache. Wrong ordering = slow builds.

```dockerfile
# ❌ BAD — every code change re-runs npm install
COPY . .
RUN npm install

# ✅ GOOD — npm install is cached until package.json changes
COPY package*.json ./    # layer cached until package.json changes
RUN npm install
COPY . .                 # code changes only bust THIS layer
```

```dockerfile
# ❌ BAD — 4 separate layers, last cleanup is in a NEW layer (doesn't help!)
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN rm -rf /var/lib/apt/lists/*

# ✅ GOOD — single layer, cleanup is in the SAME layer
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl wget && \
    rm -rf /var/lib/apt/lists/*
```

---

### .dockerignore (Always Create This)

Like `.gitignore` but for Docker. Prevents unnecessary files from being sent to the build daemon — makes builds faster and images smaller.

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
dist/
build/
*.test.js
__tests__/
```

---

## Module 6 — Docker Flags

### `docker build` Flags

| Flag | Full Name | Purpose | Example |
|------|-----------|---------|---------|
| `-t` | `--tag` | Name and tag the image | `docker build -t myapp:v1 .` |
| `-f` | `--file` | Specify a custom Dockerfile | `docker build -f Dockerfile.prod -t myapp .` |
| `--no-cache` | `--no-cache` | Build without using cache | `docker build --no-cache -t myapp .` |
| `--build-arg` | `--build-arg` | Pass build-time variable | `docker build --build-arg ENV=prod .` |
| `--target` | `--target` | Stop at a specific multi-stage stage | `docker build --target builder .` |

### `docker run` Flags

| Flag | Full Name | Purpose | Example |
|------|-----------|---------|---------|
| `-d` | `--detach` | Run in background | `docker run -d nginx` |
| `-it` | `--interactive --tty` | Interactive terminal (for shells) | `docker run -it ubuntu bash` |
| `-p` | `--publish` | Map host:container port | `docker run -p 8080:80 nginx` |
| `--name` | `--name` | Give container a name | `docker run --name web nginx` |
| `-v` | `--volume` | Mount volume or directory | `docker run -v data:/app/data nginx` |
| `-e` | `--env` | Set environment variable | `docker run -e PORT=8080 myapp` |
| `--env-file` | `--env-file` | Load env vars from file | `docker run --env-file .env myapp` |
| `--rm` | `--rm` | Auto-remove when stopped | `docker run --rm alpine echo hi` |
| `--restart` | `--restart` | Restart policy | `docker run --restart=always nginx` |
| `--memory` | `--memory` | Memory limit | `docker run --memory=512m myapp` |
| `--cpus` | `--cpus` | CPU limit | `docker run --cpus=1.5 myapp` |
| `--network` | `--network` | Connect to network | `docker run --network mynet myapp` |
| `--user` | `--user` | Run as specific user | `docker run --user 1000 myapp` |
| `--read-only` | `--read-only` | Read-only filesystem | `docker run --read-only myapp` |

### Restart Policies

| Policy | When Container Restarts |
|--------|------------------------|
| `no` | Never (default) |
| `always` | Always — even on manual stop |
| `on-failure` | Only on non-zero exit code |
| `on-failure:3` | On failure, max 3 times |
| `unless-stopped` | Always, unless manually stopped |

### `docker ps` Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-a` | Show all containers (including stopped) | `docker ps -a` |
| `-q` | Show only container IDs | `docker ps -aq` |

### `docker logs` Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-f` | Follow logs in real time | `docker logs -f myapp` |
| `--tail` | Show last N lines | `docker logs --tail 50 myapp` |
| `--since` | Show logs since time | `docker logs --since 1h myapp` |

### `docker system prune` Flags

| Flag | Purpose | Warning |
|------|---------|---------|
| `-a` | Remove all unused images too | Removes images not used by any container |
| `--volumes` | Also remove unused volumes | ⚠️ Can delete database data! |

---

## Module 7 — Images — Commands

```bash
# ── BUILD ──────────────────────────────────────────────────────
docker build -t myapp:v1 .
docker build -t myapp:v1 -f Dockerfile.prod .    # custom Dockerfile
docker build --no-cache -t myapp:latest .         # skip cache
docker build --build-arg ENV=prod -t myapp .      # pass build arg
docker build --target builder -t myapp:builder .  # specific stage only

# ── TAG ────────────────────────────────────────────────────────
docker tag myapp:v1 sahil0117/myapp:v1            # tag for Docker Hub
docker tag myapp:v1 myapp:latest                  # also tag as latest

# ── PUSH / PULL ────────────────────────────────────────────────
docker login                                       # login to Docker Hub
docker push sahil0117/myapp:v1                    # push to Docker Hub
docker pull nginx:1.25-alpine                     # pull specific version
docker pull --all-tags myapp                      # pull all tags

# ── LIST / INSPECT ─────────────────────────────────────────────
docker images                                      # list all images
docker images -a                                   # include intermediate
docker image inspect myapp:v1                     # full JSON metadata
docker image history myapp:v1                     # show all layers
docker image inspect --format='{{.Config.Cmd}}' myapp:v1  # specific field

# ── SAVE / LOAD (offline transfer) ────────────────────────────
docker save myapp:v1 | gzip > myapp.tar.gz        # export to file
docker load < myapp.tar.gz                         # import from file

# ── CLEANUP ────────────────────────────────────────────────────
docker rmi myapp:v1                               # remove one image
docker rmi $(docker images -q)                    # remove all images
docker image prune                                # remove dangling images
docker image prune -a                             # remove all unused images
docker system prune -a --volumes                  # full cleanup ⚠️
```

---

## Module 8 — Containers — Commands

```bash
# ── CREATE / RUN ───────────────────────────────────────────────
docker create --name mycontainer ubuntu            # create but don't start
docker run nginx                                   # foreground (blocks terminal)
docker run -d nginx                                # detached (background)
docker run -it ubuntu bash                         # interactive shell
docker run --name web -d -p 8080:80 nginx         # named + port + detached
docker run -e DB_HOST=localhost nginx              # with env variable
docker run --env-file .env nginx                   # env from file
docker run -v myvolume:/data nginx                 # named volume
docker run -v /host/path:/app nginx                # bind mount
docker run --rm alpine echo "hello"               # auto-remove on exit
docker run --restart=always nginx                  # always restart
docker run --memory=512m --cpus=1.0 nginx         # resource limits
docker run --network mynet nginx                   # custom network

# ── MANAGE ─────────────────────────────────────────────────────
docker ps                                          # running containers
docker ps -a                                       # all containers
docker start mycontainer                           # start stopped container
docker stop mycontainer                            # graceful stop (SIGTERM)
docker stop -t 30 mycontainer                     # 30s grace period
docker kill mycontainer                            # force stop (SIGKILL)
docker restart mycontainer
docker pause mycontainer                           # freeze processes
docker unpause mycontainer
docker rm mycontainer                              # remove stopped container
docker rm -f mycontainer                           # force remove running
docker rm $(docker ps -aq)                        # remove all containers

# ── INSPECT / DEBUG ────────────────────────────────────────────
docker logs mycontainer                            # view logs
docker logs -f mycontainer                         # follow live logs
docker logs --tail=50 mycontainer                 # last 50 lines
docker logs --since 1h mycontainer                # last 1 hour of logs
docker exec -it mycontainer bash                  # open shell inside
docker exec mycontainer env                        # run one-off command
docker inspect mycontainer                         # full metadata JSON
docker stats mycontainer                           # live resource usage
docker stats --no-stream                           # one-time snapshot
docker top mycontainer                             # processes inside
docker port mycontainer                            # port mappings
docker diff mycontainer                            # changed files

# ── COPY FILES ─────────────────────────────────────────────────
docker cp mycontainer:/path/to/file ./             # copy FROM container
docker cp ./file mycontainer:/path/to/             # copy TO container
```

---

## Module 9 — Networking

### Network Types

```
┌──────────────────────────────────────────────────────────────┐
│                     DOCKER NETWORKING                        │
│                                                              │
│  BRIDGE (default)           CUSTOM BRIDGE                    │
│  ┌──────────────────┐       ┌────────────────────────┐       │
│  │ Container A      │       │ Container A            │       │
│  │ 172.17.0.2       │       │ reachable by name ✅   │       │
│  │    │             │       │       │                │       │
│  │ docker0          │       │  my-network            │       │
│  │ 172.17.0.1       │       │       │                │       │
│  │    │             │       │ Container B            │       │
│  │  HOST            │       │ reachable by name ✅   │       │
│  └──────────────────┘       └────────────────────────┘       │
│                                                              │
│  HOST NETWORK                NONE NETWORK                    │
│  ┌──────────────────┐       ┌────────────────────────┐       │
│  │ No isolation     │       │ No network at all      │       │
│  │ Uses host ports  │       │ Fully isolated          │       │
│  │ directly         │       │                        │       │
│  └──────────────────┘       └────────────────────────┘       │
└──────────────────────────────────────────────────────────────┘
```

| Type | Description | Use Case |
|------|-------------|---------|
| `bridge` | Default — isolated, NAT to host | Single-host apps |
| `host` | Container uses host network directly | High performance |
| `none` | No network interface | Fully isolated jobs |
| `overlay` | Multi-host networking (Swarm/K8s) | Distributed apps |
| `macvlan` | Container gets its own MAC/IP | Legacy app integration |

### Important: Default Bridge vs Custom Bridge

| Feature | Default `bridge` | Custom bridge |
|---------|-----------------|---------------|
| DNS by name | ❌ Must use IP | ✅ Use container name |
| Isolation | Shared with all | Isolated per network |
| Best practice | Dev/test only | Always use in production |

### Network Commands

```bash
# ── CREATE / MANAGE ────────────────────────────────────────────
docker network create mynet                        # custom bridge network
docker network create --driver bridge mybridge
docker network create --subnet 192.168.1.0/24 mynet  # custom subnet

# ── CONNECT / DISCONNECT ───────────────────────────────────────
docker run -d --network mynet --name app nginx     # connect at run time
docker network connect mynet existing-container   # connect existing
docker network disconnect mynet container          # disconnect

# ── INSPECT ────────────────────────────────────────────────────
docker network ls                                  # list all networks
docker network inspect mynet                       # full details

# ── DNS (Custom Bridge only) ───────────────────────────────────
# Container A and B on same custom network:
# Inside B: curl http://app   ← resolved by container name ✅
# Default bridge: must use IP ❌

# ── CLEANUP ────────────────────────────────────────────────────
docker network rm mynet
docker network prune                               # remove unused networks
```

---

## Module 10 — Volumes & Storage

### Storage Types Compared

```
┌──────────────────────────────────────────────────────────────┐
│                     DOCKER STORAGE                           │
│                                                              │
│  VOLUMES (Best for persistent data)                          │
│  ├── Managed by Docker                                       │
│  ├── Stored at /var/lib/docker/volumes/                      │
│  ├── Works on all platforms                                  │
│  └── Can be shared between containers                        │
│                                                              │
│  BIND MOUNTS (Best for development)                          │
│  ├── Points to an exact host path                            │
│  ├── Changes on host reflect instantly in container          │
│  └── Less portable between machines                          │
│                                                              │
│  TMPFS MOUNTS (Temporary in-memory storage)                  │
│  ├── Data lives in RAM only                                  │
│  ├── Lost when container stops                               │
│  └── Good for sensitive data (tokens, temp files)            │
└──────────────────────────────────────────────────────────────┘
```

| Feature | Volumes | Bind Mounts |
|---------|---------|-------------|
| Managed by | Docker | OS (user) |
| Location | `/var/lib/docker/volumes/` | Any host path |
| Portability | High | Low |
| Use case | Production data | Dev hot-reload |

### Volume Commands

```bash
# ── CREATE / USE ───────────────────────────────────────────────
docker volume create myvolume                      # create a named volume
docker volume ls                                   # list all volumes
docker volume inspect myvolume                    # details
docker volume rm myvolume                          # remove volume
docker volume prune                                # remove unused volumes

# Use in docker run:
docker run -d -v myvolume:/app/data nginx          # named volume
docker run -d -v /host/path:/app nginx             # bind mount
docker run -d -v /host/path:/app:ro nginx          # read-only bind mount
docker run -d --tmpfs /tmp nginx                   # tmpfs mount

# ── BACKUP A VOLUME ────────────────────────────────────────────
# Backup to tar.gz
docker run --rm \
  -v myvolume:/data \
  -v $(pwd):/backup \
  ubuntu tar czf /backup/volume-backup.tar.gz /data

# Restore from tar.gz
docker run --rm \
  -v myvolume:/data \
  -v $(pwd):/backup \
  ubuntu tar xzf /backup/volume-backup.tar.gz -C /
```

---

## Module 11 — Docker Compose

### What is Docker Compose?

Docker Compose runs **multi-container applications** from a single YAML file. Instead of running 4 `docker run` commands for frontend, backend, database, and cache — one `docker compose up` starts everything.

### Complete `docker-compose.yml` Reference

```yaml
version: "3.9"

services:

  # ── FRONTEND ────────────────────────────────────────────────
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      target: production              # use multi-stage target
      args:
        NODE_ENV: production
    image: myapp-frontend:latest
    container_name: myapp-frontend
    restart: unless-stopped
    ports:
      - "80:80"
    networks:
      - frontend-net
    depends_on:
      backend:
        condition: service_healthy    # wait for backend health check
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

  # ── BACKEND ─────────────────────────────────────────────────
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
        condition: service_healthy    # wait for DB health check
    environment:
      - DB_HOST=database
      - DB_USER=${DB_USER}           # from .env file
      - DB_PASSWORD=${DB_PASSWORD}
    env_file:
      - .env                         # load from .env automatically
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8081/health"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 60s
    volumes:
      - logs-volume:/app/logs

  # ── DATABASE ────────────────────────────────────────────────
  database:
    image: postgres:15-alpine
    container_name: myapp-db
    restart: unless-stopped
    networks:
      - backend-net
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: mydb
    volumes:
      - db-data:/var/lib/postgresql/data   # persist data!
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ── REDIS ───────────────────────────────────────────────────
  redis:
    image: redis:7-alpine
    container_name: myapp-redis
    restart: unless-stopped
    networks:
      - backend-net
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes

# ── NETWORKS ────────────────────────────────────────────────────
networks:
  frontend-net:
    driver: bridge
  backend-net:
    driver: bridge
    internal: true     # no external access to DB network!

# ── VOLUMES ─────────────────────────────────────────────────────
volumes:
  db-data:
  redis-data:
  logs-volume:
```

### Docker Compose Commands

```bash
# ── LIFECYCLE ──────────────────────────────────────────────────
docker compose up                         # start all services (foreground)
docker compose up -d                      # start in background
docker compose up --build                 # rebuild images then start
docker compose up --build backend         # rebuild one service only
docker compose down                       # stop + remove containers/networks
docker compose down -v                    # also remove volumes ⚠️
docker compose start                      # start stopped services
docker compose stop                       # stop without removing
docker compose restart backend            # restart one service

# ── INSPECT ────────────────────────────────────────────────────
docker compose ps                         # status of all services
docker compose logs                       # logs from all services
docker compose logs -f backend            # follow backend logs
docker compose logs --tail=50 backend
docker compose top                        # processes per service

# ── EXEC / RUN ─────────────────────────────────────────────────
docker compose exec backend bash          # shell into running service
docker compose run --rm backend pytest    # one-off command in new container

# ── BUILD / PULL ───────────────────────────────────────────────
docker compose build                      # build all services
docker compose build --no-cache backend   # rebuild one without cache
docker compose pull                       # pull latest images

# ── SCALE ──────────────────────────────────────────────────────
docker compose up --scale backend=3       # run 3 backend instances

# ── CONFIG ─────────────────────────────────────────────────────
docker compose config                     # validate + show merged config
docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```

### `.env` File (Auto-loaded by Compose)

```env
# .env
DB_USER=admin
DB_PASSWORD=secretpass
REDIS_PASSWORD=redispass
```

---

## Module 12 — Docker Swarm

### What is Docker Swarm?

Docker Swarm turns multiple Docker hosts into a single cluster. You declare services and Swarm schedules containers across all nodes automatically.

### Swarm Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    DOCKER SWARM CLUSTER                      │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │         MANAGER NODES (Raft Consensus)                 │  │
│  │  Manager 1 (Leader) — schedules + maintains state      │  │
│  │  Manager 2 (Follower)                                  │  │
│  │  Manager 3 (Follower)                                  │  │
│  └───────────────────────┬────────────────────────────────┘  │
│                          │ assign tasks                      │
│             ┌────────────┼────────────┐                      │
│             ▼            ▼            ▼                      │
│       Worker Node 1  Worker Node 2  Worker Node 3            │
│       (runs containers/tasks)                                │
└──────────────────────────────────────────────────────────────┘
```

### Swarm Commands

```bash
# ── INIT / JOIN ────────────────────────────────────────────────
docker swarm init --advertise-addr 192.168.1.10
docker swarm join --token <token> 192.168.1.10:2377
docker swarm join-token worker                     # get worker join token
docker swarm join-token manager                    # get manager join token
docker swarm leave --force                         # leave the swarm

# ── NODE MANAGEMENT ────────────────────────────────────────────
docker node ls                                     # list all nodes
docker node inspect <node>
docker node update --availability=drain <node>     # remove from rotation
docker node promote <node>                         # worker → manager
docker node demote <node>                          # manager → worker
docker node rm <node>                              # remove node from swarm

# ── SERVICES ───────────────────────────────────────────────────
docker service create --name web --replicas 3 -p 80:80 nginx
docker service ls                                  # list services
docker service ps web                              # tasks per service
docker service logs web -f                         # follow service logs
docker service scale web=5                         # scale up/down
docker service update --image nginx:1.25 web      # rolling update
docker service update --rollback web               # rollback
docker service rm web                              # delete service

# ── STACKS (Compose files in Swarm) ───────────────────────────
docker stack deploy -c docker-compose.yml myapp   # deploy stack
docker stack ls                                    # list stacks
docker stack services myapp                        # services in stack
docker stack ps myapp                              # tasks in stack
docker stack rm myapp                              # remove stack

# ── SECRETS ────────────────────────────────────────────────────
echo "mypassword" | docker secret create db_password -
docker secret ls
docker secret rm db_password
```

### Swarm vs Kubernetes

| Feature | Docker Swarm | Kubernetes |
|---------|-------------|-----------|
| Setup | Easy (5 min) | Complex |
| Learning curve | Low | High |
| Scaling | Good | Excellent |
| Auto-healing | Yes | Yes (more advanced) |
| Load balancing | Built-in | Ingress controller |
| Best for | Small/medium apps | Enterprise/large scale |

---

## Module 13 — Security

### Security Layers

```
1. Build-time Security
   ├── Use official base images
   ├── Pin image versions (nginx:1.25.3 not nginx:latest)
   ├── Scan images with Trivy or Snyk
   ├── Use non-root USER
   └── Minimal base (alpine, distroless)

2. Runtime Security
   ├── Read-only filesystem (--read-only)
   ├── Resource limits (--memory, --cpus)
   ├── Drop capabilities (--cap-drop ALL)
   ├── No privileged mode
   └── Security profiles (seccomp, AppArmor)

3. Network Security
   ├── Custom networks (not default bridge)
   ├── Expose only required ports
   └── Use encrypted overlay networks in Swarm

4. Secrets Management
   ├── Docker Swarm secrets
   ├── HashiCorp Vault
   └── NEVER store secrets in Dockerfiles or git
```

### Secure Dockerfile Pattern

```dockerfile
FROM node:20.11.0-alpine3.19        # pinned version, not :latest

# Install security updates
RUN apk update && apk upgrade && rm -rf /var/cache/apk/*

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production         # production deps only

COPY --chown=node:node . .          # correct file ownership

USER node                            # non-root user

EXPOSE 8080
CMD ["node", "server.js"]
```

### Secure `docker run` Pattern

```bash
docker run \
  --read-only \                       # read-only filesystem
  --tmpfs /tmp:size=100m \            # writable temp in RAM only
  --user 1000:1000 \                  # non-root user
  --cap-drop ALL \                    # drop ALL Linux capabilities
  --cap-add NET_BIND_SERVICE \        # add back only what's needed
  --security-opt=no-new-privileges \  # no privilege escalation
  --memory 512m \                     # memory limit
  --memory-swap 512m \                # disable swap
  --cpus 1.0 \                        # CPU limit
  --pids-limit 100 \                  # process count limit
  --network my-secure-net \           # named custom network
  myapp:v1
```

### Image Scanning with Trivy

```bash
# Scan an image
trivy image myapp:v1

# Show only HIGH and CRITICAL severity
trivy image --severity HIGH,CRITICAL myapp:v1

# Fail the build if CRITICAL vulnerabilities found (use in CI/CD)
trivy image --exit-code 1 --severity CRITICAL myapp:v1

# Scan the Dockerfile itself
trivy config ./

# Scan a local directory
trivy fs .
```

---

## Module 14 — Registry

### Registry Types

| Registry | Type | Best For |
|----------|------|---------|
| Docker Hub | Public/Private cloud | Default, open source |
| AWS ECR | Private cloud | AWS-hosted workloads |
| Google Artifact Registry | Private cloud | GCP workloads |
| GitHub GHCR | Public/Private | GitHub CI/CD |
| Harbor | Self-hosted | Enterprise, air-gapped |

### Registry Commands

```bash
# ── DOCKER HUB ─────────────────────────────────────────────────
docker login                                       # login
docker login -u sahil0117                         # specify username
docker logout

docker push sahil0117/myapp:v1
docker pull sahil0117/myapp:v1

# ── AWS ECR ────────────────────────────────────────────────────
# Authenticate with ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS \
  --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com

# Create a repository
aws ecr create-repository --repository-name myapp --region us-east-1

# Tag and push
docker tag myapp:v1 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:v1
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:v1

# ── PRIVATE REGISTRY ───────────────────────────────────────────
# Run your own registry
docker run -d -p 5000:5000 --name registry \
  -v registry-data:/var/lib/registry \
  registry:2

# Push and pull from private registry
docker tag myapp:v1 localhost:5000/myapp:v1
docker push localhost:5000/myapp:v1
docker pull localhost:5000/myapp:v1
```

---

## Module 15 — Monitoring & Logging

### Container Stats

```bash
docker stats                                       # live stats — all containers
docker stats --no-stream                           # one-time snapshot
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

docker logs myapp                                  # view logs
docker logs -f myapp                               # follow in real time
docker logs --tail 100 myapp                       # last 100 lines
docker logs --since 1h myapp                       # last 1 hour
docker logs --until 2026-05-24T10:00 myapp
docker logs myapp 2>&1 | grep ERROR                # filter for errors

docker system df                                   # disk usage
docker system events                               # real-time events
docker system info                                 # system information
```

### Logging Drivers

```bash
# json-file (default) — stored in /var/lib/docker/containers/
docker run --log-driver=json-file \
  --log-opt max-size=10m \           # rotate at 10 MB
  --log-opt max-file=3 \             # keep 3 rotated files
  nginx

# Send to AWS CloudWatch
docker run \
  --log-driver=awslogs \
  --log-opt awslogs-group=/docker/myapp \
  --log-opt awslogs-region=us-east-1 \
  nginx

# Send to Fluentd
docker run \
  --log-driver=fluentd \
  --log-opt fluentd-address=localhost:24224 \
  nginx
```

### Prometheus + Grafana Monitoring Stack

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

  cadvisor:                          # collects container metrics
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

## Module 16 — Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| **Container exits immediately** | Main process finished / no foreground process | `docker run -it ubuntu bash` or `docker logs myapp` to see why |
| **Port not accessible** | Port not published or firewall block | `docker run -p 8080:80`; check EC2 Security Groups |
| **Image not found** | Wrong image name/tag or not pushed | `docker images`; verify `docker push` succeeded |
| **No internet inside container** | Docker network issue | `sudo systemctl restart docker` |
| **No space left on device** | Old images/containers consuming disk | `docker system prune -a` |
| **Permission denied on volume** | Wrong folder ownership | `sudo chown -R $USER /path` |
| **Cannot connect to Docker daemon** | Service not running or wrong user | `sudo systemctl start docker`; `sudo usermod -aG docker $USER` |
| **Name already in use** | Container name exists | `docker rm -f <name>` then re-run |
| **Port already allocated** | Another container using that port | `docker ps` to find it; use different host port |
| **Device or resource busy** | Container still running | `docker stop <name>` then `docker rm <name>` |
| **COPY/ADD error in Dockerfile** | Wrong path in build context | Check relative paths; verify file exists |
| **Mounts denied (macOS)** | Path not shared with Docker Desktop | Docker Desktop → Settings → Resources → File Sharing |
| **Slow build** | No cache, large layers | Optimize Dockerfile layer order; use alpine base |
| **CrashLoopBackOff (K8s)** | Container crashes on startup | `docker logs <name>` or `kubectl logs <pod>` |
| **Docker Compose service fails** | YAML syntax error or wrong deps | `docker compose config` to validate |

### Troubleshooting Commands

```bash
docker logs mycontainer                            # first thing to check
docker inspect mycontainer                         # full metadata
docker stats                                       # resource usage
docker events                                      # real-time events
docker system df                                   # disk usage
sudo systemctl status docker                       # Docker service status
sudo systemctl restart docker                      # restart Docker
```

---

## Module 17 — Best Practices

### Production Checklist

```
✅ DO:
├── Pin image versions (nginx:1.25.3, NOT nginx:latest)
├── Use multi-stage builds to minimize final image size
├── Add USER <non-root> in every Dockerfile
├── Always create a .dockerignore file
├── Use official or verified base images
├── One main process per container
├── Set resource limits (--memory, --cpus)
├── Add HEALTHCHECK to every service
├── Use named volumes for all persistent data
├── Scan images before deploying (Trivy)
├── Use custom networks (never the default bridge)
├── Tag images with git commit hash or version number
└── Store secrets in Swarm secrets or Vault — never in Dockerfiles

❌ DON'T:
├── Use :latest in production deployments
├── Run containers as root
├── Hardcode passwords or tokens in Dockerfiles or Compose files
├── Use --privileged unless absolutely required
├── Put all commands in one giant RUN layer
├── Skip HEALTHCHECK configuration
├── Commit .env files to git
├── Expose ports you don't need
└── Ignore docker system df until disk is full
```

### Image Size Optimization

```dockerfile
# Use alpine (tiny base)
FROM python:3.11-alpine            # vs python:3.11 (~900 MB smaller)

# Use Google distroless (no shell, minimal attack surface)
FROM gcr.io/distroless/python3:latest

# Always clean package cache in the SAME RUN layer
RUN apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*    # NOT in a separate RUN below!
```

---

## Module 18 — Quick Reference Cheatsheet

```bash
# ── IMAGES ──────────────────────────────────────────────────────
docker build -t name:tag .               # build image
docker build --no-cache -t name:tag .   # build without cache
docker images                            # list images
docker pull nginx:alpine                 # pull image
docker push user/repo:tag                # push to registry
docker tag src:tag user/repo:tag         # tag for registry
docker rmi name:tag                      # remove image
docker image prune -a                    # remove all unused images
docker image history name:tag            # show all layers
docker save name:tag | gzip > file.tar  # export to file
docker load < file.tar.gz                # import from file

# ── CONTAINERS ──────────────────────────────────────────────────
docker run -d -p 8080:80 --name web nginx  # run in background
docker run -it ubuntu bash               # interactive shell
docker run --rm alpine echo hello        # auto-remove on exit
docker ps                                # running containers
docker ps -a                             # all containers
docker stop myapp                        # graceful stop
docker kill myapp                        # force stop
docker rm myapp                          # remove stopped
docker rm -f myapp                       # force remove running
docker rm $(docker ps -aq)              # remove all
docker logs -f myapp                     # follow logs
docker exec -it myapp bash               # shell into container
docker stats myapp                       # resource usage
docker inspect myapp                     # full details
docker cp file myapp:/path               # copy to container
docker cp myapp:/path file               # copy from container

# ── NETWORKS ────────────────────────────────────────────────────
docker network create mynet              # create network
docker network ls                        # list networks
docker network inspect mynet             # details
docker network connect mynet myapp       # connect container
docker network rm mynet                  # remove network
docker network prune                     # remove unused

# ── VOLUMES ─────────────────────────────────────────────────────
docker volume create myvol               # create volume
docker volume ls                         # list volumes
docker volume inspect myvol              # details
docker volume rm myvol                   # remove volume
docker volume prune                      # remove unused

# ── COMPOSE ─────────────────────────────────────────────────────
docker compose up -d                     # start all services
docker compose up --build -d             # rebuild and start
docker compose down                      # stop all
docker compose down -v                   # stop + remove volumes
docker compose ps                        # service status
docker compose logs -f backend           # follow logs
docker compose exec backend bash         # shell into service
docker compose scale backend=3           # scale service
docker compose config                    # validate config

# ── SWARM ───────────────────────────────────────────────────────
docker swarm init                        # start swarm
docker swarm join --token <t> host:2377 # join as worker
docker node ls                           # list nodes
docker service create --replicas 3 ...  # create service
docker service scale svc=5               # scale service
docker service update --image img svc   # rolling update
docker service rollback svc              # rollback
docker stack deploy -c compose.yml app  # deploy stack

# ── SYSTEM ──────────────────────────────────────────────────────
docker system info                       # system info
docker system df                         # disk usage
docker system prune                      # remove unused
docker system prune -a --volumes         # full cleanup ⚠️
docker system events                     # real-time events
docker version                           # Docker version

# ── REGISTRY ────────────────────────────────────────────────────
docker login                             # login to Docker Hub
docker logout                            # logout
docker search nginx                      # search images
```

---

*Reference: Docker official documentation — https://docs.docker.com*
