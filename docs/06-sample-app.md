# 06 - Whoami Test Deployment

## Objective
Deploy a simple test service (`whoami`) to validate Traefik ingress, service routing, and TLS termination.

## Steps

### 1. Create namespace
```bash
kubectl create namespace whoami
```

### 2. Deploy from repo-managed manifest
```
kubectl apply -f /etc/system-bootstrap/whoami/whoami.yaml -n whoami
```

### 3. Verify deployment and service
```
kubectl get pods -n whoami
kubectl get svc -n whoami
```
Expected output:
```
NAME                      READY   STATUS    RESTARTS   AGE
whoami-cb4cdc4dd-g4d5v    1/1     Running   0          2m

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
whoami-svc   ClusterIP   10.96.203.178   <none>        80/TCP    2m
```

#### 4. Verify ingress
```
kubectl describe ingress whoami-ingress -n whoami
```

Expected:
```
Host: whoami.digitalepalika.com
Address: <external-ip>
TLS: whoami-tls
```


### 5. Validate endpoint reachability
Inside cluster:
```
kubectl run nettest --rm -it --image=appropriate/curl --restart=Never -- \
  curl -v http://whoami-svc.whoami.svc.cluster.local
```
Outside cluster (via Traefik NodePort or LB):
```
curl -vk https://whoami.digitalepalika.com
```
Expected response:
```
Hostname: whoami-cb4cdc4dd-g4d5v
IP: 127.0.0.1
```

### 6. Debug failures
If 502 Bad Gateway occurs:
```
kubectl logs -n traefik deploy/traefik | grep whoami
kubectl get endpoints whoami-svc -n whoami
```
If pod has no listener:
```
kubectl exec -n whoami -it <pod> -- wget -qO- localhost:80
```

### 7. Cleanup test
```
kubectl delete ns whoami
```
