# Services and Networking

## Introduction to Services

Services are Kubernetes objects that provide stable network endpoints to access pods. They abstract away the ephemeral nature of pod IPs and names.

### Why Services?

- **Stable Endpoint**: Pod IPs change, service IP remains constant
- **Load Balancing**: Distribute traffic across multiple pod replicas
- **Service Discovery**: DNS-based automatic discovery
- **Network Abstraction**: Decouple clients from backend pods

## Service Types

### 1. ClusterIP (Default)

**Characteristics:**
- Service only accessible within the cluster
- Kubernetes assigns internal IP address
- No external access
- Ideal for inter-pod communication

**Example YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80              # Service port
    targetPort: 8080      # Pod port
```

**Access:**
```bash
# From within cluster
curl http://nginx-service:80
curl http://nginx-service.default.svc.cluster.local:80
```

### 2. NodePort

**Characteristics:**
- Service accessible on every node's IP
- Kubernetes assigns port in range 30000-32767
- Exposes service outside cluster
- Each node proxies the port

**Example YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80              # Service port (internal)
    targetPort: 8080      # Pod port
    nodePort: 30001       # Port on each node (30000-32767)
```

**Access:**
```bash
# From external client
curl http://<NODE_IP>:30001
curl http://<NODE_HOSTNAME>:30001
```

### 3. LoadBalancer

**Characteristics:**
- Provisions external load balancer
- Works with cloud providers (AWS, GCP, Azure)
- Assigns external IP address
- Automatically creates NodePort and ClusterIP

**Example YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**Access:**
```bash
# Get external IP
kubectl get svc nginx-loadbalancer

# Access via external IP
curl http://<EXTERNAL_IP>:80
```

### 4. ExternalIP

**Characteristics:**
- Maps external IPs to pods
- Manually assigned IPs
- Requires network infrastructure setup
- Minimal use case

**Example YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-external
spec:
  selector:
    app: nginx
  externalIPs:
  - 192.168.1.100
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### 5. ExternalName

**Characteristics:**
- Routes to external DNS name
- No selector or endpoints
- Creates CNAME DNS record
- Used for external service integration

**Example YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: database.example.com
  ports:
  - protocol: TCP
    port: 5432
```

**Access:**
```bash
# From within cluster
curl http://external-db:5432
```

### 6. Headless Service

**Characteristics:**
- No ClusterIP assigned (set to None)
- DNS returns all pod IPs
- Used for StatefulSets and direct pod access
- No load balancing

**Example YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

## Service Configuration Details

### Port Terminology

```yaml
apiVersion: v1
kind: Service
spec:
  ports:
  - name: http
    protocol: TCP              # TCP or UDP (default: TCP)
    port: 80                   # Service port (exposed internally)
    targetPort: 8080           # Pod container port
    nodePort: 30080            # Node port (30000-32767, NodePort/LB only)
```

| Port Type | Purpose |
|-----------|---------|
| **port** | Service port (internal to cluster) |
| **targetPort** | Container port inside pod |
| **nodePort** | Port on each node (NodePort/LoadBalancer services) |

### Supported Protocols

```yaml
ports:
- protocol: TCP      # Default
- protocol: UDP
- protocol: SCTP
```

## Service Discovery

### DNS-based Discovery

Kubernetes automatically creates DNS records:

```
<service-name>.<namespace>.svc.cluster.local
```

**Examples:**
```bash
# Service in same namespace
curl http://nginx-service:80

# Service in different namespace
curl http://nginx-service.prod.svc.cluster.local:80
```

### Environment Variables

Kubernetes injects environment variables for services in the same namespace:

```bash
# Service: mysql-service in default namespace
MYSQL_SERVICE_HOST=10.96.0.1
MYSQL_SERVICE_PORT=3306
```

## kube-proxy and Service Networking

### What is kube-proxy?

kube-proxy is a network proxy running on every node that maintains network rules for service routing.

### Proxy Modes

#### 1. iptables Mode (Default)
- Uses Linux iptables for packet filtering
- Lower overhead
- Suitable for moderate number of services
- Issues with scaling to thousands of services

**Detection:**
```bash
ps aux | grep kube-proxy | grep iptables
```

#### 2. IPVS Mode (IP Virtual Server)
- Uses Linux IPVS for load balancing
- Better performance and scaling
- Supports advanced load balancing algorithms
- Requires IPVS kernel module

**Configuration:**
```bash
# Check if mode is IPVS
kubectl logs -n kube-system -l component=kube-proxy
```

#### 3. Userspace Mode (Deprecated)
- kube-proxy acts as proxy for TCP/UDP
- High CPU overhead
- No longer recommended
- Replaced by iptables and IPVS

#### 4. nftables Mode (Emerging)
- Modern replacement for iptables
- Improved performance
- Kernel 5.4+

### Proxy Mode Selection

```bash
# Configure kube-proxy mode
# Edit /etc/kubernetes/manifests/kube-proxy-config.yaml
# or via command line:
kube-proxy --proxy-mode=ipvs
```

## Service Endpoints and EndpointSlices

### Endpoints

Kubernetes automatically creates Endpoints objects tracking service member pods:

```bash
# View endpoints
kubectl get endpoints nginx-service

# Describe endpoints
kubectl describe endpoints nginx-service
```

**Endpoint Structure:**
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: nginx-service
subsets:
- addresses:
  - ip: 10.244.1.10
    targetRef:
      kind: Pod
      name: nginx-1
      namespace: default
  ports:
  - port: 8080
```

### EndpointSlices

Modern alternative to Endpoints, supporting larger scale:

```bash
# View EndpointSlices
kubectl get endpointslices

# Describe EndpointSlice
kubectl describe endpointslice nginx-service-abc
```

## Service Commands

```bash
# Get services
kubectl get svc

# Get services with more details
kubectl get svc -o wide

# Describe service
kubectl describe svc nginx-service

# Get service endpoints
kubectl get endpoints nginx-service

# Port forward to service
kubectl port-forward svc/nginx-service 8080:80

# Delete service
kubectl delete svc nginx-service
```

## Networking Architecture

### Pod-to-Pod Communication

```
Pod A (10.244.1.10:8080) ─→ Pod B (10.244.2.20:80)
      └─ Direct network path (overlay network from CNI)
```

### Pod-to-Service Communication

```
Pod A ─→ Service (10.96.0.1:80)
      └─ kube-proxy (iptables/IPVS)
      └─→ Pod B (10.244.2.20:80)
      └─→ Pod C (10.244.3.30:80)
      └─→ Pod D (10.244.4.40:80)
```

### Pod-to-External Communication

```
Pod A ─→ Service (Type: LoadBalancer/NodePort)
      └─ Node Port (30080)
      └─ External LB / Node IP
      └─→ External Client
```

## HostPort

**Note:** Not recommended for production use. Use Services instead.

hostPort exposes a container port directly on the host.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-hostport
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
      hostPort: 8080        # Pod accessible on host:8080
```

**Issues:**
- Port conflicts when multiple pods use same hostPort
- Not load balanced
- Pod-to-node binding is rigid
- Use Services instead for proper load balancing

## Service Examples

### Web Application Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-frontend
spec:
  type: LoadBalancer
  selector:
    app: web
    tier: frontend
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
```

### Database Service (ClusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres-db
spec:
  type: ClusterIP
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
```

### Multi-port Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
spec:
  selector:
    app: myapp
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: metrics
    port: 9090
    targetPort: 9090
  - name: debug
    port: 40000
    targetPort: 40000
```

## Next Steps

- [Health Probes](../06-Probes/README.md)
- [Advanced Topics](../07-Advanced/README.md)
