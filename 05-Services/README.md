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
- No direct external access
- Ideal for inter-pod communication

**Important EC2/VM note (temporary internet access):**
- You can use `kubectl port-forward` to access a ClusterIP-backed app temporarily.
- This works only while the `kubectl port-forward` command is running.
- In EC2/VM setups, if the host has a public IP and the forwarded port is allowed in Security Group/firewall rules, external users can reach it.
- For external reachability, bind explicitly: `kubectl port-forward --address 0.0.0.0 svc/nginx-service 8080:80`
- Use this only for temporary testing/troubleshooting, not as a permanent exposure method.

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

# Temporary access from admin machine/VM
kubectl port-forward svc/nginx-service 8080:80

# If internet access is required temporarily (be careful)
kubectl port-forward --address 0.0.0.0 svc/nginx-service 8080:80
```

**Traffic flow (ClusterIP + port-forward):**

```
┌──────────────────────┐
│   Internet Client    │
└──────────┬───────────┘
           │ Public IP:8080 (SG/Firewall allow)
┌──────────▼───────────┐
│ EC2/VM with kubectl  │
│     port-forward     │
└──────────┬───────────┘
           │ Tunnel
┌──────────▼───────────┐
│  ClusterIP Service   │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│ Endpoints/EndpointSlices │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│         Pod          │
└──────────────────────┘
```

### 2. NodePort

**Characteristics:**
- Service accessible on every node's IP
- Kubernetes assigns port in range 30000-32767
- Exposes service outside cluster
- Each node proxies the port

**Disadvantages (especially on EC2/cloud VMs):**
- Usually requires nodes to be reachable from outside (public IP or private connectivity via VPN/peering).
- NodePort must be opened in Security Groups/firewalls on the node port range or specific nodePort.
- The nodePort is exposed on all nodes, even nodes where the target pod is not running.
- Managing SG/firewall rules at scale becomes operationally heavy.

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

**Traffic flow (NodePort):**

```
┌──────────────────────┐
│   External Client    │
└──────────┬───────────┘
           │ NodeIP:NodePort
┌──────────▼───────────┐
│  Any Kubernetes Node │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│ kube-proxy (iptables │
│      or IPVS)        │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│  ClusterIP Service   │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│ Endpoints/EndpointSlices │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│     Backend Pod      │
└──────────────────────┘
```

### 3. LoadBalancer

**Characteristics:**
- Provisions external load balancer
- Works with cloud providers (AWS, GCP, Azure)
- Assigns external IP address
- Automatically creates NodePort and ClusterIP
- In AWS, LB forwards to node/EC2 private IPs on NodePort, then service routes to endpoints/pods

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

**Traffic flow (LoadBalancer):**

```
┌──────────────────────┐
│   External Traffic   │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│ Cloud Load Balancer  │
└──────────┬───────────┘
           │ Target Group
┌──────────▼───────────┐
│ Node/EC2 Private IP  │
│      :NodePort       │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│      kube-proxy      │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│       Service        │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│ Endpoints/EndpointSlices │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│        Pods          │
└──────────────────────┘
```

This is the typical path you mentioned:
`External traffic -> Node/EC2 private IP -> Service -> Endpoints -> Pods`

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

**DNS behavior (important):**
- Normal Service returns one virtual IP (ClusterIP).
- Headless Service returns pod IPs directly via DNS records.
- Commonly used with StatefulSet for stable network identity.

**Headless traffic view:**

```
┌──────────────────────┐
│      Client Pod      │
└──────────┬───────────┘
     │ DNS query: app-headless.default.svc.cluster.local
┌──────────▼───────────┐
│      CoreDNS         │
└──────────┬───────────┘
     │ Returns multiple pod IPs
┌──────────▼───────────┐
│ 10.244.1.10 (Pod A)  │
│ 10.244.2.20 (Pod B)  │
│ 10.244.3.30 (Pod C)  │
└──────────────────────┘
```

## Ingress and Ingress Controller

### What is Ingress?

Ingress is a Kubernetes API object that manages external HTTP/HTTPS access to services inside the cluster.

It provides:
- Host-based routing (example: `app.example.com` to one service)
- Path-based routing (example: `/api` to API service, `/` to web service)
- TLS/SSL termination at the edge

### Ingress vs Service

- Service (`ClusterIP`/`NodePort`/`LoadBalancer`) exposes one service endpoint.
- Ingress provides L7 routing rules across multiple services.
- Ingress requires an Ingress Controller to work.

### What is an Ingress Controller?

Ingress resource is just configuration. The controller is the actual data-plane component that reads those rules and routes traffic.

Popular controllers:
- NGINX Ingress Controller
- HAProxy Ingress
- Traefik
- Cloud-native controllers (AWS ALB, GCE Ingress)

### Traffic flow with Ingress

```
┌───────────────┐
│ Internet User │
└───────┬───────┘
  │ HTTPS/HTTP
┌───────▼──────────────────┐
│ LoadBalancer / NodePort  │
│ for Ingress Controller   │
└───────┬──────────────────┘
  │
