# Probes - YAML Examples

This directory contains practical YAML examples for Kubernetes health probes.

## Files Overview

### Basic Probes
- `probe-http.yml` - HTTP GET health checks
- `probe-tcp.yml` - TCP socket connection checks
- `probe-exec.yml` - Custom command execution checks
- `probe-grpc.yml` - gRPC health checks

### Complete Examples
- `probe-all-types.yml` - Comprehensive startup, readiness, and liveness probes
- `probe-production.yml` - Production-ready probe configuration

## Probe Types Flowchart

```
Container Lifecycle
    ↓
START
    ↓
Startup Probe (if defined)
    ↓ Success → Disable startup probe
    ↓        → Enable readiness & liveness
    ↓
RUNNING
    ├─ Readiness Probe (periodic)
    │  ├─ Success → Add to service endpoints
    │  └─ Failure → Remove from service endpoints
    │
    └─ Liveness Probe (periodic)
       ├─ Success → Container continues
       └─ Failure → Container killed & restarted
    ↓
Receiving Traffic (if ready)
    ↓
TERMINATION
    ↓
preStop Hook (if defined)
    ↓
SIGTERM sent
    ↓
Grace Period (default 30 sec)
    ↓
SIGKILL sent (if still running)
    ↓
END
```

## Probe Decision Matrix

| Scenario | Startup | Readiness | Liveness |
|----------|---------|-----------|----------|
| Slow app init | Yes | Yes | Yes |
| Quick startup | No | Yes | Yes |
| Stateless service | No | Yes | Yes |
| Critical app | Yes | Yes | Yes |
| Development | No | Optional | Optional |
| Database | Yes | Yes | No |

## Quick Commands

```bash
# Apply probe examples
kubectl apply -f probe-http.yml
kubectl apply -f probe-all-types.yml

# View probe status
kubectl describe pod <pod-name>
kubectl get pods -o wide

# Check probe events
kubectl get events | grep probe

# View pod restart count (indicates probe failures)
kubectl get pods -o custom-columns=NAME:.metadata.name,RESTARTS:.status.containerStatuses[0].restartCount

# Watch probe activity
kubectl logs <pod-name> -f
```

## HTTP Probe Status Codes

| Code | Result |
|------|--------|
| 200-399 | Success ✓ |
| 400-599 | Failure ✗ |
| Timeout | Failure ✗ |
| Connection refused | Failure ✗ |

## Probe Parameter Guidelines

```yaml
# For slow applications
startupProbe:
  failureThreshold: 30      # ~5 min at 10sec interval
  periodSeconds: 10

# For responsive applications
readinessProbe:
  periodSeconds: 5          # Check every 5 sec
  timeoutSeconds: 2         # Wait 2 sec for response
  failureThreshold: 2       # Fail after 2 misses

# For critical services
livenessProbe:
  periodSeconds: 10         # Check every 10 sec
  timeoutSeconds: 2         # Wait 2 sec for response
  failureThreshold: 3       # Restart after 3 misses
```

See individual YAML files for detailed examples and best practices.
