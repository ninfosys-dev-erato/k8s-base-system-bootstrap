# 05 - Traefik Ingress Controller

## Objective
Deploy Traefik as the cluster ingress controller with automatic Let's Encrypt certificate management.

## Steps

### 1. Add Helm repo
```bash
helm repo add traefik https://traefik.github.io/charts
helm repo update
```

### 2. Create namespace
```
kubectl create namespace traefik
```

### 3. Install Traefik with repo-managed values
```
helm install traefik traefik/traefik \
  -n traefik \
  -f /etc/system-bootstrap/traefik/values.yaml
```

### 4. Confirm deployment
```
kubectl get pods -n traefik
kubectl get svc -n traefik
```

Expected output:
```
NAME                        READY   STATUS    RESTARTS   AGE
traefik-6b587bdc8b-lqbnt    1/1     Running   0          2m
```

### 5. Verify entrypoints and resolvers
```
kubectl logs -n traefik deploy/traefik | grep acme
```
You should see lines like:
```
Certificates obtained for domains [example.domain.com]
```

### 6. Configure persistence (optional)
```
persistence:
  enabled: true
  name: data
  accessMode: ReadWriteOnce
  size: 1Gi
```

### 7. Reload Traefik after cert fix
```
kubectl rollout restart deploy/traefik -n traefik
```

### 8. Confirm dashboard access (optional)
Expose dashboard via port-forward:
```
kubectl port-forward -n traefik deploy/traefik 9000:9000
```

Then open:
```
http://localhost:9000/dashboard/
```
