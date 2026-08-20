# Controllers - YAML Examples

This directory contains practical YAML examples for Kubernetes controllers.

## Files Overview

### ReplicaSet Examples
- `replicaset-basic.yml` - Basic ReplicaSet with equality selectors
- `replicaset-advanced.yml` - Advanced ReplicaSet with set-based selectors

### Deployment Examples
- `deployment-basic.yml` - Basic deployment with 3 replicas
- `deployment-rolling-update.yml` - Deployment with rolling update strategy
- `deployment-recreate.yml` - Deployment with recreate strategy
- `deployment-production.yml` - Production-ready deployment with probes and resources

### DaemonSet Examples
- `daemonset-basic.yml` - Basic DaemonSet example
- `daemonset-with-nodeaffinity.yml` - DaemonSet with node affinity

### Job & CronJob Examples
- `job-basic.yml` - Basic batch job
- `cronjob-basic.yml` - Scheduled CronJob

## Quick Start

```bash
# Apply a ReplicaSet
kubectl apply -f replicaset-basic.yml

# Apply a Deployment
kubectl apply -f deployment-basic.yml

# View resources
kubectl get rs
kubectl get deployments
kubectl get daemonsets

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5

# Check rollout status
kubectl rollout status deployment nginx-deployment
```

## Resource Relationships

```
┌─────────────────────────────────┐
│        Deployment               │
│  (manages rollouts/rollbacks)   │
└──────────────┬──────────────────┘
               │
               │ Creates/Updates
               ↓
         ┌──────────────┐
         │ ReplicaSet   │
         │ (maintains   │
         │ pod count)   │
         └──────┬───────┘
                │
                │ Creates
                ↓
        ┌───────────────┐
        │     Pods      │
        │ (containers)  │
        └───────────────┘
```

See individual YAML files for detailed examples and comments.
