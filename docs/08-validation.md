# 08 - Cluster Validation

## Objective
Verify that the control plane, networking, and workloads are functioning correctly after deployment.

---

### 1. Validate node readiness
```bash
kubectl get nodes -o wide
```
Expected: all nodes in Ready state.

### 2. Validate system pods
```
kubectl get pods -A -o wide
```
All pods under `kube-system`, `calico-system`, `traefik`, etc. should be Running.

### 3. Validate cluster networking
```
kubectl run nettest --rm -it --image=appropriate/curl --restart=Never -- sh
```
Inside:
```
ping 10.244.x.x
curl http://10.244.x.x:80
exit
```

### 4. Validate service routing
```
kubectl get svc -A
kubectl get endpoints -A
```
Check that service endpoints resolve to pod IPs.

### 5. Validate ingress controller
```
kubectl get ingress -A
kubectl describe ingress whoami-ingress -n whoami
```
Test ingress:
```
curl -vk https://whoami.digitalepalika.com
```

### 6. Optional: run conformance tests
```
sonobuoy run --wait
sonobuoy results $(sonobuoy retrieve)
```
