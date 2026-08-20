# Health Probes

## Overview of Probes

Probes allow Kubernetes to determine the health and readiness of containers and applications. They are fundamental for maintaining application reliability.

## Types of Probes

### 1. Startup Probe

**Purpose**: Determine if application has started successfully

**Characteristics:**
- First probe to run during container startup
- Disables liveness and readiness probes until successful
- Useful for applications with long startup times
- Prevents premature termination of starting apps

**Use Cases:**
- Applications with slow initialization
- Database migrations on startup
- Large dataset loading
- Complex initialization logic

**Example:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: slow-startup-app
spec:
  containers:
  - name: app
    image: myapp:latest
    startupProbe:
      httpGet:
        path: /health/startup
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
```

### 2. Readiness Probe

**Purpose**: Determine if container is ready to receive traffic

**Characteristics:**
- Checked periodically during pod lifecycle
- Failed probes remove pod from service endpoints
- Container continues running even if probe fails
- Traffic routed only to ready pods

**Use Cases:**
- Application in maintenance mode
- Database connection failures
- Cache warm-up phase
- Temporary service dependencies unavailable

**Example:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: app
    image: webapp:latest
    readinessProbe:
      httpGet:
        path: /health/ready
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
      timeoutSeconds: 2
      failureThreshold: 3
```

### 3. Liveness Probe

**Purpose**: Determine if container should be restarted

**Characteristics:**
- Checks if application is still running
- Failed probes trigger container restart
- Used to recover from deadlocks or hangs
- Container replaced even if process exists

**Use Cases:**
- Application deadlock detection
- Hang/freeze detection
- Resource exhaustion recovery
- Infinite loops or stuck threads

**Example:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-monitoring
spec:
  containers:
  - name: app
    image: myapp:latest
    livenessProbe:
      httpGet:
        path: /health/live
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 10
      timeoutSeconds: 2
      failureThreshold: 3
```

## Probe Check Methods

### 1. HTTP GET

Performs HTTP GET request to container

```yaml
httpGet:
  path: /healthz
  port: 8080
  httpHeaders:
  - name: Authorization
    value: "Bearer token123"
  scheme: HTTP  # or HTTPS
```

**HTTP Response Codes:**
| Code Range | Status |
|-----------|--------|
| 1xx | Information (treated as failure) |
| 2xx | Success (success) |
| 3xx | Redirection (treated as failure) |
| 4xx | Client Error (treated as failure) |
| 5xx | Server Error (treated as failure) |

### 2. TCP Socket

Attempts TCP connection to container port

```yaml
tcpSocket:
  port: 8080
  host: localhost
```

**Use Cases:**
- Database connectivity checks
- Connection pool monitoring
- Custom TCP protocol validation

### 3. Exec

Executes command inside container

```yaml
exec:
  command:
  - /bin/sh
  - -c
  - "check-status.sh"
```

**Exit Code:**
- Exit code 0: Success
- Non-zero exit code: Failure

**Use Cases:**
- Complex health checks
- File system validation
- Custom scripts
- Database query execution

### 4. gRPC

Calls gRPC service for health check (Kubernetes 1.24+)

```yaml
grpc:
  port: 50051
  service: grpc.health.v1.Health
```

## Probe Configuration Parameters

### Timing Parameters

```yaml
probes:
  initialDelaySeconds: 10    # Wait before first check (default: 0)
  periodSeconds: 10          # Check interval (default: 10)
  timeoutSeconds: 2          # Timeout per check (default: 1)
  successThreshold: 1        # Consecutive successes to mark healthy (default: 1)
  failureThreshold: 3        # Consecutive failures to mark unhealthy (default: 3)
```

### Configuration Details

| Parameter | Default | Description |
|-----------|---------|-------------|
| initialDelaySeconds | 0 | Delay before first probe |
| periodSeconds | 10 | How often to perform probe |
| timeoutSeconds | 1 | Probe timeout duration |
| successThreshold | 1 | Successes needed after failure to mark healthy |
| failureThreshold | 3 | Failures needed to mark unhealthy |

## Complete Probe Examples

### Multi-Probe Application

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: complete-health-check
spec:
  containers:
  - name: app
    image: webapp:latest
    ports:
    - containerPort: 8080

    # Startup: App initialization
    startupProbe:
      httpGet:
        path: /api/startup
        port: 8080
      failureThreshold: 30
      periodSeconds: 5

    # Readiness: Ready for traffic
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      timeoutSeconds: 1
      failureThreshold: 2

    # Liveness: Still alive
    livenessProbe:
      httpGet:
        path: /api/live
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 10
      timeoutSeconds: 2
      failureThreshold: 3
```

