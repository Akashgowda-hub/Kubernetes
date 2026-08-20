# Quick Navigation Guide

Fast access to all Kubernetes repository content.

## 📂 Main Documentation (Start Here)

| Document | Purpose | Time |
|----------|---------|------|
| [README.md](README.md) | Overview and quick start | 5 min |
| [REPOSITORY-SUMMARY.md](REPOSITORY-SUMMARY.md) | Complete file listing and stats | 10 min |
| **[Examples/README.md](Examples/README.md)** | **Live YAML examples guide** | **15 min** |

## 📚 Learning Documents

### Foundational Topics
| Topic | File | Key Content | Level |
|-------|------|-------------|-------|
| Introduction | [01-Introduction/README.md](01-Introduction/README.md) | What is Kubernetes, architecture | ⭐ |
| Installation | [02-Installation/README.md](02-Installation/README.md) | kubeadm, containerd, Calico | ⭐ |
| Core Concepts | [03-Core-Concepts/README.md](03-Core-Concepts/README.md) | Pods, lifecycle, labels | ⭐⭐ |

### Operational Topics
| Topic | File | Key Content | Level |
|-------|------|-------------|-------|
| Controllers | [04-Controllers/README.md](04-Controllers/README.md) | Deployment, DaemonSet, Job | ⭐⭐ |
| Services | [05-Services/README.md](05-Services/README.md) | All service types, networking | ⭐⭐ |
| Probes | [06-Probes/README.md](06-Probes/README.md) | Health checks, startup/ready/live | ⭐⭐ |

### Advanced Topics
| Topic | File | Key Content | Level |
|-------|------|-------------|-------|
| Advanced | [07-Advanced/README.md](07-Advanced/README.md) | Taints, affinity, lifecycle | ⭐⭐⭐ |
| Cheatsheet | [08-Cheatsheet/README.md](08-Cheatsheet/README.md) | 100+ kubectl commands | ⭐ |
| YAML Lib | [08-Cheatsheet/YAML-EXAMPLES.md](08-Cheatsheet/YAML-EXAMPLES.md) | Complete YAML template library | ⭐⭐ |

## 💻 Live YAML Examples

### Organized by Feature

#### Controllers (11 files)
```
Examples/01-Controllers/
├── replicaset-basic.yml              # Simple replica set
├── replicaset-advanced.yml           # Advanced selectors
├── deployment-basic.yml              # Basic deployment
├── deployment-rolling-update.yml     # Rolling update strategy
├── deployment-recreate.yml           # Recreate strategy
├── deployment-production.yml         # Full production setup
├── daemonset-basic.yml               # DaemonSet example
├── daemonset-with-nodeaffinity.yml   # Node-specific DaemonSet
├── job-basic.yml                     # Batch job
├── cronjob-basic.yml                 # Scheduled job
└── README.md                         # Folder guide
```

**Quick Apply**:
```bash
kubectl apply -f Examples/01-Controllers/deployment-basic.yml
```

#### Services (7 files)
```
Examples/02-Services/
├── service-clusterip.yml             # Internal only
├── service-nodeport.yml              # External on node ports
├── service-loadbalancer.yml          # Cloud LB integration
├── service-externalname.yml          # DNS alias
├── service-headless.yml              # Direct pod access
├── service-multiport.yml             # Multiple ports
├── service-with-sessionaffinity.yml  # Sticky sessions
└── README.md                         # Folder guide
```

**Quick Apply**:
```bash
kubectl apply -f Examples/02-Services/service-clusterip.yml
```

#### Probes (4 files)
```
Examples/03-Probes/
├── probe-http.yml                    # HTTP health checks
├── probe-tcp.yml                     # TCP connection checks
├── probe-exec.yml                    # Custom command checks
├── probe-all-types.yml               # All three probe types
└── README.md                         # Folder guide
```

**Quick Apply**:
```bash
kubectl apply -f Examples/03-Probes/probe-all-types.yml
```

#### Pods (5 files)
```
Examples/04-Pods/
├── pod-simple.yml                    # Single container
├── pod-simple-with-resources.yml     # With limits/requests
├── pod-multicontainer.yml            # Multiple containers
├── pod-sidecar.yml                   # Logging/monitoring sidecar
├── pod-initcontainer.yml             # Init container patterns
└── README.md                         # Folder guide
```

