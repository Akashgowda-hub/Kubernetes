# Header-based Ingress Routing

Header-based routing sends traffic based on HTTP headers.

In ingress-nginx, a practical way is header-match annotations.

Example:

- Default traffic -> `web-main-service`
- Requests with header `x-route: tools` -> `web-tools-service`

## Mandatory fields for header-based ingress

Base ingress required fields:

- `apiVersion: networking.k8s.io/v1`
- `kind: Ingress`
- `metadata.name`
- `spec.ingressClassName`
- `spec.rules[].http.paths[].backend.service`

Header-match ingress additional required annotations in ingress-nginx:

- `nginx.ingress.kubernetes.io/canary: "true"`
- One header selector, for example:
  - `nginx.ingress.kubernetes.io/canary-by-header: "x-route"`
  - `nginx.ingress.kubernetes.io/canary-by-header-value: "tools"`

## YAML difference: path-based vs header-based

Path-based:

- Decision logic in `spec.rules[].http.paths[]` (`path`, `pathType`).
- No controller-specific annotation required for basic routing.

Header-based:

- Usually same host/path but adds controller-specific annotations in `metadata.annotations`.
- Requires a base ingress plus a header-match ingress rule.

## Files in this folder

- `Examples/demo-services.yml` - nginx backend services/deployments
- `Examples/header-routing-by-header.yml` - base + header-match ingress resources

## Apply steps

```bash
kubectl apply -f Examples/demo-services.yml
kubectl apply -f Examples/header-routing-by-header.yml
kubectl get ingress
```

## Test examples

```bash
# Default route -> main
curl -H "Host: app.akashshankar.in" http://<INGRESS_IP>/

# Header route -> tools
curl -H "Host: app.akashshankar.in" -H "x-route: tools" http://<INGRESS_IP>/
```
