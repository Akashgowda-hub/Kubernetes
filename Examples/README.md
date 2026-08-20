# Kubernetes Live Examples Repository

Complete collection of practical, production-ready YAML examples for all Kubernetes concepts.

## 📁 Directory Structure

```
Examples/
├── 01-Controllers/          # Deployments, ReplicaSets, DaemonSets, Jobs
│   ├── replicaset-basic.yml
│   ├── replicaset-advanced.yml
│   ├── deployment-basic.yml
│   ├── deployment-rolling-update.yml
│   ├── deployment-recreate.yml
│   ├── deployment-production.yml
│   ├── daemonset-basic.yml
│   ├── daemonset-with-nodeaffinity.yml
│   ├── job-basic.yml
│   ├── cronjob-basic.yml
│   └── README.md
│
├── 02-Services/             # All service types and networking
│   ├── service-clusterip.yml
│   ├── service-nodeport.yml
│   ├── service-loadbalancer.yml
│   ├── service-externalname.yml
│   ├── service-headless.yml
│   ├── service-multiport.yml
│   ├── service-with-sessionaffinity.yml
│   └── README.md
│
├── 03-Probes/               # Health checks (Startup, Readiness, Liveness)
│   ├── probe-http.yml
│   ├── probe-tcp.yml
│   ├── probe-exec.yml
│   ├── probe-all-types.yml
│   └── README.md
│
├── 04-Pods/                 # Pod patterns and configurations
│   ├── pod-simple.yml
│   ├── pod-simple-with-resources.yml
│   ├── pod-multicontainer.yml
│   ├── pod-sidecar.yml
│   ├── pod-initcontainer.yml
│   └── README.md
│
├── 05-Advanced/             # Taints, Affinity, Lifecycle, Quotas
│   ├── taints-tolerations.yml
│   ├── lifecycle-hooks.yml
│   ├── resource-quota.yml
│   └── README.md
│
├── 06-ConfigMaps-Secrets/   # Configuration and secret management
│   ├── configmap-examples.yml
│   ├── secret-examples.yml
│   └── README.md
│
└── README.md                # This file
```

## 🚀 Quick Start Guide

### 1. Explore by Topic

Each folder contains practical examples:

```bash
# View Controllers examples
cd Examples/01-Controllers/
cat README.md

# View available Services
ls -la Examples/02-Services/

# Check Pod patterns
cat Examples/04-Pods/README.md
```

### 2. Apply Examples

```bash
# Apply a single example
kubectl apply -f Examples/01-Controllers/deployment-basic.yml

# Apply all examples in a directory
kubectl apply -f Examples/02-Services/

# Apply specific namespace examples
kubectl apply -f Examples/05-Advanced/resource-quota.yml -n team-dev
```

### 3. View Resource Status

```bash
# Check Deployment status
kubectl get deployment -f Examples/01-Controllers/deployment-basic.yml
kubectl describe deployment nginx-deployment

# Check Service endpoints
kubectl get svc -f Examples/02-Services/service-clusterip.yml
kubectl get endpoints

# Monitor Pods
kubectl get pods -o wide
kubectl logs <pod-name>
```

## 📚 Examples by Use Case

### 🏗️ **Web Application Deployment**

```bash
# 1. Create deployment with rolling updates
kubectl apply -f Examples/01-Controllers/deployment-rolling-update.yml

# 2. Expose with service
kubectl apply -f Examples/02-Services/service-loadbalancer.yml

# 3. Add health checks
kubectl apply -f Examples/03-Probes/probe-all-types.yml

# 4. Configure resources and quotas
kubectl apply -f Examples/05-Advanced/resource-quota.yml
```

### 🗄️ **Database Deployment**

```bash
# 1. Create StatefulSet (in deployment file)
kubectl apply -f Examples/04-Pods/pod-simple-with-resources.yml

# 2. Create headless service
kubectl apply -f Examples/02-Services/service-headless.yml

# 3. Add TCP probes
kubectl apply -f Examples/03-Probes/probe-tcp.yml

# 4. Configure secrets for credentials
kubectl apply -f Examples/06-ConfigMaps-Secrets/secret-examples.yml
```

### 📦 **Batch Job/Scheduled Task**

```bash
# 1. One-time job
kubectl apply -f Examples/01-Controllers/job-basic.yml

# 2. Scheduled CronJob
kubectl apply -f Examples/01-Controllers/cronjob-basic.yml

# 3. Monitor job status
kubectl get jobs
kubectl describe job database-backup
```

### 📊 **Logging & Monitoring Stack**

```bash
# 1. Deploy DaemonSet for log collection
kubectl apply -f Examples/01-Controllers/daemonset-basic.yml

# 2. Deploy with sidecar collectors
kubectl apply -f Examples/04-Pods/pod-sidecar.yml

# 3. Expose metrics
kubectl apply -f Examples/02-Services/service-multiport.yml
```

### 🔐 **Production Security Setup**

```bash
# 1. Configure taints on production nodes
kubectl apply -f Examples/05-Advanced/taints-tolerations.yml

# 2. Set resource quotas per team
kubectl apply -f Examples/05-Advanced/resource-quota.yml

# 3. Use secrets for credentials
kubectl apply -f Examples/06-ConfigMaps-Secrets/secret-examples.yml

# 4. Configure lifecycle hooks for graceful shutdown
kubectl apply -f Examples/05-Advanced/lifecycle-hooks.yml
```

