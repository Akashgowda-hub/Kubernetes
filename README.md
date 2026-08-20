# Kubernetes Learning Repository

A comprehensive guide to Kubernetes (k8s) covering installation, core concepts, controllers, services, and advanced topics.

## 📚 Table of Contents

### 📖 Documentation
1. [Introduction to Kubernetes](01-Introduction/README.md)
2. [Installation Guide](02-Installation/README.md)
3. [Core Concepts](03-Core-Concepts/README.md)
4. [Controllers](04-Controllers/README.md)
5. [Services & Networking](05-Services/README.md)
6. [Health Probes](06-Probes/README.md)
7. [Advanced Topics](07-Advanced/README.md)
8. [Cheatsheet & Reference](08-Cheatsheet/README.md)

### 💻 Live Examples & YAML Files
**[Examples/](Examples/README.md)** - Production-ready YAML files organized by topic
- [Controllers](Examples/01-Controllers/) - Deployments, ReplicaSets, DaemonSets, Jobs
- [Services](Examples/02-Services/) - ClusterIP, NodePort, LoadBalancer, Headless
- [Probes](Examples/03-Probes/) - Health checks (Startup, Readiness, Liveness)
- [Pods](Examples/04-Pods/) - Pod patterns, multi-container, sidecars
- [Advanced](Examples/05-Advanced/) - Taints, Affinity, Lifecycle, Quotas
- [ConfigMaps & Secrets](Examples/06-ConfigMaps-Secrets/) - Configuration management

## 🎯 What You'll Learn

- **Installation**: kubeadm setup with containerd and Calico CNI plugin
- **API Server**: Client-API server communication and API resources
- **Kubectl**: Command-line usage (declarative and imperative methods)
- **Pods**: Lifecycle, phases, container states, and restart policies
- **Controllers**: Replication, Deployments, DaemonSets, and more
- **Services**: ClusterIP, NodePort, LoadBalancer, ExternalName
- **Networking**: Network-level changes, kubeproxy modes
- **Events & Monitoring**: Event management, resource quotas, limits
- **Advanced Concepts**: Taints, tolerations, lifecycle hooks, and more

## 📖 How to Use This Repository

### Learning Path
1. Start with [Introduction](01-Introduction/README.md) for basic concepts
2. Follow [Installation Guide](02-Installation/README.md) to set up your cluster
3. Explore [Core Concepts](03-Core-Concepts/README.md) to understand Pods and Namespaces
4. Study [Controllers](04-Controllers/README.md) for workload management
5. Review [Services](05-Services/README.md) for networking
6. Check [Health Probes](06-Probes/README.md) for application health management
7. Learn [Advanced Topics](07-Advanced/README.md) for production patterns
8. Reference [Cheatsheet](08-Cheatsheet/README.md) for quick command lookups

### Hands-On Practice
- **Start with Examples**: Browse [Examples/README.md](Examples/README.md) for an overview
- **Follow by Topic**: Each subfolder contains:
  - `README.md` - Explanations and use cases
  - `*.yml` - Production-ready YAML files ready to apply
- **Apply Examples**:
  ```bash
  # Apply specific example
  kubectl apply -f Examples/01-Controllers/deployment-basic.yml
  
  # Apply all examples in a folder
  kubectl apply -f Examples/02-Services/
  
  # Validate before applying
  kubectl apply -f Examples/deployment.yml --dry-run=client
  ```
- **Test & Learn**:
  ```bash
  # View pod logs
  kubectl logs <pod-name>
  
  # Describe resources
  kubectl describe deployment <deployment-name>
  
  # Port forward to test
  kubectl port-forward svc/<service-name> 8080:80
  ```

---

**Last Updated**: 2026-08-20
