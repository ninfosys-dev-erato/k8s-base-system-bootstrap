# 04 - Local Path Storage

## Objective
Enable dynamic PersistentVolume provisioning using Local Path Provisioner.

## Steps

### 1. Verify Helm installed
```bash
helm version
```

### 2. Add Rancher chart repo
```
helm repo add rancher-lpp https://rancher.github.io/local-path-provisioner
helm repo update
```

### 3. Create namespace
```
kubectl create namespace local-path-storage
```

### 4. Install chart with repo-managed values file
```
helm install local-path-provisioner rancher-lpp/local-path-provisioner \
  -n local-path-storage \
  -f /etc/system-bootstrap/local-path-storage/local-path-values.yaml
```

### 5. Confirm storage class
```
kubectl get storageclass
```

Expected output:
```
NAME                   PROVISIONER                     DEFAULT
local-path (default)   rancher.io/local-path           true
```

### 6. Test provisioning
```
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/examples/pvc.yaml
kubectl get pvc
kubectl delete pvc local-path-pvc
```
