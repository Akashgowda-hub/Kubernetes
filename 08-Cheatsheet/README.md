# Kubernetes Cheatsheet & Reference

Quick reference guide for common Kubernetes commands and configurations.

## Basic kubectl Commands

### Cluster Information

```bash
# Get cluster information
kubectl cluster-info

# Get nodes
kubectl get nodes
kubectl get nodes -o wide

# Get node details
kubectl describe node <node-name>

# Get API resources
kubectl api-resources

# Check Kubernetes version
kubectl version
```

### Context and Configuration

```bash
# View kubeconfig
kubectl config view

# Get current context
kubectl config current-context

# Set default namespace
kubectl config set-context --current --namespace=dev-team

# Switch context
kubectl config use-context <context-name>
```

## Pod Management

### Pod Operations

```bash
# Get pods
kubectl get pods
kubectl get pods -n <namespace>
kubectl get pods -A  # All namespaces
kubectl get pods -o wide

# Get pod details
kubectl describe pod <pod-name>

# Get pod YAML
kubectl get pod <pod-name> -o yaml

# Create pod from YAML
kubectl apply -f pod.yaml

# Create pod imperatively
kubectl run nginx --image=nginx:1.14

# Edit pod
kubectl edit pod <pod-name>

# Delete pod
kubectl delete pod <pod-name>

# Delete pod forcefully
kubectl delete pod <pod-name> --grace-period=0 --force
```

### Pod Logs and Debugging

```bash
# View logs
kubectl logs <pod-name>

# View logs from specific container (multi-container pod)
kubectl logs <pod-name> -c <container-name>

# View previous logs (before restart)
kubectl logs <pod-name> --previous

# Stream logs
kubectl logs <pod-name> -f

# View logs from multiple pods
kubectl logs -l app=nginx

# Get pod events
kubectl describe pod <pod-name> | grep -A 10 Events

# Port forward to pod
kubectl port-forward pod/<pod-name> 8080:8080

# Execute command in pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec <pod-name> -- ls -la

# Copy files from pod
kubectl cp <pod-name>:/path/file.txt ./local-file.txt

# Copy files to pod
kubectl cp ./local-file.txt <pod-name>:/path/file.txt
```

## Deployment Management

### Deployment Operations

```bash
# Get deployments
kubectl get deployments
kubectl get deploy -o wide

# Describe deployment
kubectl describe deployment <deployment-name>

# Create deployment
kubectl apply -f deployment.yaml

# Scale deployment
kubectl scale deployment <deployment-name> --replicas=5

# Update deployment image
kubectl set image deployment/<deployment-name> <container>=<image>:<tag>

# Update image (declarative)
kubectl apply -f deployment.yaml

# Delete deployment
kubectl delete deployment <deployment-name>
```

### Rollout Management

```bash
# Get rollout history
kubectl rollout history deployment/<deployment-name>

# Get details of specific revision
kubectl rollout history deployment/<deployment-name> --revision=2

# Check rollout status
kubectl rollout status deployment/<deployment-name>

# Pause rollout
kubectl rollout pause deployment/<deployment-name>

# Resume rollout
kubectl rollout resume deployment/<deployment-name>

# Rollback to previous version
kubectl rollout undo deployment/<deployment-name>

# Rollback to specific revision
kubectl rollout undo deployment/<deployment-name> --to-revision=2
```

## Services

### Service Operations

```bash
# Get services
kubectl get svc
kubectl get svc -o wide

# Describe service
kubectl describe svc <service-name>

# Create service
kubectl apply -f service.yaml

# Expose pod as service
kubectl expose pod <pod-name> --port=80 --target-port=8080 --type=LoadBalancer

# Get service endpoints
kubectl get endpoints <service-name>

# Port forward to service
kubectl port-forward svc/<service-name> 8080:80

# Delete service
kubectl delete svc <service-name>
```

## Namespace Operations

```bash
# List namespaces
kubectl get namespace

# Describe namespace
kubectl describe namespace <namespace-name>

# Create namespace
kubectl create namespace <namespace-name>

# Create namespace from YAML
kubectl apply -f namespace.yaml

# Set default namespace
kubectl config set-context --current --namespace=<namespace-name>

# Get resources in namespace
kubectl get all -n <namespace-name>

# Delete namespace
kubectl delete namespace <namespace-name>
```

