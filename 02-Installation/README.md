# Kubernetes Installation Guide

## Installation Methods

### 1. Using kubeadm (Recommended)

kubeadm is a tool for bootstrapping a minimal Kubernetes cluster that conforms to best practices.

#### Prerequisites
- Linux OS (Ubuntu, CentOS, etc.)
- 2+ CPUs on master node
- 2GB+ RAM
- Network connectivity between nodes
- Unique hostname, MAC address, and product_uuid on each node
- Certain ports open and firewall rules configured

#### Installation Steps

##### Step 1: Install Container Runtime (containerd)

```bash
# Update system packages
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg

# Add Docker GPG key and repository
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install containerd
sudo apt-get update
sudo apt-get install -y containerd.io

# Configure containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```

##### Step 2: Install kubeadm, kubelet, and kubectl

```bash
# Add Kubernetes repository
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list

# Install kubelet, kubeadm, kubectl
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

##### Step 3: Initialize the Cluster (Master/Control Plane)

```bash
# Initialize cluster
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=<MASTER_IP>

# Setup kubeconfig for current user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Verify installation
kubectl cluster-info
kubectl get nodes
```

##### Step 4: Install Container Network Interface (CNI) - Calico

```bash
# Install Calico
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/tigera-operator.yaml

# Wait for operator to be ready
kubectl wait --for=condition=Ready pod -l k8s-app=tigera-operator -n tigera-operator --timeout=300s

# Create Calico installation
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/custom-resources.yaml

# Verify Calico installation
kubectl get tigerastatus
```

##### Step 5: Join Worker Nodes

```bash
# On master node, get the join command
kubeadm token create --print-join-command

# On worker node, run the join command (replace with actual command)
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>

# Verify on master
kubectl get nodes
```

## kubeadm Reset

If you need to reset the cluster and start over:

### 4-Phase Reset Process

1. **preflight**: Pre-reset checks
2. **update-cluster-status**: Remove current node from cluster
3. **remove-etcd-members**: Remove etcd members (control plane only)
4. **cleanup-node**: Clean up kubelet state

### Reset Commands

```bash
# Full reset
sudo kubeadm reset

# Reset skipping specific phases
sudo kubeadm reset --skip-phases remove-etcd-members

# If CNI plugin was removed, reinstall it
# For Calico:
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/tigera-operator.yaml
```

## Advanced Installation Scenarios

### Installation with Specific kubeadm Configurations

```bash
# Custom pod network CIDR
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Custom service CIDR
sudo kubeadm init --service-cidr=10.96.0.0/12

# Specific Kubernetes version
sudo kubeadm init --kubernetes-version=1.26.0
```

### Joining Worker Nodes with Discovery File

```bash
# Method 1: Token-based discovery
kubeadm join --discovery-token-unsafe-skip-ca-verification <MASTER_IP>:6443 --token <TOKEN>

# Method 2: File-based discovery (recommended for production)
kubeadm join --discovery-file https://<MASTER_IP>:6443/discovery.crt
```

## Post-Installation Verification

```bash
# Check cluster components
kubectl get componentstatuses

# Check nodes status
kubectl get nodes -o wide

# Check system pods
kubectl get pods -n kube-system

# Check network connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
# Inside pod: nslookup kubernetes.default
# nslookup any-service.any-namespace
```

## Troubleshooting Installation

### Common Issues

1. **API Server unreachable**
   - Check firewall rules for port 6443
   - Verify network connectivity between nodes

2. **Pod network not working**
   - Verify CNI plugin is installed
   - Check CNI pod logs: `kubectl logs -n kube-system -l <cni-label>`

3. **Nodes not joining**
   - Verify kubeadm token hasn't expired
   - Check firewall between master and worker nodes
   - Verify container runtime is running on worker nodes

## Node Configuration

### kubelet Configuration Location
```
/var/lib/kubelet/config.yaml
```

### Key Configuration Parameters
- `maxPods`: Default 110 pods per node (can be changed)
- `kubeReserved`: Reserved resources for kubelet
- `systemReserved`: Reserved resources for OS

```bash
# Change max pods per node
# Edit /var/lib/kubelet/config.yaml and change maxPods value
# Then restart kubelet
sudo systemctl restart kubelet
```

## Next Steps

- [Core Concepts](../03-Core-Concepts/README.md)
- [Cheatsheet](../08-Cheatsheet/README.md)
