# Services - YAML Examples

This directory contains practical YAML examples for Kubernetes Services.

## Files Overview

### Service Types
- `service-clusterip.yml` - ClusterIP service (internal access only)
- `service-nodeport.yml` - NodePort service (expose on node ports)
- `service-loadbalancer.yml` - LoadBalancer service (external load balancer)
- `service-externalname.yml` - ExternalName service (DNS alias)
- `service-headless.yml` - Headless service (no ClusterIP)

### Advanced Services
- `service-multiport.yml` - Service with multiple ports
- `service-with-sessionaffinity.yml` - Service with session affinity

## Service Architecture Overview

```
External Request
    ↓
┌───────────────────────────────┐
│  External LB (if type: LB)    │
└───────────────┬───────────────┘
                ↓
    ┌───────────────────────────┐
    │  Node Port (30000-32767)  │
    └───────────────┬───────────┘
                    ↓
        ┌──────────────────────┐
        │  kube-proxy          │
        │ (iptables/IPVS)      │
        └──────────┬───────────┘
                   ↓
        ┌──────────────────────┐
        │   Service (ClusterIP)│
        │  10.96.x.x:port      │
        └──────────┬───────────┘
                   ↓
        ┌──────────────────────┐
        │  Endpoint/Slice      │
        │ Pod IPs: 10.244.x.x  │
        └──────────┬───────────┘
                   ↓
        ┌──────────────────────┐
        │  Pods (Containers)   │
        └──────────────────────┘
```

## Quick Start

```bash
# Apply a ClusterIP service
kubectl apply -f service-clusterip.yml

# Apply a NodePort service
kubectl apply -f service-nodeport.yml

# View services
kubectl get svc
kubectl get svc -o wide

# Get service endpoints
kubectl get endpoints

# Test service connectivity
kubectl run -it --rm debug --image=busybox -- sh
# Inside pod: wget service-name:port
```

## Service Selection and Routing

```
┌─────────────────────────────────────┐
│       Service Selector              │
│   app: nginx, tier: frontend        │
└──────────────┬──────────────────────┘
               │ Matches labels
               ↓
     ┌─────────────────────────┐
     │ Pod 1 (10.244.1.10)     │
     │ app: nginx              │
     │ tier: frontend          │
     └─────────────────────────┘
     
     ┌─────────────────────────┐
     │ Pod 2 (10.244.2.20)     │
     │ app: nginx              │
     │ tier: frontend          │
     └─────────────────────────┘
     
     ┌─────────────────────────┐
     │ Pod 3 (10.244.3.30)     │
     │ app: nginx              │
     │ tier: frontend          │
     └─────────────────────────┘
```

See individual YAML files for detailed examples and configurations.
