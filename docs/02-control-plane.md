# 02 - Control Plane Bootstrap

## Objective
Initialize the control plane using the repo-managed kubeadm configuration.

## Steps

### 1. Update control-plane IP
Edit this file to match the node’s static IP:
```bash
vi /etc/system-bootstrap/kubeadm-control-plane-config.yaml
```

Ensure the following fields are correct:
```
controlPlaneEndpoint: "<CONTROL_PLANE_IP>:6443"
localAPIEndpoint:
  advertiseAddress: "<CONTROL_PLANE_IP>"
```

### 2. Initialize control plane
```
kubeadm init --config=/etc/system-bootstrap/kubeadm-control-plane-config.yaml
```

### 3. Configure kubectl
```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

### 4. Install Helm
```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

### 5. Add Calico Helm repo
```
helm repo add projectcalico https://projectcalico.docs.tigera.io/charts
helm repo update
```

### 6. Install Calico CNI
```
helm install calico projectcalico/tigera-operator \
  --namespace tigera-operator \
  --create-namespace
```

Wait for pods to be ready:
```
kubectl get pods -n calico-system
```

### 7. Verify cluster status
```
kubectl get nodes -o wide
kubectl get pods -A
```

### 8. Generate worker join command
```
kubeadm token create --print-join-command > /etc/system-bootstrap/join-command.sh
cat /etc/system-bootstrap/join-command.sh
```
Example output:
```
kubeadm join <CONTROL_PLANE_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```
