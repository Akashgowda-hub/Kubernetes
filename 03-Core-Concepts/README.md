# Core Kubernetes Concepts

## Pods

### Introduction to Pods

A Pod is the smallest and most basic deployable unit in Kubernetes. It's a wrapper around one or more containers that run together on the same node.

#### Key Characteristics
- **Smallest Unit**: Atomic unit of deployment in Kubernetes
- **Single IP Address**: All containers in a pod share the same IP
- **Shared Storage**: Can mount shared volumes
- **Shared Network Namespace**: Containers communicate via localhost
- **Tight Coupling**: Containers in a pod are tightly coupled

### Pod Lifecycle

#### Pod Phases

1. **Pending**: Pod accepted but containers not yet running
   - Waiting for scheduler
   - Pulling container image
   - Starting containers

2. **Running**: Pod bound to node and at least one container running
   - Ready to serve requests
   - Containers may be initializing

3. **Succeeded**: All containers in pod terminated successfully
   - Exit code 0
   - Pod completed its work

4. **Failed**: One or more containers terminated unsuccessfully
   - At least one container exit code ≠ 0
   - Pod did not complete successfully

5. **Unknown**: Pod state cannot be determined
   - Communication lost with node

### Container States

#### Waiting
- Container not yet running
- Reasons:
  - Image pulling
  - Applying secret
  - Waiting on temporary storage
  - Waiting on PersistentVolumeClaim

#### Running
- Container started successfully
- Status checks:
  - Application process running
  - Ready to handle requests

#### Terminated
- Container execution ended
- Reasons for termination:
  - Normal completion (exit code 0)
  - Application crash (exit code ≠ 0)
  - Container killed by signal
  - Resource limits exceeded (OOM killed)

### Pod Restart Policy

Restart policies control what happens when a container terminates.

#### Restart Policy Types

1. **Always** (default)
   - Container always restarts when it terminates
   - Infinite retry with exponential backoff
   - **Use Case**: Web servers, APIs, long-running services

2. **OnFailure**
   - Container only restarts if exit code ≠ 0
   - Does not restart on successful completion
   - **Use Case**: Batch jobs, one-time tasks

3. **Never**
   - Container never restarts
   - Parent controller responsible for new pod creation
   - **Use Case**: One-time jobs, debugging pods

#### Restart Policy Decisions

| Use Case | Restart Policy |
|----------|----------------|
| Web Applications | Always |
| REST APIs | Always |
| Batch Jobs | OnFailure |
| Cron Jobs | OnFailure |
| Databases | Never |
| One-time Scripts | Never |

### Exit Code Reference

| Exit Code | Description |
|-----------|-------------|
| 0 | Successful exit |
| 1 | General error / Application crash |
| 2 | Misuse of shell builtins |
| 126 | Invoked command cannot execute |
| 127 | Command not found |
| 128 | Exit using invalid signal |
| 128+2 | SIGINT (Ctrl+C) |
| 128+9 | SIGKILL (Forceful termination) |
| 128+15 | SIGTERM (Graceful termination) |
| 137 | Pod killed (usually due to resource limits) |
| 139 | Segmentation fault |
| 143 | Pod terminated by SIGTERM |
| 255 | Exit status out of range |

### Crash Loop Backoff

When a container keeps crashing and restarting, Kubernetes implements exponential backoff to avoid resource wastage.

#### Behavior

```
Attempt 1: 10 seconds wait
Attempt 2: 20 seconds wait
Attempt 3: 40 seconds wait
Attempt 4: 80 seconds wait
Attempt 5: 160 seconds wait
... (continues with exponential backoff)
Max backoff: 5 minutes (300 seconds)
```

#### Pod Status During Crash Loop
```
Status: Failed
Container State: Waiting (CrashLoopBackOff)
Reason: Back-off restarting failed container
```

#### Debugging Crash Loop BackOff

```bash
# Check pod status
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>

# View previous container logs (before crash)
kubectl logs <pod-name> --previous

# Check events
kubectl get events
kubectl describe pod <pod-name> | grep -A 5 Events
```

## Pod Termination

### Termination Grace Period

Default termination grace period: **30 seconds**

### Termination Process

1. **Pod received delete request**
   - Pod status: Terminating
   - Grace period countdown begins

2. **SIGTERM signal sent** (first 30 seconds)
   - Application should gracefully shutdown
   - Close connections
   - Perform cleanup

3. **Graceful shutdown period**
   - Application has up to 30 seconds to shutdown
   - If shutdown before grace period, pod terminates immediately

4. **SIGKILL signal** (after grace period expires)
   - Forceful termination
   - Process killed immediately
   - No cleanup possible

### Termination at Network Level

When a pod is deleted:

1. **Pod object removed from cluster**
2. **Endpoint & EndpointSlices updated**
   - Pod IP removed from service endpoints
3. **kube-proxy updates iptables/IPVS rules**
   - Traffic stops routing to pod
4. **Connection cleanup**
   - Existing connections closed
   - New connections refused

### Node Failure vs Pod Crash

