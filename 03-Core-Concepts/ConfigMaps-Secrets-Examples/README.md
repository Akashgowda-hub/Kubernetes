# ConfigMaps and Secrets - YAML Examples

This directory contains practical YAML examples for ConfigMaps and Secrets.

## Files Overview

- `configmap-examples.yml` - Various ConfigMap configurations
- `secret-examples.yml` - Various Secret types and usage patterns

## ConfigMap vs Secret Comparison

```
┌──────────────────────────────────────┐
│  ConfigMap                           │
├──────────────────────────────────────┤
│ • Store configuration data           │
│ • Non-sensitive values               │
│ • Up to 1MB per ConfigMap            │
│ • Base64 NOT encrypted               │
│ • Use for: Config files, settings    │
└──────────────────────────────────────┘

┌──────────────────────────────────────┐
│  Secret                              │
├──────────────────────────────────────┤
│ • Store sensitive data               │
│ • Passwords, tokens, keys            │
│ • Up to 1MB per Secret               │
│ • Base64 encoded (not encrypted)     │
│ • ETCD encryption recommended        │
│ • Use for: Credentials, certificates│
└──────────────────────────────────────┘
```

## ConfigMap Creation Methods

```
1. From literal key-value pairs
kubectl create configmap app-config \
  --from-literal=db.host=postgres \
  --from-literal=db.port=5432

2. From file
kubectl create configmap app-config \
  --from-file=config.yaml

3. From directory
kubectl create configmap app-config \
  --from-file=./config/

4. From YAML manifest
kubectl apply -f configmap.yaml
```

## Secret Creation Methods

```
1. From literal
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123

2. From file
kubectl create secret generic db-secret \
  --from-file=./password.txt

3. Docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass

4. TLS secret
kubectl create secret tls tls-secret \
  --cert=path/to/cert.pem \
  --key=path/to/key.pem

5. From YAML
kubectl apply -f secret.yaml
```

## Consuming ConfigMaps

```yaml
# Method 1: Environment variables
env:
- name: DB_HOST
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: db.host

# Method 2: Volume mount
volumeMounts:
- name: config
  mountPath: /etc/config

volumes:
- name: config
  configMap:
    name: app-config

# Method 3: All keys as env vars
envFrom:
- configMapRef:
    name: app-config
```

## Consuming Secrets

```yaml
# Method 1: Environment variables
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password

# Method 2: Volume mount
volumeMounts:
- name: secrets
  mountPath: /etc/secrets
  readOnly: true

volumes:
- name: secrets
  secret:
    secretName: db-secret

# Method 3: All keys as env vars
envFrom:
- secretRef:
    name: db-secret

# Method 4: Image pull secret
imagePullSecrets:
- name: regcred
```

## Quick Commands

```bash
# Apply ConfigMap
kubectl apply -f configmap-examples.yml

# Apply Secrets
kubectl apply -f secret-examples.yml

# View ConfigMap
kubectl get configmap
kubectl describe configmap <name>

# View Secret (values are base64 encoded)
kubectl get secret
kubectl describe secret <name>

# Decode secret value
kubectl get secret <name> -o jsonpath='{.data.password}' | base64 -d

# Delete ConfigMap/Secret
kubectl delete configmap <name>
kubectl delete secret <name>
```

See individual YAML files for detailed examples.