## Labels and Selectors

```bash
# Get pods with specific label
kubectl get pods -l app=nginx

# Get pods with label key
kubectl get pods -l app

# Get pods without label
kubectl get pods -l '!app'

# Multiple label selectors
kubectl get pods -l app=nginx,env=prod

# Add label to resource
kubectl label pod <pod-name> version=v1

# Remove label
kubectl label pod <pod-name> version-

# Label nodes
kubectl label nodes <node-name> gpu=true
```

## Events and Monitoring

```bash
# Get cluster events
kubectl get events

# Get events in namespace
kubectl get events -n <namespace-name>

# Get all events
kubectl get events -A

# Watch events
kubectl get events -w

# Export events to file
kubectl get events >> events.txt

# View resource metrics (requires metrics-server)
kubectl top nodes
kubectl top pods
kubectl top pods -n <namespace-name>
```

## Configuration Management

### ConfigMaps

```bash
# Create ConfigMap from file
kubectl create configmap app-config --from-file=config.properties

# Create ConfigMap from literal
kubectl create configmap app-config --from-literal=key=value

# Get ConfigMaps
kubectl get configmaps

# Describe ConfigMap
kubectl describe configmap <configmap-name>

# Edit ConfigMap
kubectl edit configmap <configmap-name>

# Delete ConfigMap
kubectl delete configmap <configmap-name>
```

### Secrets

```bash
# Create secret from file
kubectl create secret generic db-secret --from-file=password.txt

# Create secret from literal
kubectl create secret generic db-secret --from-literal=password=mypassword

# Create TLS secret
kubectl create secret tls tls-secret --cert=cert.pem --key=key.pem

# Get secrets
kubectl get secrets

# View secret (base64 decoded)
kubectl get secret <secret-name> -o jsonpath='{.data.password}' | base64 -d
```

## Resource Quotas and Limits

```bash
# Get resource quotas
kubectl get resourcequota

# Describe resource quota
kubectl describe resourcequota <quota-name>

# Get limit ranges
kubectl get limitrange

# View namespace resource usage
kubectl describe namespace <namespace-name>
```

## Node Operations

```bash
# Get nodes
kubectl get nodes

# Describe node
kubectl describe node <node-name>

# Taint node
kubectl taint nodes <node-name> key=value:NoSchedule

# Remove taint
kubectl taint nodes <node-name> key:NoSchedule-

# Label node
kubectl label nodes <node-name> gpu=true

# Cordon node (prevent new pods)
kubectl cordon <node-name>

# Uncordon node
kubectl uncordon <node-name>

# Drain node (remove existing pods)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

## Advanced Operations

### Debugging

```bash
# Debug pod
kubectl debug pod/<pod-name> -it --image=busybox

# Attach to running container
kubectl attach <pod-name> -it

# Proxy API server
kubectl proxy

# Access dashboard
# Visit: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

# Get resource YAML
kubectl get pod <pod-name> -o yaml

# Get resource JSON
kubectl get pod <pod-name> -o json

# Dry run (don't actually apply)
kubectl apply -f deployment.yaml --dry-run=client

# Dry run and show output
kubectl apply -f deployment.yaml --dry-run=client -o yaml
```

### Scaling and Updates

```bash
# Scale resource
kubectl scale deployment <deployment-name> --replicas=5
kubectl scale rs <replicaset-name> --replicas=3

# Update strategy (rolling)
kubectl patch deployment <deployment-name> -p \
  '{"spec":{"strategy":{"type":"RollingUpdate","rollingUpdate":{"maxSurge":1,"maxUnavailable":1}}}}'

# Set resource requests and limits
kubectl set resources deployment <deployment-name> --requests=cpu=100m,memory=128Mi --limits=cpu=500m,memory=512Mi
```

### Batch Operations

```bash
# Delete multiple resources
kubectl delete pods,services -l app=myapp

# Delete all in namespace
kubectl delete all -n <namespace-name>

# Apply multiple files
kubectl apply -f file1.yaml -f file2.yaml

