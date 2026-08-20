# Pods - YAML Examples

This directory contains practical YAML examples for Kubernetes Pods.

## Files Overview

### Basic Pods
- `pod-simple.yml` - Simple single-container pod
- `pod-simple-with-resources.yml` - Pod with resource requests and limits

### Multi-Container Pods
- `pod-multicontainer.yml` - Pod with multiple containers
- `pod-sidecar.yml` - Pod with logging sidecar container

### Advanced Pod Patterns
- `pod-initcontainer.yml` - Pod with init containers
- `pod-lifecycle-hooks.yml` - Pod with lifecycle hooks (preStop, postStart)
- `pod-with-volumes.yml` - Pod with various volume types

## Pod Architecture Diagram

```
┌──────────────────────────────────────┐
│         Pod (Single Node)            │
├──────────────────────────────────────┤
│  Shared Namespace:                   │
│  ├─ Network (eth0)                   │
│  │  └─ Single IP: 10.244.1.10        │
│  ├─ IPC                              │
│  ├─ UTS (hostname)                   │
│  └─ Volumes (shared storage)         │
├──────────────────────────────────────┤
│                                      │
│  ┌──────────────┐  ┌──────────────┐ │
│  │ Container 1  │  │ Container 2  │ │
│  │ (app)        │  │ (sidecar)    │ │
│  │ Ports:       │  │              │ │
│  │ - :8080      │  │ - :3000      │ │
│  └──────────────┘  └──────────────┘ │
│                                      │
│  ┌──────────────────────────────┐   │
│  │ Volumes (shared between)     │   │
│  │ - logs                       │   │
│  │ - config                     │   │
│  │ - secrets                    │   │
│  └──────────────────────────────┘   │
└──────────────────────────────────────┘
```

## Pod Lifecycle Flowchart

```
Create Pod Request
    ↓
Pending Phase
├─ kube-scheduler assigns to node
├─ Containers: Pulling images, creating
    ↓
Init Containers Run (if defined)
├─ Must complete successfully
├─ Run sequentially
├─ Each must exit with code 0
    ↓
Main Containers Start
├─ Startup Probe (if defined)
├─ Waits for application initialization
├─ Readiness/Liveness enabled after startup succeeds
    ↓
Running Phase
├─ Readiness Probe checks periodically
├─ Liveness Probe checks periodically
├─ Ready to handle traffic if readiness passes
    ↓
Pod Termination
├─ Receive SIGTERM
├─ preStop Hook runs (if defined)
├─ Grace period: 30 seconds
├─ If not terminated: SIGKILL sent
    ↓
Succeeded/Failed Phase
    ↓
Pod Deleted (garbage collected)
```

## Quick Start

```bash
# Apply a simple pod
kubectl apply -f pod-simple.yml

# View pod
kubectl get pods
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>

# Execute command in pod
kubectl exec -it <pod-name> -- /bin/bash

# Port forward to pod
kubectl port-forward pod/<pod-name> 8080:8080

# Delete pod
kubectl delete pod <pod-name>
```

See individual YAML files for detailed examples and comments.
