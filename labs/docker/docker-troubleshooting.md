# Docker Troubleshooting Reference

Quick diagnostic reference for resolving common Docker container issues in customer support scenarios.

## Diagnostic Workflow

```
1. docker ps -a                  → Check container status and exit codes
2. docker logs <container>       → Review application output and errors
3. docker inspect <container>    → Examine configuration (env vars, mounts, networks)
4. docker exec -it <container> sh → Interactive debugging inside running container
5. docker ps --format "table {{.Names}}\t{{.Ports}}" → Verify port mappings
```

## Common Container Statuses

### Up

Container running successfully. Check logs for application-level issues if behavior is unexpected.

### Exited (0)

Container stopped after command completed successfully.

**Cause:** CMD was one-shot command (echo, script that exits, npm install)

**Fix:** Change CMD to long-running process (web server, tail -f, sleep infinity)

### Exited (1)

Container stopped due to error.

**Causes:** Application crash, missing dependencies, configuration errors, missing environment variables

**Debug:**

```bash
docker logs <container>  # Check error messages
docker inspect <container> | grep -A 20 Env  # Verify env vars
```

### Exited (137)

Container killed by system (out of memory).

**Identify:** Exit code 137 in STATUS column

**Fix:** 

- Increase Docker memory limit (Docker Desktop → Settings → Resources)
- Give container more memory: `docker run -m 2g <image>`
- Investigate application memory leaks

### Exited (139)

Segmentation fault (application tried to access invalid memory).

**Cause:** Application bug, typically in C/C++ programs

**Fix:** Debug application code

### Created

Container created but never started.

**Causes:** Port conflict, volume mount error, image pull failure

**Fix:** Remove failed container (`docker rm <container>`), resolve underlying issue, recreate

## Essential Commands

```bash
# Status
docker ps                           # Running containers
docker ps -a                        # All containers (including stopped)
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Logs
docker logs <container>             # View output
docker logs <container> -f          # Follow logs (live tail)
docker logs <container> --tail 50   # Last 50 lines

# Inspection
docker inspect <container>                    # Full config JSON
docker inspect <container> | grep -A 20 Env   # Environment variables
docker exec <container> env                   # Env vars inside container

# Debugging
docker exec -it <container> sh      # Interactive shell
docker exec <container> ps aux      # Running processes
docker exec <container> ls -la /app # Check files

# Management
docker stop <container>             # Gracefully stop
docker start <container>            # Start stopped container
docker restart <container>          # Stop then start
docker rm <container>               # Delete stopped container
docker rm -f <container>            # Force delete (stops and deletes)

# Cleanup
docker container prune              # Remove all stopped containers
docker volume prune                 # Remove unused volumes
docker system prune                 # Remove stopped containers, unused networks, dangling images
```

## Support Scenarios

### "Container keeps stopping immediately"

**Diagnostic sequence:**

1. `docker ps -a` - Check exit code
2. `docker logs <container>` - Check output before exit
3. Ask: "What's your CMD in your Dockerfile?"

**If exit code 0:** CMD is one-shot command (completes and exits)

- **Fix:** Change CMD to long-running process

**If exit code 1:** Application error

- **Fix:** Check logs for specific error, verify configuration

**If exit code 137:** Out of memory

- **Fix:** Increase memory allocation or investigate memory usage

### "Can't start container - port already in use"

**Error:** `Bind for 0.0.0.0:8080 failed: port is already allocated`

**Diagnostic sequence:**

```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"  # Which container uses port?
lsof -i :8080  # Non-Docker process? (Mac/Linux)
netstat -ano | findstr :8080  # Non-Docker process? (Windows)
```

**Solutions:**

- Use different host port: `-p 8081:80` instead of `-p 8080:80`
- Stop conflicting container: `docker stop <container>`
- Auto-assign port: `-p 80` (Docker picks available port)

### "Application says configuration is missing"

**Symptoms:** Exit code 1, logs show "ERROR: Missing API_KEY" or similar

**Diagnostic sequence:**

```bash
docker logs <container>  # Confirm missing config error
docker inspect <container> | grep -A 20 Env  # Check env vars
```

**Causes:**

- Environment variables not passed at runtime
- Wrong variable names
- Variables set in Dockerfile but not at runtime

**Fix:** Pass env vars with `-e` flag:

```bash
docker run -e API_KEY=value -e DB_HOST=localhost <image>
```

### "Data disappears when container restarts"

**Cause:** Data written inside container filesystem (not a volume)

**Fix:** Use volumes for persistent data:

```bash
docker run -v /host/path:/container/path <image>  # Bind mount
docker volume create mydata                       # Named volume
docker run -v mydata:/container/path <image>
```

**Verify mount:**

```bash
docker exec <container> ls -la /container/path
docker inspect <container> | grep -A 10 Mounts
```

### "Volume mount is empty or wrong files"

**Causes:**

- Relative path instead of absolute path (creates new volume)
- Wrong host path
- Permission issues

**Debug:**

```bash
docker exec <container> ls -la /mount/point  # Files visible in container?
docker volume ls  # Check for unexpected volumes
```

**Fix:** Use absolute paths:

```bash
# Wrong (relative path)
docker run -v ./data:/app/data <image>

# Correct (absolute path)
docker run -v /Users/name/data:/app/data <image>
docker run -v $(pwd)/data:/app/data <image>
```

### "Works locally, fails in container"

**Common differences:**

- **localhost inside container ≠ localhost on host**
  - Use host.docker.internal to reach host services
  - Use container names on same Docker network
- **Environment variables must be explicit**
  - No automatic inheritance from host
  - Must pass with `-e` or env file
- **File paths differ**
  - Container has own filesystem
  - Host paths not accessible without volume mounts
- **Resource limits**
  - Container may have CPU/memory limits
  - Check with `docker stats <container>`

**Debug:**

```bash
docker exec <container> env              # Check environment
docker exec <container> curl localhost   # Test networking
docker exec <container> ls -la /path     # Verify files present
```

## Port Mapping Reference

```bash
-p 8080:80           # Host port 8080 → container port 80
-p 127.0.0.1:8080:80 # Only localhost can access
-p 80                # Docker assigns random host port
```

**Key concepts:**

- Format: `-p HostPort:ContainerPort`
- Only one container per host port
- Multiple containers can use same container port
- `EXPOSE` in Dockerfile is documentation only (doesn't publish port)

## Common Mistakes

**Using EXPOSE without -p:**

```dockerfile
EXPOSE 3000  # This is just documentation
```

Still need: `docker run -p 3000:3000 <image>`

**Forgetting -d flag:**

```bash
docker run nginx  # Blocks terminal
docker run -d nginx  # Runs in background
```

**Removing container instead of stopping:**

```bash
docker rm <container>  # Error: container is running
docker stop <container> && docker rm <container>  # Correct sequence
docker rm -f <container>  # Force (stops and removes)
```

**Not checking exit codes:**

```bash
docker ps  # Only shows running containers
docker ps -a  # Shows all containers with exit codes
```

## Troubleshooting Tips

**Exit codes first:** `docker ps -a` STATUS column immediately shows why container stopped

**Logs are critical:** Most errors reveal themselves in `docker logs <container>`

**Inspect when confused:** `docker inspect <container>` shows complete configuration

**Interactive debugging:** `docker exec -it <container> sh` tests commands inside container

**Clean up regularly:** Stopped containers and unused volumes accumulate quickly

---

## Related

- [Docker Fundamentals](./README.md) - Core concepts and hands-on labs
- [Workflow Automation](../workflow-automation/) - Automation patterns applicable to container monitoring
