# Advanced Topics

## Taints and Tolerations

### Taints on Nodes

Taints allow nodes to repel pods that don't tolerate the taint.

#### Master Node Taints

By default, control plane (master) nodes have taints to prevent workload pods:

```bash
# View taints
kubectl describe node <master-node-name> | grep Taint

# Expected output:
# Taint: node-role.kubernetes.io/control-plane:NoSchedule
```

#### Taint Structure

```
key=value:effect
```

**Components:**
- **key**: Taint identifier
- **value**: Taint value
- **effect**: How to handle pods without toleration

#### Taint Effects

| Effect | Behavior |
|--------|----------|
| NoSchedule | Pod not scheduled on tainted node |
| NoExecute | Pod evicted if already running; not scheduled if new |
| PreferNoSchedule | Scheduler prefers not to schedule pod (soft constraint) |

### Adding and Removing Taints

```bash
# Add taint to node
kubectl taint nodes <node-name> key1=value1:NoSchedule

# Add taint (multiple)
kubectl taint nodes <node-name> key1=value1:NoSchedule key2=value2:NoExecute

# Remove taint
kubectl taint nodes <node-name> key1:NoSchedule-

# Remove all taints of a key
kubectl taint nodes <node-name> key1-

# Remove specific taint
kubectl taint nodes <node-name> node-role.kubernetes.io/control-plane:NoSchedule-
```

### Tolerations in Pods

Tolerations allow pods to tolerate (endure) node taints.

#### Toleration Syntax

```yaml
tolerations:
- key: key1
  operator: Equal
  value: value1
  effect: NoSchedule
  tolerationSeconds: 300  # For NoExecute only
```

#### Toleration Operators

| Operator | Behavior |
|----------|----------|
| Equal | Key/value must match exactly |
| Exists | Key must exist (value ignored) |

### Toleration Examples

#### Allow Pod on Master

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: master-pod
spec:
  containers:
  - name: app
    image: myapp:latest
  tolerations:
  - key: node-role.kubernetes.io/control-plane
    operator: Exists
    effect: NoSchedule
```

#### Allow Pod with Custom Taint

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-workload
spec:
  containers:
  - name: gpu-app
    image: gpu-app:latest
  tolerations:
  - key: gpu-type
    operator: Equal
    value: nvidia-a100
    effect: NoSchedule
```

#### Tolerate Node Eviction

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resilient-app
spec:
  containers:
  - name: app
    image: myapp:latest
  tolerations:
  - key: node.kubernetes.io/not-ready
    operator: Exists
    effect: NoExecute
    tolerationSeconds: 300  # Wait 300 sec before eviction
```

### Taint Use Cases

| Use Case | Taint | Toleration |
|----------|-------|-----------|
| Dedicated GPU nodes | gpu-type=nvidia:NoSchedule | Match gpu-type |
| Production only | workload-type=prod:NoSchedule | Match prod |
| Maintenance window | maintenance=true:NoExecute | Match with tolerationSeconds |
| Master/Control plane | node-role.kubernetes.io/control-plane:NoSchedule | Exists operator |

### Node Affinity vs Taints/Tolerations

**Node Affinity:**
- Pod specifies which nodes it wants
- "Pod attracts to node"

**Taints/Tolerations:**
- Node specifies which pods it accepts
- "Node repels certain pods"

## Lifecycle Hooks

### Pod and Container Lifecycle

```
Pod Created
    ↓
Init Containers (if defined)
    ↓
Main Container Starting
    ↓
postStart Hook (if defined)
    ↓
Readiness Probe (if defined)
    ↓
Container Ready
    ↓
preStop Hook (if defined) [when terminating]
    ↓
SIGTERM Sent
    ↓
Grace Period (default 30 sec)
    ↓
SIGKILL Sent
    ↓
Container Terminated
    ↓
Pod Deleted
```

### postStart Hook

Executed immediately after container creation, asynchronously.

#### postStart Execution

- Runs in parallel with main application
- Application may not be ready
- Failure doesn't prevent pod from running
- Block-level hook means container doesn't start until hook completes

#### postStart Examples

```yaml
# HTTP postStart
apiVersion: v1
kind: Pod
metadata:
  name: app-with-poststart
spec:
  containers:
  - name: app
    image: myapp:latest
    lifecycle:
      postStart:
        httpGet:
          path: /api/init
          port: 8080
```

```yaml
# Exec postStart
apiVersion: v1
kind: Pod
metadata:
  name: init-app
spec:
  containers:
  - name: app
    image: webapp:latest
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo 'App starting' > /tmp/app.log"]
```

### preStop Hook

Executed before container termination, synchronously.

#### preStop Execution

- Runs during pod termination
- Application receives preStop before SIGTERM
- Graceful shutdown opportunity
- Blocking hook delays SIGTERM (up to grace period)

#### preStop Examples

```yaml
# HTTP preStop for graceful shutdown
apiVersion: v1
kind: Pod
metadata:
  name: graceful-shutdown
