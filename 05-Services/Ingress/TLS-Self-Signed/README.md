# TLS and SSL Termination (Self-signed)

TLS termination at Ingress means client HTTPS is decrypted at Ingress Controller, then forwarded to backend service (usually HTTP in-cluster).

## Flow

```
┌──────────────┐   HTTPS 443   ┌──────────────────────┐   HTTP 80   ┌──────────────┐
│    Client    │ -------------> │ Ingress Controller   │ ----------> │ Service/Pods │
└──────────────┘                │ (TLS termination)    │             └──────────────┘
                                └──────────────────────┘
```

## Mandatory fields for TLS ingress

- `spec.tls[]`
- `spec.tls[].hosts[]`
- `spec.tls[].secretName`
- Matching host in `spec.rules[].host`
- TLS secret of type `kubernetes.io/tls`

## Step-by-step (self-signed)

### 1) Generate self-signed certificate

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key \
  -out tls.crt \
  -subj "/CN=secure.akashshankar.in/O=akashshankar.in"
```

### 2) Create TLS secret in Kubernetes

In Kubernetes, TLS certificate and private key are stored in a Secret object of type `kubernetes.io/tls`.

Mapping:

- `tls.crt` file -> stored as `data.tls.crt`
- `tls.key` file -> stored as `data.tls.key`

This Secret is stored in etcd (the Kubernetes data store), and Ingress reads it using `spec.tls[].secretName`.

```bash
kubectl create secret tls akashshankar-in-tls \
  --cert=tls.crt \
  --key=tls.key
```

### 2.1) Verify Secret is created and type is TLS

```bash
kubectl get secret akashshankar-in-tls
kubectl describe secret akashshankar-in-tls
kubectl get secret akashshankar-in-tls -o jsonpath='{.type}'
```

Expected type:

`kubernetes.io/tls`

### 2.2) Verify certificate and key entries exist in Secret

```bash
kubectl get secret akashshankar-in-tls -o jsonpath='{.data.tls\.crt}'
kubectl get secret akashshankar-in-tls -o jsonpath='{.data.tls\.key}'
```

Optional decode check:

```bash
kubectl get secret akashshankar-in-tls -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -subject -issuer -dates
```

### 3) Apply backend service and ingress

Ingress references the same secret name in `spec.tls[].secretName`.

```bash
kubectl apply -f Examples/demo-service.yml
kubectl apply -f Examples/ingress-tls-selfsigned.yml
kubectl get ingress
```

### 4) Map host to ingress IP

Linux/macOS:

```bash
sudo sh -c 'echo "<INGRESS_IP> secure.akashshankar.in" >> /etc/hosts'
```

Windows hosts file path:

`C:\Windows\System32\drivers\etc\hosts`

Add line:

`<INGRESS_IP> secure.akashshankar.in`

### 5) Test HTTPS

```bash
curl -k https://secure.akashshankar.in/
```

## Files in this folder

- `Examples/demo-service.yml`
- `Examples/ingress-tls-selfsigned.yml`
