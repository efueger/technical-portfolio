# Kubernetes Troubleshooting Reference

Quick diagnostic reference for resolving common Kubernetes issues in customer support scenarios.

## Diagnostic Workflow

```
1. kubectl get pods              → Check status
2. kubectl describe pod <n>      → Read events
3. kubectl logs <pod>            → Application logs
4. kubectl exec -it <pod> -- sh  → Interactive debugging
5. kubectl get services          → Verify networking
```

## Common Pod Statuses

### Running

Container executing successfully. Check logs for application-level errors.

### Pending

Waiting to be scheduled.

**Causes:** Insufficient resources, image pull in progress, node selector mismatch, volume issues

**Fix:** Check `kubectl describe pod` Events section for scheduling failures

### ImagePullBackOff

Cannot pull container image.

**Causes:** Wrong image/tag, private registry without credentials, network issues, rate limiting

**Fix:**

```bash
docker pull <image>  # Verify locally
kubectl create secret docker-registry regcred --docker-server=<registry>
```

### CrashLoopBackOff

Container starts then exits; Kubernetes backing off retries.

**Causes:** Application exits (missing config/env vars), health check failures, OOMKilled

**Debug:**

```bash
kubectl logs <pod>           # Current logs
kubectl logs <pod> --previous # Before crash
```

### OOMKilled

Memory limit exceeded.

**Identify:** `kubectl describe pod` shows Exit Code: 137

**Fix:** Increase memory limit or investigate memory leak

## Essential Commands

```bash
# Status
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <n>

# Logs
kubectl logs <pod>
kubectl logs <pod> -c <container>
kubectl logs <pod> --previous
kubectl logs <pod> -f

# Debug
kubectl exec -it <pod> -- sh
kubectl exec <pod> -- env

# Scale
kubectl scale deployment <n> --replicas=3

# Network
kubectl get services
kubectl get endpoints <service>
```

## Support Scenarios

### "My workflow can't reach my API"

1. Verify pod running: `kubectl get pods`
2. Check service exists: `kubectl get services`
3. Test from within cluster: `kubectl run curl-test --image=curlimages/curl -i --rm -- curl http://service:8080`
4. Check service selector matches pod labels

**Common causes:** Service selector mismatch, port mismatch, NetworkPolicy blocking traffic

### "How do I store secrets?"

**Basic:** `kubectl create secret generic keys --from-literal=api-key=value`  
**Production:** AWS Secrets Manager with External Secrets Operator  
**Enterprise:** HashiCorp Vault

### "Works locally, not in Kubernetes"

**Differences:**

- `localhost` in Docker ≠ `localhost` in pod (use Service DNS)
- Environment variables must be configured in deployment
- Resource limits not present locally
- Different dependency addresses

**Debug:** `kubectl describe pod` for config, `kubectl logs` for errors, `kubectl exec` for interactive testing

## AWS Fargate Notes

Serverless containers - AWS provisions compute per pod.

**Requirements:**

- Explicit CPU/memory requests matching Fargate sizes
- VPC networking with proper security groups
- NAT gateway for internet access

**Limitations:** No DaemonSets, HostPath volumes, or privileged containers

## Troubleshooting Tips

**Events first:** `kubectl describe pod` Events section shows Kubernetes decisions

**Use --previous for crashes:** Shows logs before restart (critical for CrashLoopBackOff)

**Interactive debugging:** `kubectl exec -it <pod> -- sh` tests commands inside container

**Labels must match:** Service selectors must match pod labels for routing

**Pods are ephemeral:** Deleting pod doesn't fix underlying problem - fix deployment spec

---

## Related

- [Kubernetes Fundamentals](./kubernetes/README.md) - Core concepts and hands-on labs
- [Workflow Automation](../labs/workflow-automation/) - Automation patterns applicable to Kubernetes monitoring and orchestration
