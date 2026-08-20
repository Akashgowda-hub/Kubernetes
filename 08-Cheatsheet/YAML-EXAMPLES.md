# YAML Examples and Common Scenarios

Practical YAML examples for common Kubernetes scenarios.

## Pod Examples

### Simple Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-pod
  namespace: default
  labels:
    app: myapp
spec:
  containers:
  - name: app
    image: nginx:latest
    ports:
    - containerPort: 80
```

### Pod with Resource Limits

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-resources
spec:
  containers:
  - name: app
    image: myapp:latest
    resources:
      requests:
        cpu: "250m"
        memory: "256Mi"
      limits:
        cpu: "500m"
        memory: "512Mi"
```

### Multi-Container Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  # Main application
  - name: app
    image: myapp:latest
    ports:
    - containerPort: 8080

  # Logging sidecar
  - name: logger
    image: logging-agent:latest
    volumeMounts:
    - name: shared-data
      mountPath: /var/log

  # App container logs to shared volume
  volumes:
  - name: shared-data
    emptyDir: {}
```

### Pod with Init Container

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-init
spec:
  initContainers:
  - name: init
    image: busybox:latest
    command: ['sh', '-c', 'echo "Initialize..." > /init/data.txt']
    volumeMounts:
    - name: init-volume
      mountPath: /init

  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: init-volume
      mountPath: /data

  volumes:
  - name: init-volume
    emptyDir: {}
```

## Deployment Examples

### Basic Deployment

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
        image: nginx:1.14-alpine
        ports:
        - containerPort: 80
```

### Deployment with Rolling Update

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: webapp:1.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
```

### Deployment with Probes and Lifecycle

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: prod-app
  template:
    metadata:
      labels:
        app: prod-app
    spec:
      terminationGracePeriodSeconds: 30
      containers:
      - name: app
        image: production-app:1.0
        ports:
        - containerPort: 8080

        # Resource limits
        resources:
          requests:
            cpu: "250m"
            memory: "256Mi"
          limits:
            cpu: "1"
            memory: "1Gi"

        # Startup probe
        startupProbe:
          httpGet:
            path: /health/startup
            port: 8080
          failureThreshold: 30
          periodSeconds: 5

        # Readiness probe
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 2
          failureThreshold: 2

        # Liveness probe
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
          timeoutSeconds: 2
          failureThreshold: 3

        # Lifecycle hooks
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 10"]
```

## Service Examples

### ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8080
```

### NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30080
```

### LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8080
  - name: https
    protocol: TCP
    port: 443
    targetPort: 8443
```

### Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None
  selector:
    app: myapp
  ports:
  - name: http
    port: 80
    targetPort: 8080
```

### Multi-Port Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
spec:
  selector:
    app: myapp
  type: ClusterIP
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: metrics
    port: 9090
    targetPort: 9090
  - name: grpc
    port: 50051
    targetPort: 50051
```

## ConfigMap and Secret Examples

### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.host: "postgres.default.svc.cluster.local"
  database.port: "5432"
  app.settings: |
    debug=true
    log_level=info
```

### Using ConfigMap in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-config
spec:
  containers:
  - name: app
    image: myapp:latest
    envFrom:
    - configMapRef:
        name: app-config
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
```

### Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  # echo -n 'admin' | base64
  username: YWRtaW4=
  # echo -n 'password123' | base64
  password: cGFzc3dvcmQxMjM=
```

### Using Secret in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-secret
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

## StatefulSet Example

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: root-password
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

## DaemonSet Example

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - name: exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
        volumeMounts:
        - name: sys
          mountPath: /sys
        - name: root
          mountPath: /rootfs
      volumes:
      - name: sys
        hostPath:
          path: /sys
      - name: root
        hostPath:
          path: /
```

## Job Example

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-job
spec:
  parallelism: 2
  completions: 4
  backoffLimit: 3
  activeDeadlineSeconds: 3600
  template:
    metadata:
      labels:
        job: backup
    spec:
      containers:
      - name: backup
        image: backup-tool:latest
        command: ["./backup.sh"]
      restartPolicy: OnFailure
```

## CronJob Example

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-cleanup
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleanup
            image: db-tool:latest
            command: ["./cleanup.sh"]
          restartPolicy: OnFailure
      backoffLimit: 3
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
```

## Namespace with Quotas and Limits

```yaml
---
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: team-dev

---
# Resource Quota
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: team-dev
spec:
  hard:
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    pods: "100"
    persistentvolumeclaims: "5"

---
# Limit Range
apiVersion: v1
kind: LimitRange
metadata:
  name: container-limits
  namespace: team-dev
spec:
  limits:
  - type: Container
    min:
      cpu: "50m"
      memory: "64Mi"
    max:
      cpu: "2"
      memory: "2Gi"
    default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "250m"
      memory: "256Mi"
  - type: Pod
    min:
      cpu: "100m"
      memory: "128Mi"
    max:
      cpu: "4"
      memory: "4Gi"
```

## Pod Disruption Budget

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

## Network Policy Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    ports:
    - protocol: TCP
      port: 8080
```

## RBAC Example

```yaml
---
# Service Account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: default

---
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
spec:
  rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]

---
# Role Binding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
spec:
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: Role
    name: pod-reader
  subjects:
  - kind: ServiceAccount
    name: app-sa
    namespace: default
```

## Volume Examples

### EmptyDir

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-example
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: temp-storage
      mountPath: /tmp/data
  volumes:
  - name: temp-storage
    emptyDir: {}
```

### HostPath

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-example
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: host-storage
      mountPath: /data
  volumes:
  - name: host-storage
    hostPath:
      path: /var/data
      type: DirectoryOrCreate
```

### ConfigMap Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
```

## Best Practices

### Pod Quality of Service (QoS) Classes

#### Guaranteed (Highest Priority)
```yaml
spec:
  containers:
  - name: app
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "100m"
        memory: "128Mi"  # requests == limits
```

#### Burstable (Medium Priority)
```yaml
spec:
  containers:
  - name: app
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "512Mi"  # requests < limits
```

#### BestEffort (Lowest Priority)
```yaml
spec:
  containers:
  - name: app
    # No resources specified
```

## Next Steps

- Refer to [Cheatsheet](README.md) for command reference
- Check [Advanced Topics](../07-Advanced/README.md) for complex scenarios
