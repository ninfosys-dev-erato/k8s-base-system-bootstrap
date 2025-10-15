# 09 - Security and Hardening

## Objective

Apply minimal, practical Kubernetes and host-level hardening.

### 1. OS-level hardening
```
ufw enable
ufw allow 22/tcp
ufw allow 6443,10250,10255,30000:32767/tcp
ufw status verbose
```

Disable password login:
```
sed -i 's/^#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
systemctl restart ssh
```

### 2. Kubelet and API access restrictions
Edit `/var/lib/kubelet/config.yaml`:
```
authentication:
  anonymous:
    enabled: false
authorization:
  mode: Webhook
```
Then:
```
systemctl restart kubelet
```

### 3. RBAC and namespace isolation
Create least-privileged roles:
```
kubectl create role readonly --verb=get,list,watch --resource=pods,services
kubectl create rolebinding readonly --role=readonly --user=<user> -n <namespace>
```

### 4. Network policies
```
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
EOF
```

### 5. Traefik security

- Disable dashboard in production or protect via auth middleware.
- Use Let’s Encrypt DNS or HTTP challenge only.
- Ensure /data/acme.json has 600 permissions.

### 6. Secrets management
```
kubectl create secret generic example-secret --from-literal=key=value
```
Store sensitive data externally (Vault, SOPS, etc).

### 7. Regular patching
```
apt update && apt upgrade -y
kubectl version --short
```
