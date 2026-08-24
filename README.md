# Kubernetes Learning Repository

A practical Kubernetes learning repository with topic-wise guides and ready-to-apply YAML examples.

## Documentation Map

1. [Introduction](01-Introduction/README.md)
2. [Installation](02-Installation/README.md)
3. [Core Concepts](03-Core-Concepts/README.md)
4. [Controllers](04-Controllers/README.md)
5. [Services and Networking](05-Services/README.md)
6. [Ingress Deep Dive](05-Services/Ingress/README.md)
7. [Health Probes](06-Probes/README.md)
8. [Advanced Topics](07-Advanced/README.md)
9. [Cheatsheet](08-Cheatsheet/README.md)

## YAML Examples (Current Structure)

Examples are organized inside each topic folder.

- Controllers: [04-Controllers/Examples/README.md](04-Controllers/Examples/README.md)
- Pods: [03-Core-Concepts/Examples/README.md](03-Core-Concepts/Examples/README.md)
- ConfigMaps and Secrets: [03-Core-Concepts/ConfigMaps-Secrets-Examples/README.md](03-Core-Concepts/ConfigMaps-Secrets-Examples/README.md)
- Services: [05-Services/Examples/README.md](05-Services/Examples/README.md)
- Probes: [06-Probes/Examples/README.md](06-Probes/Examples/README.md)
- Advanced: [07-Advanced/Examples/README.md](07-Advanced/Examples/README.md)

Ingress examples are grouped separately for clarity:

- Ingress overview: [05-Services/Ingress/README.md](05-Services/Ingress/README.md)
- Path-based: [05-Services/Ingress/Path-Based/README.md](05-Services/Ingress/Path-Based/README.md)
- Header-based: [05-Services/Ingress/Header-Based/README.md](05-Services/Ingress/Header-Based/README.md)
- TLS self-signed: [05-Services/Ingress/TLS-Self-Signed/README.md](05-Services/Ingress/TLS-Self-Signed/README.md)

## What You Will Learn

- kubeadm installation with containerd and Calico
- Pod lifecycle, container states, restart behavior
- Deployments, ReplicaSets, DaemonSets, Jobs and CronJobs
- Service types, kube-proxy, endpoint flow
- Ingress concepts, controller setup, path/header routing
- TLS termination with self-signed certs and Kubernetes TLS Secret
- Probes, taints/tolerations, lifecycle hooks, quotas

## Recommended Study Order

1. [01-Introduction/README.md](01-Introduction/README.md)
2. [02-Installation/README.md](02-Installation/README.md)
3. [03-Core-Concepts/README.md](03-Core-Concepts/README.md)
4. [04-Controllers/README.md](04-Controllers/README.md)
5. [05-Services/README.md](05-Services/README.md)
6. [05-Services/Ingress/README.md](05-Services/Ingress/README.md)
7. [06-Probes/README.md](06-Probes/README.md)
8. [07-Advanced/README.md](07-Advanced/README.md)
9. [08-Cheatsheet/README.md](08-Cheatsheet/README.md)

## Quick Start

```bash
# Example: deploy a controller workload
kubectl apply -f 04-Controllers/Examples/deployment-basic.yml

# Example: expose via service
kubectl apply -f 05-Services/Examples/service-clusterip.yml

# Example: create ingress path rules
kubectl apply -f 05-Services/Ingress/Path-Based/Examples/path-routing-basic.yml
```

## Navigation Docs

- Quick navigation: [QUICK-NAVIGATION.md](QUICK-NAVIGATION.md)
- Repository summary: [REPOSITORY-SUMMARY.md](REPOSITORY-SUMMARY.md)

---

Last Updated: 2026-08-24
