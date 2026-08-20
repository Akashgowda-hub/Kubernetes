# Advanced Topics - YAML Examples

This directory contains practical YAML examples for advanced Kubernetes concepts.

## Files Overview

### Taints and Tolerations
- `taints-tolerations.yml` - Node taints and pod tolerations
- `dedicated-nodes.yml` - Dedicated node pools for specific workloads

### Lifecycle and Affinity
- `lifecycle-hooks.yml` - Pod lifecycle hooks (preStop, postStart)
- `node-affinity.yml` - Node affinity rules
- `pod-affinity.yml` - Pod-to-pod affinity and anti-affinity

### Resource Management
- `resource-quota.yml` - Namespace resource quotas
- `limitrange.yml` - Per-container and per-pod limits

### Scheduling and Disruption
- `pod-disruption-budget.yml` - Pod Disruption Budget

## Scheduling Decision Flowchart

```
Pod Creation Request
    ↓
kube-scheduler evaluates
    ├─ Node Affinity
    │  ├─ requiredDuring... (must have)
    │  └─ preferredDuring... (should have)
    ├─ Pod Affinity
    │  ├─ Pod should be near X pods
    │  └─ Pod should be away from X pods
    ├─ Taints & Tolerations
    │  └─ Node repels pods without toleration
    ├─ nodeSelector
    │  └─ Simple label matching
    └─ Resource availability
       └─ Node has enough CPU/Memory
    ↓
Selected Node
    ↓
Pod Scheduled
    ↓
kubelet starts containers
```

## Taints vs Node Selector

```
Node Selector (Pod → Node)
"I want to run on nodes with gpu=true"

         Pod with
         nodeSelector:
         gpu: true
              ↓
              │ Attracts to
              ↓
    ┌─────────────────────┐
    │ Node with gpu=true  │
    └─────────────────────┘

─────────────────────────────────────

Taints/Tolerations (Node → Pod)
"I repel pods unless they tolerate my taint"

    ┌─────────────────────┐
    │ Node tainted:       │
    │ gpu=true:NoSchedule │
    └──────────┬──────────┘
               │ Repels pods
               │
         Pod without
         toleration
         (REJECTED)
         
         Pod WITH toleration
         gpu=true:NoSchedule
         (ACCEPTED)
```

## Quick Commands

```bash
# Apply examples
kubectl apply -f taints-tolerations.yml
kubectl apply -f node-affinity.yml

# View taints on nodes
kubectl describe node <node-name> | grep Taint

# Add taint to node
kubectl taint nodes <node-name> key=value:NoSchedule

# View pod affinity
kubectl describe pod <pod-name> | grep -A 5 Affinity

# View resource quotas
kubectl describe resourcequota -n <namespace>
```

See individual YAML files for detailed examples and configurations.