### Database Probe

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: postgres-pod
spec:
  containers:
  - name: postgres
    image: postgres:14

    # Check if database is ready
    readinessProbe:
      exec:
        command:
        - /bin/sh
        - -c
        - "pg_isready -U postgres"
      initialDelaySeconds: 10
      periodSeconds: 5

    # Check if database is alive
    livenessProbe:
      exec:
        command:
        - /bin/sh
        - -c
        - "pg_isready -U postgres"
      initialDelaySeconds: 30
      periodSeconds: 10
```

### TCP Connection Probe

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-pod
spec:
  containers:
  - name: redis
    image: redis:7

    readinessProbe:
      tcpSocket:
        port: 6379
      initialDelaySeconds: 5
      periodSeconds: 5

    livenessProbe:
      tcpSocket:
        port: 6379
      initialDelaySeconds: 15
      periodSeconds: 10
```

## Probe Behavior Guide

### Startup Probe Behavior

```
Container started
    ↓
Startup probe runs every 5 seconds
    ↓
Max 30 failures (150 seconds total)
    ↓
Success → Disable startup probe
         → Enable readiness & liveness probes
    ↓
All failures → Container marked Failed
               → Restart according to restartPolicy
```

### Readiness Probe Behavior

```
Readiness probe runs periodically
    ↓
Failure → Pod removed from service endpoints
         → Traffic no longer routed to pod
         → Container continues running
    ↓
Success → Pod added back to endpoints
         → Traffic resumes
```

### Liveness Probe Behavior

```
Liveness probe runs periodically
    ↓
Failure → Container marked unhealthy
         → Container killed
         → New container started (restartPolicy)
    ↓
Success → Container remains running
```

## Probe Decision Tree

### Choosing Probe Type

**Startup Probe?**
- Long initialization time → **Yes**
- Quick startup → **No**

**Readiness Probe?**
- Service depends on internal state → **Yes**
- Stateless service → **Optional**
- Load-balanced service → **Yes**

**Liveness Probe?**
- Application can hang/deadlock → **Yes**
- Self-healing application → **No**
- Critical service → **Yes**

## Best Practices

### 1. Dedicated Endpoints
- Create separate health check endpoints
- Example: `/health/startup`, `/health/ready`, `/health/live`

### 2. Appropriate Thresholds
```yaml
# For critical services
failureThreshold: 3
periodSeconds: 5

# For development
failureThreshold: 1
periodSeconds: 10
```

### 3. Timeout Configuration
```yaml
timeoutSeconds: 2  # Reasonable timeout
periodSeconds: 10  # Period >= timeoutSeconds + buffer
```

### 4. Avoid Cascading Failures
```yaml
# Don't check external services in probe
# Instead, check internal state
readinessProbe:
  httpGet:
    path: /internal-health  # Internal check
    port: 8080
```

### 5. Logging
- Log probe failures
- Include probe reason in application logs
- Monitor probe metrics

## Common Pitfalls

### ❌ Wrong Response Code
```yaml
# ❌ Treating redirect as success
httpGet:
  path: /health
  port: 8080
  # If returns 301 → failure
```

### ❌ Overly Aggressive Probes
```yaml
# ❌ Too frequent = high overhead
periodSeconds: 1
failureThreshold: 1
# Results in false positives
```

### ❌ Ignoring Startup Time
```yaml
# ❌ Startup probe not configured
startupProbe: null
readinessProbe:
  initialDelaySeconds: 5  # Too short for slow apps
```

### ✅ Correct Configuration
```yaml
startupProbe:
  httpGet:
    path: /ready
    port: 8080
  failureThreshold: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5

livenessProbe:
  httpGet:
    path: /live
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10
```

## Monitoring Probes

```bash
# Describe pod to see probe status
kubectl describe pod <pod-name>

# View probe events
kubectl get events

# Check container restart count
kubectl get pods -o wide

# Monitor probe metrics (if Prometheus installed)
kubectl port-forward -n kube-system svc/kube-metrics 8080:8080
```

## Next Steps

- [Advanced Topics](../07-Advanced/README.md)
- [Cheatsheet](../08-Cheatsheet/README.md)
