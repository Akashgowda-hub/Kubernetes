# Introduction to Kubernetes

## What is Kubernetes?

Kubernetes (k8s) is an open-source container orchestration platform that automates many of the manual processes involved in deploying, managing, and scaling containerized applications.

## Key Concepts

### Container Orchestration
- Automated management of containerized applications across clusters
- Handles deployment, scheduling, and resource management
- Ensures high availability and scalability

### Master-Node Architecture
- **Control Plane (Master)**: Manages cluster state and makes decisions
- **Nodes (Workers)**: Run containerized applications

### Main Components

#### Control Plane Components
- **API Server**: Central management point for the cluster
- **etcd**: Distributed key-value store for cluster data
- **Scheduler**: Assigns pods to nodes
- **Controller Manager**: Runs controller processes
- **Cloud Controller Manager**: Interacts with cloud providers

#### Node Components
- **kubelet**: Agent running on each node
- **Container Runtime**: Runs containers (Docker, containerd, CRI-O)
- **kube-proxy**: Network proxy for service communication

## Why Use Kubernetes?

1. **Scalability**: Easily scale applications up or down
2. **High Availability**: Automatic recovery and failover
3. **Resource Efficiency**: Optimal resource utilization
4. **Declarative Configuration**: Define desired state
5. **Self-Healing**: Automatic restart of failed containers
6. **Rolling Updates**: Zero-downtime deployments
7. **Service Discovery**: Automatic DNS-based service discovery

## Kubernetes Objects

### Basic Objects
- **Pod**: Smallest deployable unit
- **Service**: Network abstraction for pod access
- **Namespace**: Virtual cluster for resource isolation
- **ConfigMap**: Configuration management
- **Secret**: Sensitive data management
- **Persistent Volume**: Storage abstraction

### Higher-Level Objects (Controllers)
- **Deployment**: Manages ReplicaSets and pods
- **ReplicaSet**: Ensures desired number of pod replicas
- **StatefulSet**: Manages stateful applications
- **DaemonSet**: Runs pod on every node
- **Job**: Runs tasks to completion
- **CronJob**: Scheduled task execution

## Cluster Architecture

```
                         ┌──────────────────────────────┐
                         │          USERS / CLIENTS     │
                         │  kubectl / Applications      │
                         └──────────────┬───────────────┘
                                        │
                                        │ HTTPS / API requests
                                        ▼
                 ┌──────────────────────────────────────────┐
                 │             CONTROL PLANE                │
                 │                                          │
                 │  ┌───────────────┐   ┌───────────────┐  │
                 │  │  API Server   │◄─►│     etcd      │  │
                 │  └───────┬───────┘   └───────────────┘  │
                 │          │                               │
                 │          │                               │
                 │  ┌───────▼───────┐   ┌───────────────┐  │
                 │  │   Scheduler   │   │ Controller    │  │
                 │  │               │   │  Manager      │  │
                 │  └───────────────┘   └───────────────┘  │
                 └──────────────┬───────────────────────────┘
                                │
                     Assigns / manages workloads
                                │
              ┌─────────────────┼──────────────────┐
              │                 │                  │
              ▼                 ▼                  ▼
     ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
     │   WORKER NODE 1│ │   WORKER NODE 2│ │   WORKER NODE 3│
     │                │ │                │ │                │
     │ ┌────────────┐ │ │ ┌────────────┐ │ │ ┌────────────┐ │
     │ │   kubelet  │ │ │ │   kubelet  │ │ │ │   kubelet  │ │
     │ └────────────┘ │ │ └────────────┘ │ │ └────────────┘ │
     │                │ │                │ │                │
     │ ┌────────────┐ │ │ ┌────────────┐ │ │ ┌────────────┐ │
     │ │ kube-proxy │ │ │ │ kube-proxy │ │ │ │ kube-proxy │ │
     │ └────────────┘ │ │ └────────────┘ │ │ └────────────┘ │
     │                │ │                │ │                │
     │ ┌──────┐┌─────┐│ │ ┌──────┐┌─────┐│ │ ┌──────┐┌─────┐│
     │ │ Pod A││Pod B││ │ │ Pod C││Pod D││ │ │ Pod E││Pod F││
     │ └──────┘└─────┘│ │ └──────┘└─────┘│ │ └──────┘└─────┘│
     │                │ │                │ │                │
     │ Container      │ │ Container      │ │ Container      │
     │ Runtime        │ │ Runtime        │ │ Runtime        │
     └────────────────┘ └────────────────┘ └────────────────┘

```

## Next Steps

- [Installation Guide](../02-Installation/README.md)
- [Core Concepts](../03-Core-Concepts/README.md)