# Apply all files in directory
kubectl apply -f ./yaml-directory/

# Diff between local and cluster
kubectl diff -f deployment.yaml
```

## Common Service Types Decision Matrix

| Requirement | Service Type |
|-------------|--------------|
| Internal cluster access only | ClusterIP |
| Access from outside cluster | NodePort or LoadBalancer |
| External DNS name | ExternalName |
| No load balancing | Headless (clusterIP: None) |
| Cloud provider load balancer | LoadBalancer |
| Static external IPs | ExternalIP |

## Pod Phase and Container State Reference

### Pod Phases
- **Pending**: Accepted, containers not yet running
- **Running**: At least one container running
- **Succeeded**: All containers exited with code 0
- **Failed**: One or more containers exited with code ≠ 0
- **Unknown**: Pod state cannot be determined

### Container States
- **Waiting**: Container not running (pulling image, applying secret)
- **Running**: Container started successfully
- **Terminated**: Container execution ended

## Restart Policy Decision Guide

| Scenario | Restart Policy |
|----------|----------------|
| Web server | Always |
| REST API | Always |
| Batch job | OnFailure |
| Scheduled job | OnFailure |
| Database | Never |
| One-time task | Never |

## Exit Code Reference

| Code | Meaning |
|------|---------|
| 0 | Successful exit |
| 1 | General error |
| 128+2 | SIGINT (Ctrl+C) |
| 128+9 | SIGKILL (forceful termination) |
| 128+15 | SIGTERM (graceful termination) |
| 137 | OOM killed |
| 143 | Killed by SIGTERM |

## Crash Loop BackOff Debugging

```bash
# 1. Check pod status
kubectl describe pod <pod-name>

# 2. View application logs
kubectl logs <pod-name>

# 3. View previous logs (before crash)
kubectl logs <pod-name> --previous

# 4. Check events
kubectl get events | grep <pod-name>

# 5. Check resource limits
kubectl describe pod <pod-name> | grep -A 5 "Limits\|Requests"

# 6. Check exit code
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[0].lastState.terminated.exitCode}'
```

## Performance and Optimization Tips

```bash
# Get resource usage
kubectl top nodes
kubectl top pods --sort-by=memory

# List pods by CPU usage
kubectl top pods --sort-by=cpu

# Find pods without resource requests
kubectl get pod -o json | jq '.items[] | select(.spec.containers[].resources.requests == null) | .metadata.name'

# Check API server performance
kubectl get --raw /metrics | grep apiserver

# Monitor cluster events
kubectl get events -A -w
```

## Useful Aliases

Add to your shell configuration (~/.bashrc or ~/.zshrc):

```bash
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgd='kubectl get deployments'
alias kgs='kubectl get services'
alias kdp='kubectl describe pod'
alias kdd='kubectl describe deployment'
alias kds='kubectl describe service'
alias klp='kubectl logs pod'
alias kep='kubectl exec -it pod'
alias kaf='kubectl apply -f'
alias kdel='kubectl delete'
```

## Quick Problem Solving

### Pod won't start

```bash
# Step 1: Check pod status
kubectl describe pod <pod-name>

# Step 2: Check pod events
kubectl get events

# Step 3: Check logs
kubectl logs <pod-name> --previous

# Step 4: Check node resources
kubectl top nodes

# Step 5: Check readiness probe
kubectl describe pod <pod-name> | grep -A 5 "Readiness"
```

### Service unreachable

```bash
# Step 1: Check service exists
kubectl get svc <service-name>

# Step 2: Check endpoints
kubectl get endpoints <service-name>

# Step 3: Check pod selection
kubectl get pods -l <selector>

# Step 4: Test connectivity
kubectl run debug --image=busybox -it -- sh
# Inside pod: wget <service-name>:port
```

### Node not joining cluster

```bash
# Step 1: Check kubelet status
sudo systemctl status kubelet

# Step 2: Check kubelet logs
sudo journalctl -u kubelet

# Step 3: Check firewall
sudo ufw status

# Step 4: Verify container runtime
docker ps
# or
containerd version
```

## Further Resources

- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [API Server Documentation](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)
