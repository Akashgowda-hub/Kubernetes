# Ingress in Kubernetes

Ingress provides Layer-7 (HTTP/HTTPS) routing to multiple services using one external entry point.

## Why we use Ingress

If every app uses a separate LoadBalancer service, cost and operations grow quickly:

- One app -> one LoadBalancer
- Ten apps -> ten LoadBalancers
- More DNS records, certs, firewall rules, and monitoring overhead

Ingress pattern reduces this:

- One Ingress Controller (often one external LB)
- Multiple apps behind host/path/header rules
- Centralized TLS termination and traffic policy

## High-level architecture

```
┌───────────────────────────┐
│       Internet Users      │
└─────────────┬─────────────┘
              │ HTTPS/HTTP
┌─────────────▼─────────────┐
│   External Load Balancer  │  (optional: NodePort in bare-metal)
└─────────────┬─────────────┘
              │
┌─────────────▼─────────────┐
│  Ingress Controller Pods  │
│ (nginx / traefik / haproxy)
└─────────────┬─────────────┘
              │ reads ingress rules
┌─────────────▼─────────────┐
│      Ingress Objects      │
└─────────────┬─────────────┘
              │ routes to service
┌─────────────▼─────────────┐
│         Services          │
└─────────────┬─────────────┘
              │
┌─────────────▼─────────────┐
│            Pods           │
└───────────────────────────┘
```

## Controllers and support lifecycle

Common controllers:

- ingress-nginx (community)
- NGINX Ingress Controller by F5/NGINX (commercial/community variants)
- Traefik
- HAProxy Ingress
- Cloud-specific controllers (AWS ALB, GCE, Azure)

Important lifecycle note:

- Support/EOL dates are controller and version specific.
- If a controller version reaches end-of-support, plan migration to a supported version/controller.
- Always check official release notes for your controller build used in company environments.

## Recommended starting point

For general Kubernetes learning, ingress-nginx is usually the easiest to start with.

## Steps to configure ingress-nginx controller

### 1) Install ingress-nginx

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

### 2) Verify components

```bash
kubectl get ns ingress-nginx
kubectl get deploy -n ingress-nginx
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

### 3) Confirm ingress class

```bash
kubectl get ingressclass
```

Expected class for examples in this repo: `nginx`

### 4) Create/test ingress resources

- Path-based examples: `Path-Based/`
- Header-based examples: `Header-Based/`
- TLS examples: `TLS-Self-Signed/`

## Cost discussion: LB per app vs one ingress entry

```
Without Ingress
┌───────┐   ┌───────┐   ┌───────┐
│ App A │   │ App B │   │ App C │
└───┬───┘   └───┬───┘   └───┬───┘
    │           │           │
┌───▼───┐   ┌───▼───┐   ┌───▼───┐
│  LB   │   │  LB   │   │  LB   │
└───────┘   └───────┘   └───────┘

With Ingress
┌───────┐ ┌───────┐ ┌───────┐
│ App A │ │ App B │ │ App C │
└───┬───┘ └───┬───┘ └───┬───┘
    └────┬────┴────┬────┘
         │ Ingress │
      ┌──▼─────────▼──┐
      │ 1 LB + rules  │
      └───────────────┘
```