┌───────▼──────────────────┐
│ Ingress Controller Pod   │
│ (NGINX/HAProxy/Traefik)  │
└───────┬──────────────────┘
  │ Reads Ingress rules
┌───────▼──────────────────┐
│       Ingress Rule       │
│ host + path + tls        │
└───────┬──────────────────┘
  │
┌───────▼───────────┐
│   Service (SVC)   │
└───────┬───────────┘
  │
┌───────▼───────────┐
│       Pods        │
└───────────────────┘
```

### Install Ingress Controller (NGINX) with kubeadm clusters

#### Step 1: Install controller manifest

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

#### Step 2: Verify controller pods

```bash
kubectl get pods -n ingress-nginx
kubectl get deploy -n ingress-nginx
```

#### Step 3: Verify controller service

```bash
kubectl get svc -n ingress-nginx
```

If your environment does not provide external LoadBalancer automatically, expose controller with NodePort or use MetalLB.

#### Step 4: Set ingressClassName

Use `ingressClassName: nginx` in Ingress resources.

### Path-based routing example

Use one host and split by URL path:
- `/api` -> `api-service`
- `/` -> `web-service`

See YAML: `05-Services/Examples/ingress-path-based-routing.yml`

### Header-based routing example

Header-based routing depends on controller capabilities.

For NGINX Ingress, a common method is canary annotations:
- Route to canary service when header exists (example: `x-canary: always`)

See YAML: `05-Services/Examples/ingress-header-based-routing.yml`

### TLS and SSL termination in Ingress

TLS termination means HTTPS is terminated at Ingress Controller. Backend service/pod can still run HTTP internally.

```
┌───────────────┐        TLS (443)        ┌─────────────────────────┐
│    Client     │ ───────────────────────> │ Ingress Controller      │
└───────────────┘                          │ (TLS termination point) │
             └───────────┬─────────────┘
                   │ HTTP (80)
             ┌───────────▼─────────────┐
             │  Service -> Pod backend  │
             └─────────────────────────┘
```

### Self-signed certificate: detailed steps

#### Step 1: Create private key and certificate

```bash
# Replace CN with your host
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key \
  -out tls.crt \
  -subj "/CN=demo.k8s.local/O=demo"
```

#### Step 2: Create Kubernetes TLS secret

```bash
kubectl create secret tls demo-tls-secret \
  --cert=tls.crt \
  --key=tls.key
```

#### Step 3: Create Ingress with TLS

Use `secretName: demo-tls-secret` under `spec.tls`.

See YAML: `05-Services/Examples/ingress-tls-selfsigned.yml`

#### Step 4: Map DNS/hosts entry for testing

For local testing, add your ingress endpoint IP to hosts file.

Linux/macOS:
```bash
sudo sh -c 'echo "<INGRESS_IP> demo.k8s.local" >> /etc/hosts'
```

Windows (Run editor as admin):
- File path: `C:\Windows\System32\drivers\etc\hosts`
- Add line: `<INGRESS_IP> demo.k8s.local`

#### Step 5: Test HTTPS endpoint

```bash
curl -k https://demo.k8s.local/
```

Notes:
- `-k` is needed because certificate is self-signed.
- For production, use trusted certs (for example, cert-manager + Let's Encrypt).

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

### External Access Comparison (EC2/VM)

```
ClusterIP (Temporary via Port-Forward)
┌───────────┐   ┌─────────────────────────┐   ┌──────────┐   ┌───────────┐   ┌──────┐
│ Internet  │-->| EC2/VM + port-forward  │-->| Service  │-->| Endpoints │-->| Pods │
└───────────┘   └─────────────────────────┘   └──────────┘   └───────────┘   └──────┘

NodePort
┌───────────┐   ┌─────────────────────────┐   ┌────────────┐   ┌──────────┐   ┌───────────┐   ┌──────┐
│ Internet  │-->| Any Node Public IP:Port │-->| kube-proxy │-->| Service  │-->| Endpoints │-->| Pods │
└───────────┘   └─────────────────────────┘   └────────────┘   └──────────┘   └───────────┘   └──────┘

LoadBalancer
┌───────────┐   ┌────────────────────┐   ┌────────────────────────┐   ┌────────────┐   ┌──────────┐   ┌───────────┐   ┌──────┐
│ Internet  │-->| Cloud LoadBalancer │-->| Node Private IP:Port   │-->| kube-proxy │-->| Service  │-->| Endpoints │-->| Pods │
└───────────┘   └────────────────────┘   └────────────────────────┘   └────────────┘   └──────────┘   └───────────┘   └──────┘
```

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

- Detailed Ingress guide: [Ingress](./Ingress/README.md)
- Path-based examples: [Path-Based](./Ingress/Path-Based/README.md)
- Header-based examples: [Header-Based](./Ingress/Header-Based/README.md)
- TLS self-signed steps: [TLS-Self-Signed](./Ingress/TLS-Self-Signed/README.md)
- [Health Probes](../06-Probes/README.md)
- [Advanced Topics](../07-Advanced/README.md)
