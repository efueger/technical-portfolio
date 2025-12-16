# Docker Fundamentals

## Overview

Hands-on practice with Docker core concepts, focusing on container troubleshooting, debugging workflows, and operational patterns relevant to technical support and solutions engineering roles.

## What I Built

Five hands-on labs using Docker Desktop, demonstrating container troubleshooting and operational patterns.

**Labs completed:**

1. Container exits immediately (exit code diagnosis and resolution)
2. Port binding conflicts (port mapping and conflict resolution)
3. Missing environment variables (configuration debugging)
4. Volume mount issues (data persistence and path resolution)
5. Debugging running containers (interactive troubleshooting)

## Core Concepts

### Images and Containers

**Image**: Read-only template containing application code, runtime, libraries, and dependencies. Built from a Dockerfile.

**Container**: Running instance of an image. Multiple containers can run from the same image, each isolated from others.

**Key principle:** Images are blueprints, containers are running instances.

### Dockerfile Instructions

**FROM**: Specifies base image (starting point)  
**RUN**: Executes commands during image build (setup/installation)  
**COPY**: Adds files from host into image  
**WORKDIR**: Sets working directory for subsequent instructions  
**CMD**: Command that runs when container starts (main process)  
**EXPOSE**: Documents which ports application uses (documentation only)

### Port Mapping

Maps ports between host system and container: `-p HostPort:ContainerPort`

**Host Port**: External access point on your system  
**Container Port**: Internal port where application listens

Multiple containers can use identical container ports as long as they map to distinct host ports.

### Exit Codes

Containers return exit codes when they stop:

**0**: Process completed successfully  
**1**: Generic error (check logs for details)  
**137**: Killed by system (typically out of memory)  
**139**: Segmentation fault (application crash)

### Data Persistence

**Volumes**: Docker-managed storage that persists beyond container lifecycle  
**Bind Mounts**: Direct mapping of host directory into container  

Containers are ephemeral - data inside them is lost when container is removed unless stored in volumes.

## Hands-On Labs

### Lab 1: Container Exits Immediately

```bash
# Create container with one-shot command (exits immediately)
cat > Dockerfile.bad-exit << 'EOF'
FROM alpine:3.19
CMD ["echo", "Starting up..."]
EOF

docker build -f Dockerfile.bad-exit -t bad-exit .
docker run --name exit-test bad-exit
docker ps -a  # Shows Exited (0)

# Fix: Use long-running process
cat > Dockerfile.good-exit << 'EOF'
FROM alpine:3.19
CMD ["tail", "-f", "/dev/null"]
EOF

docker build -f Dockerfile.good-exit -t good-exit .
docker run -d --name exit-fixed good-exit
docker ps  # Shows Up X seconds
```

**Demonstrates:** Containers need long-running foreground processes to stay alive. One-shot commands (echo, scripts that exit) cause containers to stop with exit code 0 even though they "succeeded."

**Diagnostic pattern:**

1. `docker ps -a` - Check exit code in STATUS column
2. `docker logs <container>` - Review output before exit
3. Verify CMD is long-running process, not one-shot command

### Lab 2: Port Binding Conflicts

```bash
# Start first container on port 8080
docker run -d -p 8080:80 --name nginx1 nginx:alpine

# Attempt second container on same port (fails)
docker run -d -p 8080:80 --name nginx2 nginx:alpine
# Error: Bind for 0.0.0.0:8080 failed: port is already allocated

# Solution 1: Use different host port
docker run -d -p 8081:80 --name nginx2 nginx:alpine

# Solution 2: Stop conflicting container
docker stop nginx1
docker run -d -p 8080:80 --name nginx2-retry nginx:alpine

# Solution 3: Auto-assign ephemeral port
docker run -d -p 80 --name nginx3 nginx:alpine
docker ps  # Check PORTS column for assigned port
```

**Demonstrates:** Port conflicts occur at host level - only one process can bind to each host port. Multiple containers can use identical container ports if mapped to different host ports.

**Diagnostic commands:**

```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"  # See port mappings
lsof -i :8080  # Check what's using port (Mac/Linux)
netstat -ano | findstr :8080  # Check what's using port (Windows)
```

### Lab 3: Missing Environment Variables

