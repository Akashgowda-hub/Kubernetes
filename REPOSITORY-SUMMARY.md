# Repository Summary

Current summary of this Kubernetes learning repository after latest structure updates.

## Repository Layout

```
Kubernetes/
├── README.md
├── QUICK-NAVIGATION.md
├── REPOSITORY-SUMMARY.md
├── 01-Introduction/
│   └── README.md
├── 02-Installation/
│   └── README.md
├── 03-Core-Concepts/
│   ├── README.md
│   ├── Examples/
│   └── ConfigMaps-Secrets-Examples/
├── 04-Controllers/
│   ├── README.md
│   └── Examples/
├── 05-Services/
│   ├── README.md
│   ├── Examples/
│   └── Ingress/
│       ├── README.md
│       ├── Path-Based/
│       │   ├── README.md
│       │   └── Examples/
│       ├── Header-Based/
│       │   ├── README.md
│       │   └── Examples/
│       └── TLS-Self-Signed/
│           ├── README.md
│           └── Examples/
├── 06-Probes/
│   ├── README.md
│   └── Examples/
├── 07-Advanced/
│   ├── README.md
│   └── Examples/
└── 08-Cheatsheet/
    ├── README.md
    └── YAML-EXAMPLES.md
```

## Topic Coverage

- Introduction and cluster architecture
- kubeadm installation with containerd and Calico
- Pods, phases, states, restart behavior, events
- Controllers (ReplicaSet, Deployment, DaemonSet, Job, CronJob)
- Services and networking (ClusterIP, NodePort, LoadBalancer, Headless)
- Ingress deep dive (controller setup, path routing, header routing, TLS)
- Probes (startup, readiness, liveness)
- Advanced operations (taints, lifecycle hooks, quotas)
- Command cheatsheet

## Example Locations (Current)

- Pods: [03-Core-Concepts/Examples](03-Core-Concepts/Examples)
- ConfigMaps and Secrets: [03-Core-Concepts/ConfigMaps-Secrets-Examples](03-Core-Concepts/ConfigMaps-Secrets-Examples)
- Controllers: [04-Controllers/Examples](04-Controllers/Examples)
- Services: [05-Services/Examples](05-Services/Examples)
- Ingress Path: [05-Services/Ingress/Path-Based/Examples](05-Services/Ingress/Path-Based/Examples)
- Ingress Header: [05-Services/Ingress/Header-Based/Examples](05-Services/Ingress/Header-Based/Examples)
- Ingress TLS: [05-Services/Ingress/TLS-Self-Signed/Examples](05-Services/Ingress/TLS-Self-Signed/Examples)
- Probes: [06-Probes/Examples](06-Probes/Examples)
- Advanced: [07-Advanced/Examples](07-Advanced/Examples)

## Recommended Entry Points

1. [README.md](README.md)
2. [05-Services/Ingress/README.md](05-Services/Ingress/README.md)
3. [05-Services/Ingress/TLS-Self-Signed/README.md](05-Services/Ingress/TLS-Self-Signed/README.md)
4. [QUICK-NAVIGATION.md](QUICK-NAVIGATION.md)

---

Last Updated: 2026-08-24
