# Path-based Ingress Routing

Path-based routing sends traffic to different backends based on URL path.

Example:

- `app.akashshankar.in/api` -> `api-service`
- `app.akashshankar.in/` -> `web-service`

## Mandatory fields for path-based Ingress

These are required for practical path routing:

- `apiVersion: networking.k8s.io/v1`
- `kind: Ingress`
- `metadata.name`
- `spec.ingressClassName` (recommended and usually required operationally)
- `spec.rules[].host` (optional technically, recommended for production)
- `spec.rules[].http.paths[]`
- `paths[].path`
- `paths[].pathType`
- `paths[].backend.service.name`
- `paths[].backend.service.port.number` (or port name)

## pathType: Exact vs Prefix

- `Exact`: matches only full exact path.
- `Prefix`: matches path prefix and subpaths.

Examples:

- Request `/api`
  - `Exact /api` -> match
  - `Prefix /api` -> match

- Request `/api/v1/users`
  - `Exact /api` -> no match
  - `Prefix /api` -> match

## YAML differences: Exact vs Prefix

Main difference in YAML is `pathType` value and often number/order of path entries.

- Exact rule for strict endpoint routing.
- Prefix rule for app sections or full site routing.

## Files in this folder

- `Examples/path-routing-basic.yml` - host + `/api` and `/` path split
- `Examples/path-routing-exact-vs-prefix.yml` - side-by-side Exact and Prefix behavior
- `Examples/demo-services.yml` - nginx backend services/deployments

## Apply steps

```bash
kubectl apply -f Examples/demo-services.yml
kubectl apply -f Examples/path-routing-basic.yml
kubectl apply -f Examples/path-routing-exact-vs-prefix.yml
kubectl get ingress
```

## Test examples

```bash
curl -H "Host: app.akashshankar.in" http://<INGRESS_IP>/
curl -H "Host: app.akashshankar.in" http://<INGRESS_IP>/api
curl -H "Host: app.akashshankar.in" http://<INGRESS_IP>/api/v1/users
```