```bash
# Create container requiring environment variables
cat > Dockerfile.env-demo << 'EOF'
FROM alpine:3.19
COPY check-env.sh /check-env.sh
RUN chmod +x /check-env.sh
CMD ["/check-env.sh"]
EOF

cat > check-env.sh << 'EOF'
#!/bin/sh
if [ -z "$API_KEY" ]; then
  echo "ERROR: API_KEY not set"
  exit 1
fi
if [ -z "$DB_HOST" ]; then
  echo "ERROR: DB_HOST not set"
  exit 1
fi
echo "All env vars present. Starting app..."
tail -f /dev/null
EOF

# Build and run without env vars (fails)
docker build -f Dockerfile.env-demo -t env-demo .
docker run --name env-fail env-demo
docker logs env-fail  # Shows ERROR messages

# Run with env vars (succeeds)
docker run -d --name env-success \
  -e API_KEY=test123 \
  -e DB_HOST=localhost \
  env-demo

docker ps  # Shows env-success running
docker exec env-success env | grep -E 'API_KEY|DB_HOST'
```

**Demonstrates:** Applications fail with exit code 1 when required configuration is missing. Environment variables must be passed at container runtime using `-e` flag.

**Diagnostic pattern:**

1. `docker logs <container>` - Check for config-related errors
2. `docker inspect <container> | grep -A 20 Env` - Verify env vars are set
3. `docker exec <container> env` - Inspect environment inside running container

**[SCREENSHOT PLACEHOLDER: env-fail-logs.png - docker logs showing missing env var errors]**

**[SCREENSHOT PLACEHOLDER: env-success-running.png - docker ps showing env-success container Up status]**

### Lab 4: Volume Mount Issues

```bash
# Create test data
mkdir -p ~/docker-labs/test-volume
echo "Hello from host" > ~/docker-labs/test-volume/test.txt

# Mount volume correctly (absolute path)
docker run -d --name volume-test \
  -v ~/docker-labs/test-volume:/data \
  alpine:3.19 tail -f /dev/null

docker exec volume-test cat /data/test.txt  # Shows host file content
docker exec volume-test sh -c "echo 'Written from container' >> /data/test.txt"
cat ~/docker-labs/test-volume/test.txt  # Shows both lines

# Common mistake: relative path creates new volume
docker run --name bad-mount -v ./test-volume:/data alpine:3.19 ls /data
docker volume ls  # Shows orphaned volume
```

**Demonstrates:** Volume mounts require absolute paths. Relative paths create new Docker-managed volumes instead of mounting host directories. Data written to volumes persists beyond container lifecycle.

**Diagnostic pattern:**

1. `docker exec <container> ls -la /mount/point` - Verify files are visible
2. Check if using absolute vs relative path in `-v` flag
3. `docker volume ls` - Identify orphaned volumes from incorrect mounts
4. Verify host directory permissions allow container user access

**[SCREENSHOT PLACEHOLDER: volume-mount-success.png - docker exec showing host file accessible in container]**

**[SCREENSHOT PLACEHOLDER: volume-ls-orphaned.png - docker volume ls showing unintended volume creation]**

### Lab 5: Debugging Running Containers

```bash
# Start container
docker run -d --name debug-demo nginx:alpine

# Get shell inside running container
docker exec -it debug-demo sh

# Inside container: inspect processes
ps aux

# Inside container: check files
ls -la /etc/nginx/
cat /etc/nginx/nginx.conf

# Inside container: test network
apk add curl
curl localhost

# Exit container shell
exit

# One-off commands without interactive shell
docker exec debug-demo ps aux
docker exec debug-demo cat /etc/nginx/nginx.conf
docker exec debug-demo wget -O- localhost
```

**Demonstrates:** Interactive debugging workflow for running containers. `docker exec` provides access to running container's filesystem, processes, and network without restarting.

**Use cases:**

- Verify application files are present and correct
- Check running processes and resource usage
- Test internal networking and connectivity
- Inspect logs and configuration files
- Debug environment variables and paths

**[SCREENSHOT PLACEHOLDER: docker-exec-ps.png - docker exec showing running processes inside container]**

**[SCREENSHOT PLACEHOLDER: docker-exec-files.png - docker exec showing nginx config file contents]**

## Technologies

- **Docker** - Container runtime and image management
- **Docker Desktop** - Local development environment
- **Alpine Linux** - Lightweight base image for demonstrations
- **nginx** - Web server for port mapping and debugging examples

---

## Artifacts

### Screenshots

- **lab-03-env-fail-logs.png** - docker logs showing missing environment variable errors
- **lab-03-env-success-running.png** - docker ps showing container with env vars running successfully
- **lab-04-volume-mount-success.png** - docker exec showing host file accessible in container
- **lab-04-volume-ls-orphaned.png** - docker volume ls showing unintended volume creation from relative path
- **lab-05-docker-exec-ps.png** - docker exec showing running processes inside container
- **lab-05-docker-exec-files.png** - docker exec showing nginx config file contents

---

## Related Labs

- [Docker Troubleshooting](./docker-troubleshooting.md) - Quick reference for common issues