spec:
  terminationGracePeriodSeconds: 30
  containers:
  - name: app
    image: webapp:latest
    lifecycle:
      preStop:
        httpGet:
          path: /api/shutdown
          port: 8080
```

```yaml
# Exec preStop for cleanup
apiVersion: v1
kind: Pod
metadata:
  name: cleanup-app
spec:
  terminationGracePeriodSeconds: 15
  containers:
  - name: app
    image: myapp:latest
    lifecycle:
      preStop:
        exec:
          command: ["/bin/sh", "-c", "sleep 10 && echo 'Cleanup done'"]
```

### Graceful Shutdown Pattern

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: graceful-app
spec:
  terminationGracePeriodSeconds: 30
  containers:
  - name: app
    image: production-app:latest
    ports:
    - containerPort: 8080

    # Health probes
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 5

    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 15

    # Graceful shutdown
    lifecycle:
      preStop:
        exec:
          command:
          - /bin/sh
          - -c
          - |
            # Remove from load balancer
            curl -X POST http://localhost:8080/shutdown
            # Wait for connections to close
            sleep 20
            # Cleanup
            echo "Graceful shutdown complete"
```

## Init Containers

Init containers run before main containers and must complete successfully.

### Characteristics

- Run sequentially (one at a time)
- Must complete successfully (exit code 0)
- Each init container must finish before next starts
- Restarts follow container restart policy

### Init Container Use Cases

1. **Setup and initialization**
2. **Database migration**
3. **Download configuration**
4. **Network setup**
5. **Secret provisioning**

### Init Container Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-example
spec:
  # Init containers run first
  initContainers:
  - name: wait-for-db
    image: busybox:1.28
    command: ['sh', '-c', "until nc -z postgres:5432; do echo waiting for postgres; sleep 2; done"]

  - name: migrate-db
    image: myapp:latest
    command: ["/migrate.sh"]

  # Main application container
  containers:
  - name: app
    image: myapp:latest
    ports:
    - containerPort: 8080
```

## Sidecar Containers

Sidecar containers run alongside main application container in same pod.

### Sidecar Use Cases

1. **Logging** - Log collector sidecar
2. **Monitoring** - Metrics exporter sidecar
3. **Security** - SSL/TLS proxy sidecar
4. **Service mesh** - Envoy proxy sidecar
5. **Backup** - Backup agent sidecar

### Sidecar Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  containers:
  # Main application
  - name: app
    image: myapp:latest
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: app-logs
      mountPath: /var/log/app

  # Logging sidecar
  - name: log-collector
    image: fluent-bit:latest
    volumeMounts:
    - name: app-logs
      mountPath: /var/log/app
    - name: fluent-bit-config
      mountPath: /fluent-bit/etc

  volumes:
  - name: app-logs
    emptyDir: {}
  - name: fluent-bit-config
    configMap:
      name: fluent-bit-config
```

## Container Types Summary

| Type | Purpose | Execution |
|------|---------|-----------|
| **Init Container** | Setup before main app | Sequential, must succeed |
| **Main Container** | Primary application | Runs until termination |
| **Sidecar Container** | Supporting functionality | Parallel with main |
| **Ephemeral Container** | Debugging | Attached to running pod |
| **Static Pod** | Node-level service | Run by kubelet directly |

## Node Reservations

### Node Resource Reservation

Kubernetes reserves resources on each node for system services:

```bash
# View node capacity vs allocatable
kubectl describe node <node-name> | grep -A 5 "Capacity\|Allocatable"
```

### Configuring Reservations

Edit kubelet config:
```bash
sudo nano /var/lib/kubelet/config.yaml
```

```yaml
kubeReserved:
  cpu: "500m"
  memory: "1Gi"
  ephemeralStorage: "10Gi"

systemReserved:
  cpu: "100m"
  memory: "512Mi"
  ephemeralStorage: "2Gi"

maxPods: 110  # Can be adjusted
```

Restart kubelet:
```bash
sudo systemctl restart kubelet
```

## Container Types Details

### Ephemeral Containers

Temporary containers for debugging running pods:

```bash
# Attach ephemeral container for debugging
kubectl debug <pod-name> -it --image=busybox

# Inside ephemeral container, debug the main app:
ps aux
netstat -an
df -h
```

### Static Pods

Pods created by kubelet reading manifests from local filesystem:

```bash
# Manifest directory (default)
/etc/kubernetes/manifests/

# kubelet checks this directory and creates pods
# Used for control plane components (API server, etcd, etc.)
```

## Next Steps

- [Cheatsheet](../08-Cheatsheet/README.md)
