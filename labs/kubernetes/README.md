# Kubernetes Fundamentals

## Overview

Hands-on practice with Kubernetes core concepts, focusing on deployment patterns, debugging workflows, and integration touchpoints relevant to technical support and solutions engineering roles.

## What I Built

Four hands-on labs using Docker Desktop with Kubernetes enabled, demonstrating practical troubleshooting and operational patterns.

**Labs completed:**

1. Deploy and scale nginx (self-healing demonstration)
2. Debug ImagePullBackOff errors (systematic troubleshooting)
3. Access Kubernetes API (understanding HTTP integration)
4. Port-forward for debugging (temporary access patterns)

## Core Concepts

### Cluster and Nodes

**Cluster**: Complete Kubernetes environment (control plane + worker machines)  
**Node**: Single machine running containers

Docker Desktop provides single-node cluster for local development.

### Pod

Kubernetes wrapper around one or more containers that run together. In most cases: **Pod ≈ one running app instance**.

Three pods = three copies of the app running.

### Deployment

Controller that maintains desired number of pod replicas:

- Creates and manages pods
- Replaces failed pods automatically
- Handles rolling updates safely

**Key principle:** You declare desired state (3 replicas), Kubernetes maintains it.

### Service

Stable network endpoint and load balancer for pods:

- Fixed DNS name and virtual IP
- Load balances traffic across pod replicas
- Abstracts pod lifecycle (pods are ephemeral, Services are stable)

**Types:**

- `ClusterIP` - Internal only
- `NodePort` / `LoadBalancer` - External access

## Tines Architecture Mapping

How a Tines-style application maps to Kubernetes objects:

**Component → Kubernetes Objects**

**tines-app** (web/API)  
→ Deployment (3+ replicas) + ClusterIP Service  
Serves UI and HTTP API, scaled horizontally for traffic

**tines-worker** (background jobs)  
→ Deployment (5+ replicas)  
Processes stories and actions from queue, scaled for workload

**postgres** (database)  
→ StatefulSet + PersistentVolume + ClusterIP Service  
Single-writer with persistent storage, stable network identity

**redis** (queue/cache)  
→ Deployment + PersistentVolume + ClusterIP Service  
Job queue and short-lived state

**nginx** (load balancer)  
→ Deployment + LoadBalancer Service or Ingress  
TLS termination and traffic routing

**Key integration points:**

- Tines can call Kubernetes API (HTTP requests with token credentials)
- Tines can monitor pod status and trigger automation
- Tines workflows can orchestrate deployments, rollbacks, or diagnostics

## Hands-On Labs

### Lab 1: Deploy and Scale

```bash
# Create deployment
kubectl create deployment nginx-deployment --image=nginx:latest

# Scale to 3 replicas
kubectl scale deployment nginx-deployment --replicas=3

# Watch self-healing (delete a pod, watch replacement)
kubectl delete pod <pod-name>
kubectl get pods -w
```

**Demonstrates:** Declarative state management - Deployment maintains 3 replicas regardless of individual pod failures.

### Lab 2: Debug ImagePullBackOff

```bash
# Create broken deployment
kubectl create deployment broken-nginx --image=nginx:fake-tag

# Diagnose
kubectl get pods
kubectl describe pod -l app=broken-nginx
```

**Key learning:** Events section in `describe` shows why pod won't start. Logs are empty because container never ran.

**Triage pattern:**

1. `kubectl get pods` - High-level status
2. `kubectl describe pod` - Events reveal image/registry issues
3. Verify image name, tag, and registry access
4. Check imagePullSecrets if private registry

### Lab 3: Kubernetes API as HTTP

```bash
# Terminal 1: Start proxy
kubectl proxy

# Terminal 2: Query API
curl http://127.0.0.1:8001/api/v1/namespaces/default/pods
```

**Key insight:** Everything kubectl does is HTTP to Kubernetes API. Tines can interact with Kubernetes the same way it interacts with any API - HTTP requests with token credentials.

### Lab 4: Port-Forward for Debugging

```bash
# Create temporary tunnel
kubectl port-forward deployment/nginx-deployment 8080:80

# Access at http://localhost:8080
```

**Use case:** Debug internal services without changing production ingress or firewall. Temporary tunnel from laptop to pod.



## Technologies

- **Kubernetes** - Container orchestration platform
- **Docker Desktop** - Local Kubernetes cluster
- **kubectl** - Command-line interface for Kubernetes
- **nginx** - Web server for demonstration deployments

---

## Artifacts

### Folder Structure

### Screenshots

- **docker-desktop-containers-running.png** - Docker Desktop showing containers running
- **docker-containers-running-self-healing.png** - Terminal showing deployment scaled to 3 replicas, pod deletion, automatic replacement
- **broken-container-details-logs.png** - kubectl describe output showing ImagePullBackOff Events and diagnostic information
- **kubernetes-api.png** - curl output from Kubernetes API showing JSON pod list

<img width="2574" height="1560" alt="docker-containers-running" src="https://github.com/user-attachments/assets/7451457b-7bfe-4881-bdca-9a1ab3abe052" />

<img width="1298" height="586" alt="docker-containers-running-self-healing" src="https://github.com/user-attachments/assets/bb36c5a5-bb8c-4ce0-bc5e-b8e291074796" />

<img width="2152" height="2884" alt="broken-container-details-logs" src="https://github.com/user-attachments/assets/6d99eb74-0283-40cf-9ee1-b96d0ac2c325" />

<img width="1612" height="1304" alt="kubernetes-api" src="https://github.com/user-attachments/assets/1c6ca55e-c910-4f92-992d-25f65f369607" />



---

## Related Labs

- [Kubernetes Troubleshooting](./kubernetes-troubleshooting.md) - Quick reference for common issues