**Quick Apply**:
```bash
kubectl apply -f Examples/04-Pods/pod-simple.yml
```

#### Advanced (3 files)
```
Examples/05-Advanced/
├── taints-tolerations.yml            # Node taints & pod tolerations
├── lifecycle-hooks.yml               # preStop, postStart hooks
├── resource-quota.yml                # Quotas & limits
└── README.md                         # Folder guide
```

**Quick Apply**:
```bash
kubectl apply -f Examples/05-Advanced/taints-tolerations.yml
```

#### ConfigMaps & Secrets (2 files)
```
Examples/06-ConfigMaps-Secrets/
├── configmap-examples.yml            # Configuration data
├── secret-examples.yml               # Sensitive data
└── README.md                         # Folder guide
```

**Quick Apply**:
```bash
kubectl apply -f Examples/06-ConfigMaps-Secrets/configmap-examples.yml
```

## 🔍 Find Content By Topic

### Pod Topics
- Pod lifecycle → [03-Core-Concepts/README.md](03-Core-Concepts/README.md#pod-lifecycle)
- Pod phases → [03-Core-Concepts/README.md](03-Core-Concepts/README.md#pod-phases)
- Container states → [03-Core-Concepts/README.md](03-Core-Concepts/README.md#container-states)
- Pod restart policies → [03-Core-Concepts/README.md](03-Core-Concepts/README.md#pod-restart-policy)
- Pod examples → [Examples/04-Pods/](Examples/04-Pods/)
- Crash loop backoff → [03-Core-Concepts/README.md](03-Core-Concepts/README.md#crash-loop-backoff)

### Controller Topics
- Deployment → [04-Controllers/README.md](04-Controllers/README.md#deployment)
- DaemonSet → [04-Controllers/README.md](04-Controllers/README.md#daemonset)
- Job/CronJob → [04-Controllers/README.md](04-Controllers/README.md#job)
- ReplicaSet → [04-Controllers/README.md](04-Controllers/README.md#replicaset)
- Controller examples → [Examples/01-Controllers/](Examples/01-Controllers/)

### Service Topics
- Service types → [05-Services/README.md](05-Services/README.md#service-types)
- ClusterIP → [05-Services/README.md](05-Services/README.md#1-clusterip-default)
- NodePort → [05-Services/README.md](05-Services/README.md#2-nodeport)
- LoadBalancer → [05-Services/README.md](05-Services/README.md#3-loadbalancer)
- kube-proxy → [05-Services/README.md](05-Services/README.md#kube-proxy-and-service-networking)
- Service examples → [Examples/02-Services/](Examples/02-Services/)

### Probe Topics
- Startup probe → [06-Probes/README.md](06-Probes/README.md#1-startup-probe)
- Readiness probe → [06-Probes/README.md](06-Probes/README.md#2-readiness-probe)
- Liveness probe → [06-Probes/README.md](06-Probes/README.md#3-liveness-probe)
- HTTP probes → [Examples/03-Probes/probe-http.yml](Examples/03-Probes/probe-http.yml)
- TCP probes → [Examples/03-Probes/probe-tcp.yml](Examples/03-Probes/probe-tcp.yml)

### Advanced Topics
- Taints & tolerations → [07-Advanced/README.md](07-Advanced/README.md#taints-and-tolerations)
- Lifecycle hooks → [07-Advanced/README.md](07-Advanced/README.md#lifecycle-hooks)
- Init containers → [07-Advanced/README.md](07-Advanced/README.md#init-containers)
- Sidecars → [07-Advanced/README.md](07-Advanced/README.md#sidecar-containers)
- Examples → [Examples/05-Advanced/](Examples/05-Advanced/)

## 🔧 Quick Command Reference

### Most Common Commands
```bash
# View resources
kubectl get pods
kubectl get svc
kubectl get deployment

# Get details
kubectl describe pod <name>
kubectl describe svc <name>

# Apply YAML
kubectl apply -f filename.yml
kubectl apply -f Examples/

# Logs and debugging
kubectl logs <pod>
kubectl exec -it <pod> -- /bin/bash

# Port forward
kubectl port-forward svc/<service> 8080:80
```

**Full reference**: [08-Cheatsheet/README.md](08-Cheatsheet/README.md)

## 📋 Finding Examples by Use Case

### Web Application
```bash
# 1. Deploy
kubectl apply -f Examples/01-Controllers/deployment-rolling-update.yml

# 2. Expose
kubectl apply -f Examples/02-Services/service-loadbalancer.yml

# 3. Health checks
kubectl apply -f Examples/03-Probes/probe-all-types.yml
```

### Database
```bash
# 1. StatefulSet (in deployment file)
# 2. Headless service
kubectl apply -f Examples/02-Services/service-headless.yml

# 3. TCP probes
kubectl apply -f Examples/03-Probes/probe-tcp.yml

# 4. Credentials
kubectl apply -f Examples/06-ConfigMaps-Secrets/secret-examples.yml
```

### Logging/Monitoring
```bash
# DaemonSet for all nodes
kubectl apply -f Examples/01-Controllers/daemonset-basic.yml

# Sidecar approach
kubectl apply -f Examples/04-Pods/pod-sidecar.yml

# Multi-port for metrics
kubectl apply -f Examples/02-Services/service-multiport.yml
```

### Batch Jobs
```bash
# One-time job
kubectl apply -f Examples/01-Controllers/job-basic.yml

# Scheduled cron job
kubectl apply -f Examples/01-Controllers/cronjob-basic.yml
```

## 🌟 Top 10 Most Important Files

1. **[README.md](README.md)** - Start here
2. **[Examples/README.md](Examples/README.md)** - Practical YAML guide
3. **[03-Core-Concepts/README.md](03-Core-Concepts/README.md)** - Pod fundamentals
4. **[04-Controllers/README.md](04-Controllers/README.md)** - Deployment guide
5. **[05-Services/README.md](05-Services/README.md)** - Networking essentials
6. **[06-Probes/README.md](06-Probes/README.md)** - Health checks guide
7. **[Examples/01-Controllers/deployment-production.yml](Examples/01-Controllers/deployment-production.yml)** - Production template
8. **[08-Cheatsheet/README.md](08-Cheatsheet/README.md)** - Command reference
9. **[07-Advanced/README.md](07-Advanced/README.md)** - Advanced patterns
10. **[Examples/05-Advanced/lifecycle-hooks.yml](Examples/05-Advanced/lifecycle-hooks.yml)** - Graceful shutdown

## 🎯 Recommended Learning Order

### Day 1: Basics
1. [README.md](README.md) - Overview
2. [01-Introduction/README.md](01-Introduction/README.md) - What is K8s
3. [02-Installation/README.md](02-Installation/README.md) - Setup cluster

### Day 2-3: Core Concepts
1. [03-Core-Concepts/README.md](03-Core-Concepts/README.md) - Pods
2. [Examples/04-Pods/](Examples/04-Pods/) - Pod examples

### Day 4-5: Deployments
1. [04-Controllers/README.md](04-Controllers/README.md) - Controllers
2. [Examples/01-Controllers/](Examples/01-Controllers/) - Apply examples

### Day 6-7: Networking
1. [05-Services/README.md](05-Services/README.md) - Services
2. [Examples/02-Services/](Examples/02-Services/) - Service examples

### Day 8-9: Advanced
1. [06-Probes/README.md](06-Probes/README.md) - Health checks
2. [07-Advanced/README.md](07-Advanced/README.md) - Advanced topics

### Day 10: Reference
1. [08-Cheatsheet/README.md](08-Cheatsheet/README.md) - Commands
2. [Examples/](Examples/) - Browse all examples

## 📞 Getting Help

### Looking for specific information?
1. Check table of contents in each README.md
2. Use [Examples/README.md](Examples/README.md) for YAML
3. Search [08-Cheatsheet/README.md](08-Cheatsheet/README.md) for commands
4. Read relevant topic folder for deep explanation

### Want to try something?
1. Go to [Examples/](Examples/) folder
2. Find relevant .yml file
3. Apply with: `kubectl apply -f filename.yml`
4. View results: `kubectl get ...`

---

**Last Updated**: 2026-08-20
**Repository Version**: 1.0 Complete
