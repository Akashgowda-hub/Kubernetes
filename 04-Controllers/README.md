
# Kubernetes Controllers

## Overview of Controllers

Controllers are Kubernetes control loop abstractions that manage the state of objects and attempt to move the current state toward the desired state.

## Replication Controller (Legacy)

### Characteristics
- **Selector**: Only supports equality-based label selectors
- **Status**: Causes temporary downtime during updates
- **Deprecated**: Replaced by ReplicaSet

### YAML Example
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14
```

## ReplicaSet

### Characteristics
- **Selectors**: Supports set-based and equality-based label selectors
- **Desired State**: Maintains specified number of pod replicas
- **Automatic Recovery**: Creates new pods if replicas fall below desired count
- **Direct Use**: Rarely created directly; usually managed by Deployment

### Label Selectors Supported

#### Equality-based
```yaml
selector:
  matchLabels:
    app: nginx
    version: v1
```

#### Set-based (Expression)
```yaml
selector:
  matchExpressions:
  - key: app
    operator: In
    values: ["nginx", "apache"]
  - key: version
    operator: NotIn
    values: ["v0.9"]
  - key: tier
    operator: Exists
  - key: deprecated
    operator: DoesNotExist
```

### YAML Example
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14
        ports:
        - containerPort: 80
```

### Commands
```bash
# Create ReplicaSet
kubectl apply -f replicaset.yaml

# Get ReplicaSets
kubectl get rs

# Describe ReplicaSet
kubectl describe rs nginx-rs

# Scale ReplicaSet
kubectl scale rs nginx-rs --replicas=5

# Delete ReplicaSet (cascade deletes pods)
kubectl delete rs nginx-rs

# Delete ReplicaSet (orphan pods)
kubectl delete rs nginx-rs --cascade=orphan
```

## Deployment

### Characteristics
- **Level Above ReplicaSet**: Manages ReplicaSets, allowing rollback and rollout
- **Rolling Updates**: Gradual update with zero downtime
- **Version History**: Maintains revision history for rollback
- **Most Common Controller**: Recommended for most use cases

### Deployment Workflow

```
Deployment → ReplicaSet → Pods

Update Deployment
    ↓
New ReplicaSet created
    ↓
Old ReplicaSet scaled down (max-unavailable)
New ReplicaSet scaled up (max-surge)
    ↓
Rolling update completes
Old ReplicaSet retained for rollback
```

### YAML Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14
        ports:
        - containerPort: 80
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

### Update Strategies

#### 1. Rolling Update (Default)
- Gradually replaces old pods with new ones
- Maintains service availability

**Key Parameters:**
- `maxSurge`: Max pods above desired replicas during update (default: 25%)
- `maxUnavailable`: Max pods unavailable during update (default: 25%)

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 2          # 2 extra pods allowed
    maxUnavailable: 1    # 1 pod can be unavailable
```

#### 2. Recreate
- Terminates all old pods before starting new ones
- Causes service downtime
- Faster than rolling update

```yaml
strategy:
  type: Recreate
```

### Deployment Revision History

Kubernetes maintains up to 10 revisions by default.

#### Managing Revisions

```bash
# Get deployment rollout history
kubectl rollout history deployment nginx-deployment

# Get details of specific revision
kubectl rollout history deployment nginx-deployment --revision=2

# Rollback to previous revision
kubectl rollout undo deployment nginx-deployment

# Rollback to specific revision
kubectl rollout undo deployment nginx-deployment --to-revision=2

# Pause rollout
kubectl rollout pause deployment nginx-deployment

# Resume rollout
kubectl rollout resume deployment nginx-deployment

# Check rollout status
kubectl rollout status deployment nginx-deployment
```

### Deployment Commands

```bash
# Create deployment
kubectl apply -f deployment.yaml

# Get deployments
kubectl get deployments

# Describe deployment
kubectl describe deployment nginx-deployment

# Update image
kubectl set image deployment/nginx-deployment nginx=nginx:1.16

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5

# Delete deployment
kubectl delete deployment nginx-deployment
```

## DaemonSet

### Characteristics
- **One Pod Per Node**: Ensures one pod runs on every node
- **Automatic Scheduling**: New pods on new nodes
- **Automatic Cleanup**: Removes pods when nodes are deleted
- **Taints Ignored**: By default ignores taints

### Use Cases
- Node monitoring (Prometheus node-exporter)
- Logging (Fluentd, Logstash)
- Network plugins (Calico, Weave)
- System monitoring (CollectD, New Relic)

### YAML Example

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-daemonset
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.14
        volumeMounts:
        - name: logs
          mountPath: /var/log
      volumes:
      - name: logs
        hostPath:
          path: /var/log
```

### DaemonSet Commands

```bash
# Create DaemonSet
kubectl apply -f daemonset.yaml

# Get DaemonSets
kubectl get daemonsets

# Describe DaemonSet
kubectl describe daemonset fluentd-daemonset

# Check DaemonSet status
kubectl get ds -o wide

# Delete DaemonSet
kubectl delete daemonset fluentd-daemonset
```

## Cascade Deletion

### Foreground Cascade (Default)
```bash
# Delete parent object and all children
kubectl delete deployment nginx-deployment
```

### Background Cascade
```bash
# Delete parent, but children become orphaned
kubectl delete deployment nginx-deployment --cascade=background
```

### Orphan Deletion
```bash
# Delete parent without deleting children
kubectl delete deployment nginx-deployment --cascade=orphan
```

## StatefulSet (Brief Overview)

### Characteristics
- Manages stateful applications
- Maintains pod identity
- Ordered pod creation and termination
- Persistent storage per pod

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  serviceName: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
```

## Job (Brief Overview)

### Characteristics
- Creates pods that run to completion
- Manages successful completion
- Handles retries on failure

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  parallelism: 2
  completions: 4
  backoffLimit: 3
  template:
    spec:
      containers:
      - name: batch-worker
        image: worker:latest
      restartPolicy: Never
```

## CronJob (Brief Overview)

### Characteristics
- Runs jobs on schedule
- Based on Cron format
- Maintains job history

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-backup
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:latest
          restartPolicy: OnFailure
```

## Next Steps

- [Services & Networking](../05-Services/README.md)
- [Health Probes](../06-Probes/README.md)