#### Container Crash
- **Pod Name**: Unchanged
- **Pod IP**: Unchanged (until pod is rescheduled)
- **Action**: kubelet restarts container

#### Pod Deletion
- **Pod Name**: Changed (new pod gets new name)
- **Pod IP**: Changed (new IP assigned)
- **Action**: Scheduler creates new pod

#### Node Failure
- **Pod Status**: Unknown (pod is orphaned)
- **Timeout**: 5 minutes (default node-monitor-grace-period)
- **Action**: Control plane evicts pod and creates replacement on healthy node

## Labels and Selectors

### What are Labels?

Labels are key-value pairs attached to Kubernetes objects for organization and selection.

#### Characteristics
- Non-hierarchical
- Can be added/modified anytime
- Used for organizing and selecting subsets of objects
- Multiple labels per object allowed

#### Label Conventions
```yaml
app: nginx
version: v1.0
env: production
tier: frontend
team: backend-team
```

#### Naming Rules
- Max 63 characters
- Alphanumeric, hyphens, underscores, dots allowed
- Must start and end with alphanumeric character

### Label Selectors

#### Equality-based Selector
```bash
# Exact match
kubectl get pods -l app=nginx

# Not equal
kubectl get pods -l app!=nginx

# Multiple selectors (AND operation)
kubectl get pods -l app=nginx,env=production
```

#### Set-based Selector
```bash
# In
kubectl get pods -l "env in (production, staging)"

# Not in
kubectl get pods -l "env notin (development)"

# Exists
kubectl get pods -l "app"

# Does not exist
kubectl get pods -l "!app"
```

## Namespaces

### Introduction to Namespaces

Namespaces are virtual clusters within a physical Kubernetes cluster, used for isolating groups of resources.

#### Use Cases
- Multi-tenant environments
- Environment separation (dev, staging, prod)
- Team-based resource isolation
- Resource quota enforcement per team

### Default Namespaces

```bash
# kube-system: System components
kubectl get pods -n kube-system

# kube-public: Publicly accessible resources
kubectl get configmaps -n kube-public

# kube-node-lease: Node heartbeat leases
kubectl get leases -n kube-node-lease

# default: Default namespace for user resources
kubectl get pods  # Same as: kubectl get pods -n default
```

### Creating Namespaces

#### Using kubectl imperative command
```bash
kubectl create namespace dev-team
```

#### Using YAML declarative method
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-team
```

Apply it:
```bash
kubectl apply -f namespace.yaml
```

### Managing Namespaces

```bash
# List all namespaces
kubectl get namespace

# Describe namespace
kubectl describe namespace dev-team

# Get pods in specific namespace
kubectl get pods -n dev-team

# Get events in namespace
kubectl get events -n dev-team

# Get all events across all namespaces
kubectl get events -A

# Delete namespace (cascades to all resources)
kubectl delete namespace dev-team
```

### Setting Default Namespace

```bash
# Temporarily
kubectl config set-context --current --namespace=dev-team

# Verify
kubectl config view | grep namespace
```

## Resource Limits and Quotas

### Resource Quota

Resource quotas restrict compute resource usage within a namespace.

#### Creating Resource Quota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev-team
spec:
  hard:
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    pods: "100"
    persistentvolumeclaims: "5"
```

### LimitRange

LimitRange restricts resource usage for individual containers and pods.

#### Container LimitRange

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: container-limits
  namespace: dev-team
spec:
  limits:
  - type: Container
    min:
      cpu: "100m"
      memory: "128Mi"
    max:
      cpu: "2"
      memory: "2Gi"
    default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "250m"
      memory: "256Mi"
```

#### Pod LimitRange

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: pod-limits
spec:
  limits:
  - type: Pod
    min:
      cpu: "100m"
      memory: "128Mi"
    max:
      cpu: "4"
      memory: "4Gi"
```

## Events

### Kubernetes Events

Events are objects that record important occurrences in the cluster.

### Event Structure

- **Object**: What the event is about (Pod, Node, etc.)
- **Type**: Normal or Warning
- **Reason**: Why the event occurred
- **Message**: Human-readable description
- **Source**: Component that generated the event
- **Timestamp**: When event occurred
- **Count**: How many times this event occurred

### Event TTL (Time-to-Live)

Default event retention: **1 hour**

#### Configuring Event TTL

Edit kube-apiserver manifest:
```bash
sudo nano /etc/kubernetes/manifests/kube-apiserver.yaml
```

Add or modify:
```yaml
spec:
  containers:
  - name: kube-apiserver
    command:
    - kube-apiserver
    - --event-ttl=240m  # 4 hours
```

### Viewing Events

```bash
# Get all events in default namespace
kubectl get events

# Get events in specific namespace
kubectl get events -n dev-team

# Get all events across all namespaces
kubectl get events -A

# Describe pod to see related events
kubectl describe pod <pod-name>

# Export events to file
kubectl get events >> events.txt

# Watch events in real-time
kubectl get events -w
```

## Next Steps

- [Controllers](../04-Controllers/README.md)
- [Services & Networking](../05-Services/README.md)
