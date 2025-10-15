# 03 - Worker Node Join

## Objective
Join worker nodes to the control plane using the generated token and verify node readiness.

## Steps

### 1. Copy join command from control plane
On the control-plane node:
```bash
cat /etc/system-bootstrap/join-command.sh
```
### 2. Run join command on each worker node
Execute the printed command on every worker node:
```
kubeadm join <CONTROL_PLANE_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```
If nodes need to reset before joining again:
```
kubeadm reset -f
rm -rf /etc/cni/net.d
```

### 3. Validate join success
On the control-plane node:
```
kubectl get nodes -o wide
```

Expected output:
```
NAME              STATUS   ROLES           AGE   VERSION
k8s-control       Ready    control-plane   10m   v1.30.x
k8s-worker-1      Ready    <none>          1m    v1.30.x
k8s-worker-2      Ready    <none>          1m    v1.30.x
```

### 4. Test pod scheduling
```
kubectl run test-pod --image=nginx --restart=Never
kubectl get pods -o wide
kubectl delete pod test-pod
```

### 5. Optional: label worker nodes
```
kubectl label node <worker-node-name> node-role.kubernetes.io/worker=worker
```