## 🎯 Key Concepts Reference

### Controllers
| Type | File | Use Case |
|------|------|----------|
| Deployment | `deployment-*.yml` | Web apps, APIs, stateless services |
| DaemonSet | `daemonset-*.yml` | Node monitoring, logging, networking |
| Job | `job-basic.yml` | One-time batch tasks |
| CronJob | `cronjob-basic.yml` | Scheduled tasks |
| ReplicaSet | `replicaset-*.yml` | Pod replication (rarely used directly) |

### Services
| Type | File | Access | Use Case |
|------|------|--------|----------|
| ClusterIP | `service-clusterip.yml` | Internal only | Inter-pod communication |
| NodePort | `service-nodeport.yml` | External (port:node) | Testing, internal access |
| LoadBalancer | `service-loadbalancer.yml` | External (LB IP) | Production exposure |
| Headless | `service-headless.yml` | Direct pod IPs | StatefulSets, DNS |
| ExternalName | `service-externalname.yml` | DNS alias | External service integration |

### Probes
| Type | File | Decision | Impact |
|------|------|----------|--------|
| Startup | `probe-*.yml` | App initialized? | Blocks readiness/liveness |
| Readiness | `probe-*.yml` | Ready for traffic? | Remove from endpoints |
| Liveness | `probe-*.yml` | Still alive? | Restart container |

## 🔧 Common Commands

### Applying Resources

```bash
# Apply single file
kubectl apply -f filename.yml

# Apply all files in directory
kubectl apply -f ./directory/

# Apply with dry-run
kubectl apply -f filename.yml --dry-run=client

# Show what would change
kubectl diff -f filename.yml
```

### Viewing Resources

```bash
# List resources
kubectl get deployment
kubectl get svc
kubectl get pods -o wide

# Describe details
kubectl describe deployment <name>
kubectl describe svc <name>
kubectl describe pod <name>

# View resource YAML
kubectl get deployment <name> -o yaml
```

### Managing Deployments

```bash
# Scale deployment
kubectl scale deployment <name> --replicas=5

# Update image
kubectl set image deployment/<name> <container>=<image>:<tag>

# Check rollout status
kubectl rollout status deployment/<name>

# Rollback to previous version
kubectl rollout undo deployment/<name>
```

### Debugging

```bash
# View logs
kubectl logs <pod-name>
kubectl logs <pod-name> -f  # Follow logs

# Execute command in pod
kubectl exec -it <pod-name> -- /bin/bash

# Port forward
kubectl port-forward svc/<service-name> 8080:80

# Debug pod
kubectl debug pod/<pod-name> -it --image=busybox
```

## 📋 Checklist for Production Deployment

Use these examples to build production-ready applications:

- [ ] **Deployment** - Use rolling update strategy
  ```bash
  kubectl apply -f Examples/01-Controllers/deployment-rolling-update.yml
  ```

- [ ] **Service** - Expose with appropriate type
  ```bash
  kubectl apply -f Examples/02-Services/service-loadbalancer.yml
  ```

- [ ] **Health Probes** - All three probe types
  ```bash
  kubectl apply -f Examples/03-Probes/probe-all-types.yml
  ```

- [ ] **Resources** - Requests and limits
  ```bash
  # See deployment-production.yml for example
  ```

- [ ] **Lifecycle Hooks** - Graceful shutdown
  ```bash
  kubectl apply -f Examples/05-Advanced/lifecycle-hooks.yml
  ```

- [ ] **ConfigMap/Secret** - Externalized configuration
  ```bash
  kubectl apply -f Examples/06-ConfigMaps-Secrets/
  ```

- [ ] **Resource Quotas** - Namespace limits
  ```bash
  kubectl apply -f Examples/05-Advanced/resource-quota.yml
  ```

- [ ] **Node Affinity** - Pin to specific nodes if needed
  ```bash
  kubectl apply -f Examples/05-Advanced/taints-tolerations.yml
  ```

## 🔗 Related Documentation

- [Main Documentation](../README.md)
- [Installation Guide](../02-Installation/README.md)
- [Core Concepts](../03-Core-Concepts/README.md)
- [Cheatsheet & Commands](../08-Cheatsheet/README.md)

## 💡 Tips & Tricks

### 1. Validate YAML Before Applying
```bash
kubectl apply -f filename.yml --dry-run=client --validate=true
```

### 2. Test with a Single Replica First
```bash
kubectl apply -f deployment.yml --record
kubectl rollout status deployment/<name>
```

### 3. Monitor Resource Usage
```bash
kubectl top nodes
kubectl top pods --sort-by=memory
```

### 4. Export Working Configuration
```bash
kubectl get pod <pod-name> -o yaml > exported-pod.yml
kubectl get deployment <name> -o yaml > exported-deployment.yml
```

### 5. Quick Testing with Port Forward
```bash
kubectl port-forward svc/<service-name> 8080:80
# Access via http://localhost:8080
```

## 📝 Notes

- All YAML files are production-ready but should be customized for your environment
- Replace placeholder values (e.g., image names, namespaces) as needed
- Test in development/staging before deploying to production
- Use namespace isolation for different environments
- Always define resource requests and limits
- Implement all three health probes (startup, readiness, liveness)
- Use secrets for sensitive data, never commit to git

---

**Last Updated**: 2026-08-20
**Version**: 1.0
**Kubernetes Compatibility**: 1.24+
