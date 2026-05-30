# Docker Troubleshooting Guide

## Overview

This guide covers the most common Docker issues, troubleshooting steps, solutions, debugging commands, and scenario-based interview questions frequently asked in DevOps interviews.

---

# 1. Container Exits Immediately

## Problem

Container starts and stops immediately.

### Cause

The main process inside the container finishes execution.

### Solution

```bash
docker run -it ubuntu bash
```

Keep the container alive:

```bash
docker run -d ubuntu tail -f /dev/null
```

Check logs:

```bash
docker logs <container_name>
```

---

# 2. Port Not Accessible

## Problem

Application is running but cannot be accessed.

### Cause

Port mapping is missing or firewall/security group blocks access.

### Solution

```bash
docker run -d -p 8080:80 nginx
```

Verify:

```bash
docker port <container_name>
```

Check:

* AWS Security Group
* Azure NSG
* GCP Firewall
* Linux Firewall

---

# 3. Image Not Found

## Error

```bash
manifest unknown
```

### Solution

```bash
docker pull ubuntu:22.04
```

Verify available images:

```bash
docker images
```

---

# 4. Container Cannot Access Internet

## Verify

```bash
docker exec -it container_name ping google.com
```

### Fix

```bash
sudo systemctl restart docker
```

Check network:

```bash
docker network inspect bridge
```

---

# 5. No Space Left On Device

## Check Usage

```bash
df -h
```

Docker usage:

```bash
docker system df
```

### Cleanup

```bash
docker container prune
docker image prune -a
docker volume prune
docker system prune -a
```

---

# 6. Permission Denied

## Cause

Docker user lacks permissions on mounted directory.

### Fix

```bash
sudo chmod -R 755 /data
```

or

```bash
sudo chown -R $USER:$USER /data
```

---

# 7. Cannot Connect To Docker Daemon

## Error

```bash
Cannot connect to the Docker daemon
```

### Check Docker Service

```bash
sudo systemctl status docker
```

Start:

```bash
sudo systemctl start docker
```

Enable:

```bash
sudo systemctl enable docker
```

---

# 8. Container Name Already Exists

## Error

```bash
Conflict. The container name is already in use
```

### Solution

```bash
docker ps -a
docker rm -f container_name
```

---

# 9. Port Already Allocated

## Error

```bash
Bind for 0.0.0.0:80 failed
```

### Solution

Find process:

```bash
docker ps
```

Stop container:

```bash
docker stop container_name
```

Use another port:

```bash
docker run -p 8080:80 nginx
```

---

# 10. Device Or Resource Busy

### Fix

```bash
docker stop container_name
docker rm -f container_name
```

Check volume:

```bash
docker volume ls
```

---

# 11. Docker Build Failure

## Common Error

```bash
COPY failed
```

### Fix

Verify file exists:

```bash
ls -la
```

Example:

```dockerfile
COPY ./app /usr/src/app
```

---

# 12. Mounts Denied

### Cause

Shared directory not configured.

### Fix

Docker Desktop:

```text
Settings
 → Resources
 → File Sharing
```

---

# 13. Slow Docker Build

### Cleanup Cache

```bash
docker builder prune
```

Use lightweight images:

```dockerfile
FROM alpine
```

---

# 14. Container Restart Loop

### Check Logs

```bash
docker logs container_name
```

Inspect:

```bash
docker inspect container_name
```

---

# 15. Docker Compose Issues

Validate:

```bash
docker compose config
```

Restart:

```bash
docker compose down
docker compose up -d
```

---

# 16. Too Many Open Files

Check:

```bash
ulimit -n
```

Increase:

```bash
ulimit -n 65535
```

---

# 17. Docker Pull Rate Limit Error

### Error

```bash
toomanyrequests
```

### Fix

```bash
docker login
```

---

# 18. Docker Login Failure

### Solution

```bash
docker logout
docker login
```

Verify internet connectivity.

---

# 19. Volume Data Not Persisting

Create volume:

```bash
docker volume create myvolume
```

Use volume:

```bash
docker run -v myvolume:/data nginx
```

---

# 20. DNS Resolution Failure

### Verify

```bash
docker exec -it container ping google.com
```

Check DNS:

```bash
cat /etc/resolv.conf
```

---

# 21. Container Cannot Reach Another Container

Check network:

```bash
docker network ls
```

Create network:

```bash
docker network create app-network
```

Attach containers.

---

# 22. High CPU Usage

Check:

```bash
docker stats
```

Inspect process:

```bash
top
htop
```

---

# 23. High Memory Usage

Check:

```bash
docker stats
```

Limit memory:

```bash
docker run -m 512m nginx
```

---

# 24. Docker Socket Permission Error

## Error

```bash
permission denied while trying to connect to docker daemon socket
```

### Fix

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

# 25. Docker Network Troubleshooting

List networks:

```bash
docker network ls
```

Inspect network:

```bash
docker network inspect bridge
```

---

# 26. Docker Logs Troubleshooting

View logs:

```bash
docker logs container_name
```

Follow logs:

```bash
docker logs -f container_name
```

---

# 27. Docker Image Cleanup

Remove dangling images:

```bash
docker image prune
```

Remove all unused:

```bash
docker image prune -a
```

---

# 28. Docker Volume Cleanup

List:

```bash
docker volume ls
```

Delete:

```bash
docker volume prune
```

---

# 29. Useful Troubleshooting Commands

## Containers

```bash
docker ps
docker ps -a
```

## Logs

```bash
docker logs container_name
```

## Resource Usage

```bash
docker stats
```

## Inspect

```bash
docker inspect container_name
```

## Networks

```bash
docker network ls
```

## Volumes

```bash
docker volume ls
```

## Cleanup

```bash
docker system prune -a
```

---

# 30. Scenario-Based Interview Questions

### Scenario 1

A container exits immediately after startup. How will you troubleshoot?

### Scenario 2

Application works inside container but not from browser.

### Scenario 3

Docker image build fails at COPY instruction.

### Scenario 4

Container cannot access internet.

### Scenario 5

Docker host disk is 100% full.

### Scenario 6

Container stuck in restart loop.

### Scenario 7

Port 80 already allocated.

### Scenario 8

Container name already exists.

### Scenario 9

Docker daemon is not running.

### Scenario 10

Volume data is lost after container deletion.

### Scenario 11

Container cannot communicate with another container.

### Scenario 12

Docker pull returns "manifest unknown".

### Scenario 13

Docker login fails in CI/CD pipeline.

### Scenario 14

Container shows high CPU usage.

### Scenario 15

Docker build takes too long.

### Scenario 16

Application logs are missing.

### Scenario 17

Container reports DNS lookup failure.

### Scenario 18

Permission denied while mounting volume.

### Scenario 19

Docker Compose deployment fails.

### Scenario 20

Too many open files error appears.

---

# Best Practices

* Always check logs first.
* Use named volumes.
* Use lightweight images.
* Remove unused images regularly.
* Monitor resource usage.
* Use health checks.
* Keep Docker updated.
* Use custom networks for applications.
* Avoid running containers as root.
* Follow least privilege principle.

