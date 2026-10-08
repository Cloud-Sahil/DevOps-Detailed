# 🐳 Dockerfile Complete Notes (Modular Edition)

> From first Dockerfile to interview-ready: instructions, build/run workflow, hands-on projects (Flask, Java, MySQL, Apache), CMD vs ENTRYPOINT, build optimization, security best practices, troubleshooting, and interview Q&A.

---

## 📑 Table of Contents

1. [Dockerfile Basics](#-module-1-dockerfile-basics)
2. [Instructions Reference](#-module-2-instructions-reference)
3. [Build & Run Workflow](#-module-3-build--run-workflow)
4. [Hands-On Projects](#-module-4-hands-on-projects)
   - 4.1 Python Flask
   - 4.2 Java Web Server
   - 4.3 MySQL
   - 4.4 Apache
5. [CMD vs ENTRYPOINT](#-module-5-cmd-vs-entrypoint)
6. [Build Optimization](#-module-6-build-optimization)
7. [Best Practices & Security](#-module-7-best-practices--security)
8. [Useful Docker Commands](#-module-8-useful-docker-commands)
9. [Troubleshooting](#-module-9-troubleshooting)
10. [Interview Revision](#-module-10-interview-revision)
11. [Cheat Sheet & Next Steps](#-module-11-cheat-sheet--next-steps)

---

# 🧩 Module 1: Dockerfile Basics

A **Dockerfile** is a text file containing instructions to build a Docker image. It defines how an application and its environment are set up inside a container.

## ⚙️ Key Points

- Each line in a Dockerfile is an **instruction**.
- Docker processes instructions from **top to bottom** while building the image.
- It automates installing dependencies, copying application files, configuring the environment, and defining the container startup command.
- The file is usually named exactly `Dockerfile` (no extension) and placed in the project root.

## 🧠 Image vs Container vs Registry

| Term | Meaning |
|---|---|
| Dockerfile | The recipe (text instructions) |
| Image | A read-only template built from the Dockerfile |
| Container | A running instance of an image |
| Registry | A storage service for images (Docker Hub, ECR, GCR) |

## 📦 Build Context

The `.` at the end of `docker build -t myimage .` is the **build context**: the folder whose files Docker can `COPY` into the image. Files outside the build context cannot be copied.

## 🧱 Layers

- Instructions that change the filesystem (`RUN`, `COPY`, `ADD`) each create an **image layer**.
- Other instructions (`ENV`, `EXPOSE`, `CMD`, ...) only add metadata.
- Docker **caches** layers. If a layer and everything above it is unchanged, Docker reuses the cache instead of rebuilding (see Module 6).

## 🚀 Big Picture

```text
Dockerfile
    ↓  docker build
Docker Image
    ↓  docker run
Docker Container
    ↓
Application
```

---

# 📋 Module 2: Instructions Reference

| Instruction | Description | Example |
|---|---|---|
| `FROM` | Defines the base image. | `FROM ubuntu:24.04` |
| `LABEL` | Adds image metadata such as author or version. | `LABEL maintainer="omkar@example.com"` |
| `RUN` | Executes commands during image build. | `RUN apt-get update && apt-get install -y python3` |
| `WORKDIR` | Sets the working directory for following instructions. | `WORKDIR /app` |
| `COPY` | Copies files/folders from the build context into the image. | `COPY . /app` |
| `ADD` | Similar to `COPY`, with extra archive/URL features. | `ADD app.tar.gz /app` |
| `CMD` | Defines the default command when a container starts. | `CMD ["python3", "app.py"]` |
| `ENTRYPOINT` | Defines the main executable for the container. | `ENTRYPOINT ["java", "-jar", "app.jar"]` |
| `EXPOSE` | Documents the port the application listens on. | `EXPOSE 80` |
| `ENV` | Sets environment variables (build time and run time). | `ENV DB_HOST=localhost` |
| `VOLUME` | Defines a mount point for persistent/external data. | `VOLUME /data` |
| `USER` | Specifies the user used to run commands/processes. | `USER appuser` |
| `ARG` | Defines build-time variables. | `ARG VERSION=1.0` |
| `HEALTHCHECK` | Defines a command to check container health. | `HEALTHCHECK CMD curl -f http://localhost/` |
| `ONBUILD` | Adds a trigger that runs when the image is used as a base image. | `ONBUILD COPY . /app` |

> **Note:** For `HEALTHCHECK`, the command must exit with `0` (healthy) or `1` (unhealthy). That is why real Dockerfiles write `curl -f http://localhost/ || exit 1`. Full example in Module 7.

## 🔀 COPY vs ADD

| Feature | `COPY` | `ADD` |
|---|---|---|
| Copy local files/folders | Yes | Yes |
| Auto-extract local `.tar` / `.tar.gz` | No | **Yes** |
| Download from a URL | No | Yes (not recommended) |
| Recommended for normal use | **Yes** | Only when you need auto-extraction |

**Rule of thumb:** use `COPY` by default. It is explicit and predictable.

> **Trailing slash tip:** write `COPY app.py /app/` (or `COPY app.py .` after `WORKDIR /app`). Without the trailing slash it only works because `WORKDIR` already created the `/app` directory; if it did not exist, Docker would treat `/app` as a *file name*.

## 🔀 ARG vs ENV

| Feature | `ARG` | `ENV` |
|---|---|---|
| Available during build | Yes | Yes |
| Available when container runs | **No** | **Yes** |
| Set at build time with `--build-arg` | Yes | No |
| Override at run time with `-e` | No | Yes |
| Safe for secrets | **No** (visible in `docker history`) | **No** (visible in `docker inspect`) |

```dockerfile
ARG APP_VERSION=1.0
ENV APP_ENV=production
```

```bash
docker build --build-arg APP_VERSION=2.0 -t myapp .
docker run -e APP_ENV=staging myapp
```

## 🔀 Shell Form vs Exec Form

```dockerfile
CMD python app.py              # shell form  -> runs as: /bin/sh -c "python app.py"
CMD ["python", "app.py"]       # exec form   -> runs python directly (recommended)
```

| Feature | Shell form | Exec form (JSON array) |
|---|---|---|
| Runs through a shell | Yes | No |
| Variable expansion such as `$HOME` | Yes | No (no shell) |
| Receives `docker stop` signal (SIGTERM) properly | Often **no** | **Yes** |
| Recommended for `CMD` / `ENTRYPOINT` | No | **Yes** |

## 🔌 EXPOSE vs `-p`

- `EXPOSE 5000` only **documents** the port. It does **not** publish it.
- `-p 5000:5000` at `docker run` time actually **publishes** the port to the host.
- `-P` (capital) publishes all exposed ports to random host ports.

---

# 🔨 Module 3: Build & Run Workflow

## 🧱 Basic Dockerfile Structure

```dockerfile
FROM <base-image>
WORKDIR /app
COPY . /app
RUN <commands>
EXPOSE <port>
CMD ["<command>", "<arg>"]
```

### Flow

1. `FROM` → Select the base image.
2. `WORKDIR` → Set the working directory.
3. `COPY` → Copy application files.
4. `RUN` → Install dependencies or configure the image.
5. `EXPOSE` → Document the application port.
6. `CMD` → Define the default startup command.

## 1️⃣ Create Project Folder

Keep the source code and Dockerfile together.

```text
python-web/
├── Dockerfile
└── app.py
```

## 2️⃣ Write the Dockerfile

```dockerfile
FROM <base-image>
WORKDIR <directory>
COPY <files> <destination>
RUN <install-or-setup-commands>
EXPOSE <port>
CMD ["<command>", "<args>"]
```

## 3️⃣ Build Image

```bash
docker build -t <image-name> .
```

Example:

```bash
docker build -t python-web .
```

## 4️⃣ Run Container

```bash
docker run -d -p <host-port>:<container-port> --name <container-name> <image-name>
```

Example:

```bash
docker run -d -p 5000:5000 --name python-web-container python-web
```

> **Port order:** `-p HOST:CONTAINER`. Example: `-p 8080:5000` means browser port 8080 on your machine goes to port 5000 inside the container.

## 5️⃣ Access the Application

| Where Docker runs | URL to open |
|---|---|
| Your own laptop/PC | `http://localhost:<host-port>` |
| Cloud VM (EC2, Azure VM, ...) | `http://<public-ip>:<host-port>` (open that port in the Security Group / firewall) |
| CLI-only apps (like MySQL) | Use `docker exec` or a client tool |

## 🏷️ Useful `docker build` Flags

| Flag | Purpose |
|---|---|
| `-t name:tag` | Name and tag the image |
| `-f path/to/Dockerfile` | Use a Dockerfile with a different name or location |
| `--no-cache` | Ignore the cache and rebuild every layer |
| `--build-arg KEY=VALUE` | Pass a value for an `ARG` |
| `--progress=plain` | Show full, unfolded build output (great for debugging) |
| `--platform linux/amd64` | Build for a specific CPU architecture |

## 🏷️ Useful `docker run` Flags

| Flag | Purpose |
|---|---|
| `-d` | Run in the background (detached) |
| `-it` | Interactive terminal (for shells) |
| `-p HOST:CONTAINER` | Publish a port |
| `--name` | Give the container a name |
| `-e KEY=VALUE` | Set an environment variable |
| `--env-file file` | Load many environment variables from a file |
| `-v source:destination` | Mount a volume or a host folder |
| `--rm` | Remove the container automatically when it stops |
| `--restart unless-stopped` | Restart automatically after crashes/reboots |
| `--network name` | Attach the container to a specific network |

---

# 🛠️ Module 4: Hands-On Projects

Four ready-to-run setups. The Flask and Java source files below were tested to start and respond correctly.

---

## 4.1 🐍 Python Flask Application

### 📁 Folder Structure

```text
python-web/
├── Dockerfile
└── app.py
```

### 📄 `app.py`

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Flask inside Docker! 🐳"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

> **Important:** `host="0.0.0.0"` is required. If Flask listens only on `127.0.0.1`, it is unreachable from outside the container even with `-p 5000:5000`.

### 🧱 Dockerfile (Basic)

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

RUN pip install --no-cache-dir flask

EXPOSE 5000

CMD ["python", "app.py"]
```

### Explanation

- `FROM python:3.12-slim` → Python 3.12 slim base image (smaller than the full image).
- `WORKDIR /app` → Sets `/app` as the working directory.
- `COPY app.py .` → Copies the app into `/app`.
- `RUN pip install --no-cache-dir flask` → Installs Flask; `--no-cache-dir` keeps the image smaller.
- `EXPOSE 5000` → Documents port 5000.
- `CMD ["python", "app.py"]` → Starts the Flask application.

> **Why not `python:3.9`?** Python 3.9 reached end-of-life in October 2025 and no longer gets security fixes. Use a supported version.

### ⚙️ Workflow

```bash
docker build -t python-web .
docker run -d -p 5000:5000 --name python-web-container python-web
curl http://localhost:5000
```

### 🧱 Dockerfile (Improved: caching + non-root + health check)

`requirements.txt`:

```text
flask==3.1.3
```

```dockerfile
FROM python:3.12-slim

WORKDIR /app

# 1. Dependencies first (changes rarely -> cached)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 2. Application code last (changes often)
COPY app.py .

# 3. Run as a non-root user
RUN useradd --create-home appuser
USER appuser

EXPOSE 5000

# 4. Health check (python:slim has no curl, so use Python itself)
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:5000/')" || exit 1

CMD ["python", "app.py"]
```

> Flask's built-in server is for learning/development. For production, run the app with a real WSGI server such as `gunicorn`.

---

## 4.2 ☕ Java Web Server

### 📁 Folder Structure

```text
java-web/
├── Dockerfile
└── SimpleServer.java
```

### 📄 `SimpleServer.java`

```java
import com.sun.net.httpserver.HttpServer;
import java.io.IOException;
import java.io.OutputStream;
import java.net.InetSocketAddress;
import java.nio.charset.StandardCharsets;

public class SimpleServer {
    public static void main(String[] args) throws IOException {
        HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);
        server.createContext("/", exchange -> {
            byte[] body = "Hello from Java inside Docker!".getBytes(StandardCharsets.UTF_8);
            exchange.sendResponseHeaders(200, body.length);
            try (OutputStream os = exchange.getResponseBody()) {
                os.write(body);
            }
        });
        server.start();
        System.out.println("Server started on port 8080");
    }
}
```

### 🧱 Dockerfile

```dockerfile
FROM eclipse-temurin:17-jdk-alpine

WORKDIR /app

COPY SimpleServer.java .

RUN javac SimpleServer.java

EXPOSE 8080

CMD ["java", "SimpleServer"]
```

### ⚙️ Workflow

```bash
docker build -t java-web .
docker run -d -p 8080:8080 --name java-web-container java-web
curl http://localhost:8080
```

> This image contains the full JDK even at run time. For smaller production images, use a **multi-stage build** (Module 6).

---

## 4.3 🗄️ MySQL

### 📁 Folder Structure

```text
mysql-docker/
├── Dockerfile
└── init.sql
```

### 📄 `init.sql` (optional)

```sql
CREATE TABLE IF NOT EXISTS users (
  id   INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(50) NOT NULL
);

INSERT INTO users (name) VALUES ('Alice'), ('Bob');
```

### 🧱 Dockerfile

```dockerfile
FROM mysql:8.4

ENV MYSQL_DATABASE=mydb

COPY init.sql /docker-entrypoint-initdb.d/

EXPOSE 3306
```

> **Why `mysql:8.4`?** MySQL 8.0 reached end-of-life on April 30, 2026. 8.4 is the current LTS line.
>
> **Why no password in the Dockerfile?** A password set with `ENV` is baked into the image and visible to anyone who can inspect it. Pass secrets at **run time** instead.

### ⚙️ Workflow

#### Build Image

```bash
docker build -t mysql-custom .
```

#### Run Container (with a persistent volume)

```bash
docker run -d \
  --name mysql-container \
  -e MYSQL_ROOT_PASSWORD=root \
  -v mysql_data:/var/lib/mysql \
  -p 3306:3306 \
  mysql-custom
```

> `root` as a password is fine for a local lab only. For anything real, use a strong password via `--env-file` or a secrets manager.

#### Wait Until MySQL Is Ready

```bash
docker logs -f mysql-container
```

Wait for the message `ready for connections` (first start takes longer because it initializes the database and runs `init.sql`).

#### Access MySQL

```bash
docker exec -it mysql-container mysql -uroot -p
```

```sql
SHOW DATABASES;
USE mydb;
SELECT * FROM users;
```

#### Backup and Restore (recommended)

```bash
# Backup
docker exec mysql-container sh -c 'mysqldump -uroot -p"$MYSQL_ROOT_PASSWORD" mydb' > mydb_backup.sql

# Restore
docker exec -i mysql-container sh -c 'mysql -uroot -p"$MYSQL_ROOT_PASSWORD" mydb' < mydb_backup.sql
```

#### Optional: Copy Raw Data Directory

```bash
docker cp mysql-container:/var/lib/mysql ./mysql_backup
```

> Copying the data directory of a **running** database can produce an inconsistent backup. Prefer `mysqldump` or Docker volumes.

### ⚠️ MySQL Gotchas

- Files in `/docker-entrypoint-initdb.d/` run **only on first start**, when the data directory is empty. To re-run `init.sql`, remove the volume: `docker rm -f mysql-container && docker volume rm mysql_data`.
- Some very old MySQL clients that depend on the `mysql_native_password` plugin may fail to connect to MySQL 8.4 (it is disabled by default there). Update the client or use the default authentication.

---

## 4.4 🌐 Apache Web Server

### 📁 Folder Structure

```text
apache-web/
├── Dockerfile
└── index.html
```

### 📄 `index.html`

```html
<!DOCTYPE html>
<html>
  <head><title>Apache in Docker</title></head>
  <body><h1>Hello from Apache inside Docker! 🐳</h1></body>
</html>
```

### 🧱 Dockerfile

```dockerfile
FROM httpd:2.4

WORKDIR /usr/local/apache2/htdocs/

COPY index.html .

EXPOSE 80
```

### ⚙️ Workflow

```bash
docker build -t apache-web .
docker run -d -p 80:80 --name apache-web-container apache-web
curl http://localhost
```

> If port 80 is already in use on your host, map another one, for example `-p 8080:80`, then open `http://localhost:8080`.

---

# 🧠 Module 5: CMD vs ENTRYPOINT

### `CMD`

Provides the **default command or default arguments**. It is easily overridden when running the container.

```dockerfile
CMD ["python", "app.py"]
```

```bash
docker run python-web python other.py     # replaces CMD completely
```

### `ENTRYPOINT`

Defines the **main executable** of the container. Arguments passed to `docker run` are **appended** to it.

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```bash
docker run --entrypoint /bin/sh myimage   # to override ENTRYPOINT, use --entrypoint
```

### 🤝 Using Both Together (best pattern)

`ENTRYPOINT` is the fixed program; `CMD` supplies its default arguments.

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

```bash
docker run myimage              # runs: python app.py
docker run myimage other.py     # runs: python other.py
```

### 📊 Comparison

| Feature | `CMD` | `ENTRYPOINT` |
|---|---|---|
| Purpose | Default command / default arguments | Fixed main executable |
| Overridden by `docker run image <args>` | Yes, fully replaced | No, arguments are appended |
| Override flag | Just pass a new command | `--entrypoint` |
| If written multiple times | Only the **last** one counts | Only the **last** one counts |
| Best form | Exec form | Exec form |

### Simple Interview Answer

> **CMD provides default startup instructions that can be overridden, while ENTRYPOINT defines the main executable of the container.**

---

# ⚡ Module 6: Build Optimization

## 6.1 Layer Caching: Order Matters

Docker rebuilds a layer (and every layer after it) when something above it changes. Put things that change **rarely** first and things that change **often** last.

```dockerfile
# ❌ Bad: any code change re-installs all dependencies
COPY . .
RUN pip install -r requirements.txt

# ✅ Good: dependencies stay cached until requirements.txt changes
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

## 6.2 `.dockerignore`

Keeps junk and secrets out of the build context (faster builds, smaller images, fewer leaks). Place it next to the Dockerfile.

```text
.git
.env
node_modules
__pycache__/
*.pyc
*.log
```

## 6.3 Multi-Stage Builds

Build in one stage, copy only the result into a small final image. Compilers and build tools never reach production.

```dockerfile
# Stage 1: build
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn -q clean package -DskipTests

# Stage 2: runtime (small JRE only)
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

> This example assumes your Maven project produces `target/app.jar` (for example by setting `<finalName>app</finalName>` in `pom.xml`).

## 6.4 Keep Images Small

- Use `-slim` or `-alpine` base images where compatible.
- Combine related commands into one `RUN` and clean up in the same layer.
- Use `--no-install-recommends` and delete package lists on Debian/Ubuntu.
- Never copy build output, `.git`, or test data into the image.

```dockerfile
RUN apt-get update \
 && apt-get install -y --no-install-recommends curl \
 && rm -rf /var/lib/apt/lists/*
```

Check layer sizes:

```bash
docker history <image-name>
docker images
```

---

# 🔒 Module 7: Best Practices & Security

## ✅ Checklist

| Do | Why |
|---|---|
| Pin image versions (`python:3.12-slim`) | `latest` changes silently and breaks builds |
| Use small, trusted base images | Smaller attack surface, faster pulls |
| Keep base images up to date and supported | End-of-life images get no security patches |
| Run as a non-root user (`USER`) | Limits damage if the app is compromised |
| Keep secrets out of `ENV`, `ARG`, and `COPY` | They end up in image layers and metadata |
| Use `.dockerignore` | Avoids leaking `.env`, `.git`, keys |
| Prefer `COPY` over `ADD` | More predictable |
| Use exec form for `CMD` / `ENTRYPOINT` | Proper signal handling for `docker stop` |
| Add a `HEALTHCHECK` | Docker can see when the app is unhealthy |
| One main process per container | Easier scaling, logging, restarts |
| Scan images (Docker Scout, Trivy) | Finds known vulnerabilities |

## 👤 Non-Root User

```dockerfile
# Debian/Ubuntu based images (python:slim, ubuntu)
RUN useradd --create-home appuser
USER appuser
```

```dockerfile
# Alpine based images
RUN adduser -D appuser
USER appuser
```

## ❤️ Health Check

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

- Exit `0` = healthy, exit `1` = unhealthy.
- `curl` must exist in the image. Slim/alpine images often do not include it; install it or use another command (see the Python example in Module 4.1).
- See health status with `docker ps` (the STATUS column shows `healthy` / `unhealthy`).

## 🔑 Build-Time Secrets (the safe way)

```dockerfile
# syntax=docker/dockerfile:1
RUN --mount=type=secret,id=api_token \
    API_TOKEN=$(cat /run/secrets/api_token) ./download-private-package.sh
```

```bash
docker build --secret id=api_token,src=./api_token.txt -t myapp .
```

The secret is available only during that `RUN` step and is **not** stored in the image layers.

## 🏷️ Why Avoid the `latest` Tag

- It is mutable: the same tag can point to different images tomorrow.
- Builds stop being reproducible.
- Prefer exact versions, or pin by digest for maximum safety.

---

# 🔍 Module 8: Useful Docker Commands

### 🖼️ Images

```bash
docker images                          # list images
docker build -t myapp:1.0 .            # build and tag
docker history myapp:1.0               # show layers and sizes
docker rmi myapp:1.0                   # remove an image
```

### 📦 Containers

```bash
docker ps                              # running containers
docker ps -a                           # all containers (including stopped)
docker run -d -p 8080:80 --name web nginx
docker stop web                        # graceful stop (SIGTERM, then SIGKILL after timeout)
docker start web
docker restart web
docker rm web                          # remove a stopped container
docker rm -f web                       # force remove a running container
```

### 🐞 Debugging

```bash
docker logs web                        # view logs
docker logs -f web                     # follow logs live
docker exec -it web /bin/bash          # shell inside the container
docker exec -it web /bin/sh            # use this if Bash is unavailable (Alpine)
docker inspect web                     # full low-level details (JSON)
docker stats                           # live CPU / memory usage
docker cp web:/etc/nginx/nginx.conf .  # copy a file out of a container
```

### 🌐 Registry (share your image)

```bash
docker login
docker tag myapp:1.0 <dockerhub-username>/myapp:1.0
docker push <dockerhub-username>/myapp:1.0
docker pull <dockerhub-username>/myapp:1.0
```

### 🧹 Cleanup

```bash
docker system df                       # see how much space Docker uses
docker container prune                 # remove all stopped containers
docker image prune                     # remove dangling images
docker volume ls                       # list volumes
docker volume rm <volume-name>
docker system prune                    # remove unused containers, networks, dangling images
```

> ⚠️ `docker system prune -a` also removes **all unused images**. Read the confirmation prompt carefully.

### 🔗 Connecting Two Containers (for example Flask + MySQL)

Containers on the same **user-defined network** can reach each other by container name.

```bash
docker network create app-net
docker run -d --name mysql-container --network app-net -e MYSQL_ROOT_PASSWORD=<strong-password> mysql-custom
docker run -d --name python-web-container --network app-net -p 5000:5000 python-web
# Inside python-web-container, the database host is simply: mysql-container
```

---

# 🩺 Module 9: Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| `port is already allocated` | Host port is already used by another process or container | Use another host port (`-p 8081:5000`) or stop the other container |
| Container exits immediately | The main process (PID 1) finished or crashed | Run `docker logs <name>`; make sure `CMD` starts a long-running foreground process |
| App works inside the container but not in the browser | App listens on `127.0.0.1`, or `-p` is missing | Bind to `0.0.0.0` and publish the port with `-p` |
| Works locally but not from the internet (cloud VM) | Security Group / firewall blocks the port | Open the host port in the Security Group / firewall |
| `Conflict. The container name is already in use` | An old container with that name still exists | `docker rm -f <name>` and run again |
| `COPY failed: file not found` | File is outside the build context, misspelled, or listed in `.dockerignore` | Check the path, run `docker build` from the right folder, review `.dockerignore` |
| Code changes not visible | Old image or old container is still being used | Rebuild the image and recreate the container |
| Build uses stale layers | Cache reused | `docker build --no-cache -t <name> .` |
| `permission denied` on `/var/run/docker.sock` | User is not allowed to use Docker | Add the user to the `docker` group (then log in again) or use `sudo` |
| `/bin/bash: not found` | Alpine images have no Bash | Use `/bin/sh` |
| `exec format error` | Image built for a different CPU architecture, or script missing a shebang | Build with `--platform`, or add `#!/bin/sh` to scripts |
| `no space left on device` | Old images, containers, volumes filled the disk | `docker system df`, then prune what you no longer need |
| MySQL `init.sql` did not run | Data directory was not empty (volume already initialized) | Remove the volume to re-initialize |
| Need full build output to find a failing step | Default output is folded | `docker build --progress=plain --no-cache .` |

---

# 🎯 Module 10: Interview Revision

## 10.1 Quick Revision

| Question | Short answer |
|---|---|
| What is a Dockerfile? | A text file containing instructions to build a Docker image. |
| What is `FROM`? | Defines the base image. |
| What is `RUN`? | Executes commands during image build. |
| What is `COPY`? | Copies files from the build context into the image. |
| COPY vs ADD? | ADD can also auto-extract local tar files and fetch URLs; prefer COPY. |
| What is `WORKDIR`? | Sets the working directory. |
| What is `CMD`? | Default command when a container starts; can be overridden. |
| What is `ENTRYPOINT`? | The main executable of the container; arguments are appended. |
| What is `EXPOSE`? | Documents the port; does not publish it. |
| What is `ENV`? | Environment variables available at build time and run time. |
| What is `ARG`? | Build-time variables only. |
| What is `VOLUME`? | A mount point for persistent data. |
| What is `USER`? | Sets the user for later instructions and the running process. |
| What is `HEALTHCHECK`? | A command Docker runs to check whether the app is healthy. |
| Image vs container? | Image is the template; container is a running instance of it. |
| What is a layer? | A cached filesystem change created by RUN, COPY, or ADD. |
| What is the build context? | The folder sent to Docker during build (the `.` in `docker build`). |
| What is `.dockerignore`? | Lists files that must not be sent into the build context. |
| What is a multi-stage build? | Several FROM stages; only the final stage ships, giving small images. |
| Why avoid `latest`? | It is mutable and makes builds unpredictable. |

## 10.2 Scenario-Based Questions

**1. A container exits immediately after `docker run -d`. What do you check?**
The container lives only as long as its main process (PID 1). Run `docker logs <name>` to see why it stopped, and make sure `CMD`/`ENTRYPOINT` starts a long-running foreground process (not a background daemon that returns immediately).

**2. The Flask app runs fine in the container but the browser cannot reach it. Why?**
Usually one of three things: the app binds to `127.0.0.1` instead of `0.0.0.0`, the port was not published with `-p`, or (on a cloud VM) the Security Group/firewall blocks the port.

**3. You changed `app.py` but the running container still shows the old version. Why?**
A container is created from an image snapshot. Rebuild the image, remove the old container, and run a new one from the new image.

**4. Every small code change makes the build reinstall all dependencies. How do you fix this?**
Reorder the Dockerfile: copy `requirements.txt` and run `pip install` first, then copy the application code. The dependency layer stays cached until `requirements.txt` changes.

**5. Your image is over 1 GB. How do you shrink it?**
Use a slim/alpine base, use a multi-stage build, add a `.dockerignore`, combine `RUN` commands and clean package caches in the same layer, and avoid copying unnecessary files.

**6. How do you pass a database password to a container securely?**
Not in the Dockerfile (`ENV`/`ARG` end up in image metadata). Pass it at run time with `-e`, `--env-file`, or (better) Docker/Kubernetes secrets or a secrets manager. For build-time secrets use BuildKit `--mount=type=secret`.

**7. You want a container that always runs `python`, but lets users choose the script. How?**
`ENTRYPOINT ["python"]` plus `CMD ["app.py"]`. Running the image with no arguments runs `app.py`; `docker run image other.py` runs `other.py`.

**8. How do you run the application as a non-root user?**
Create a user with `RUN useradd ...` (or `adduser` on Alpine) and switch with `USER appuser` before `CMD`.

**9. Data inside a database container disappears when the container is removed. Why, and what is the fix?**
Container filesystems are ephemeral. Mount a named volume (`-v mysql_data:/var/lib/mysql`) so data survives container removal.

**10. How do you use the same image in dev, staging, and production with different settings?**
Keep the image identical and change behavior through environment variables (`-e`, `--env-file`) or mounted config files, not by rebuilding per environment.

**11. Why is using `FROM python:latest` risky?**
`latest` can change at any time, so tomorrow's build may use a different Python version and break. Pin an exact version such as `python:3.12-slim`.

**12. A container is running but the app inside is broken. How can Docker detect this?**
Add a `HEALTHCHECK`. Docker then marks the container `healthy` or `unhealthy`, visible in `docker ps`, and orchestrators can restart or replace unhealthy containers.

**13. How do you reduce the attack surface of an image?**
Minimal base image, non-root user, no secrets in layers, pinned and up-to-date versions, remove build tools via multi-stage builds, scan the image with Trivy or Docker Scout.

**14. Why does `docker stop` sometimes take 10 seconds?**
`docker stop` sends SIGTERM, waits (10 seconds by default), then sends SIGKILL. Shell-form `CMD` often does not forward SIGTERM to the app, so it waits for the timeout. Use exec form so the app receives the signal directly.

**15. Why use a multi-stage build?**
Compilers, SDKs, and build caches are needed to build the app but not to run it. A multi-stage build copies only the final artifact into a small runtime image, making it smaller and safer.

**16. A build step fails and the output is hard to read. How do you debug it?**
Run `docker build --progress=plain --no-cache .` to see full, unfolded output for every step.

**17. Two containers (Flask and MySQL) must talk to each other. How?**
Put both on the same user-defined network (`docker network create app-net`, `--network app-net`). Flask can then use the container name (for example `mysql-container`) as the database host. For many containers, use Docker Compose.

**18. `COPY` fails with "file not found" even though the file exists on your machine. Why?**
The file is outside the build context, the path is wrong, or it is excluded by `.dockerignore`. Run `docker build` from the correct folder and check `.dockerignore`.

**19. When would you actually use `ADD` instead of `COPY`?**
Only when you want a local `.tar`/`.tar.gz` automatically extracted into the image. For everything else, use `COPY`.

**20. How do you update a running containerized application?**
Containers are immutable. Build a new image version, stop and remove the old container, and start a new one from the new image (orchestrators such as Kubernetes or Docker Compose automate rolling updates).

---

# 🚀 Module 11: Cheat Sheet & Next Steps

## 🧾 One-Page Dockerfile Template

```dockerfile
FROM python:3.12-slim              # pinned, supported base
WORKDIR /app                       # working directory
COPY requirements.txt .            # dependencies first (cache friendly)
RUN pip install --no-cache-dir -r requirements.txt
COPY . .                           # then the code
RUN useradd --create-home appuser  # non-root user
USER appuser
EXPOSE 5000                        # documentation of the port
CMD ["python", "app.py"]           # exec form
```

## 🔁 Complete Flow

```text
Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
docker run
    ↓
Docker Container
    ↓
Application
```

## ⭐ Remember

> **Dockerfile → Build → Image → Run → Container → Application**

## 🧭 What to Learn Next

1. **Docker volumes and networks** in depth
2. **Docker Compose** to run multi-container apps (Flask + MySQL) with one file
3. **Pushing images** to Docker Hub / AWS ECR and pulling them on a server
4. **Image scanning** (Trivy, Docker Scout) and CI/CD builds (GitHub Actions, Jenkins)
5. **Kubernetes basics** to run containers at scale
