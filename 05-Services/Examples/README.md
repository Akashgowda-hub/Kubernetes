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

### Ingress and TLS
- `ingress-basic.yml` - Basic host-based ingress
- `ingress-path-based-routing.yml` - Path-based routing (`/api` and `/`)
- `ingress-header-based-routing.yml` - Header-based routing (NGINX canary header)
- `ingress-tls-selfsigned.yml` - TLS termination with self-signed cert secret

### Detailed Ingress Learning Structure
- `../Ingress/README.md` - Ingress fundamentals, controller choice, cost discussion, install steps
- `../Ingress/Path-Based/` - Path-based routing notes and examples
- `../Ingress/Header-Based/` - Header-based routing notes and examples
- `../Ingress/TLS-Self-Signed/` - TLS termination and self-signed step-by-step

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

# Install NGINX ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# Apply ingress examples
kubectl apply -f ingress-basic.yml
kubectl apply -f ingress-path-based-routing.yml

# View services
kubectl get svc
kubectl get svc -o wide

# View ingress resources
kubectl get ingress
kubectl describe ingress app-path-routing

# Get service endpoints
kubectl get endpoints

# Test service connectivity
kubectl run -it --rm debug --image=busybox -- sh
# Inside pod: wget service-name:port

# Test ingress by host header
curl -H "Host: app.k8s.local" http://<INGRESS_IP>/

# Test header-based routing to canary
curl -H "Host: app.k8s.local" -H "x-canary: always" http://<INGRESS_IP>/
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

For full ingress deep-dive, use the dedicated folder tree under `05-Services/Ingress/`.
