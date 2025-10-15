# 01 - Host Setup

## Objective
Prepare host for Kubernetes control-plane or worker installation.

## System Requirements
- Ubuntu 22.04 LTS (minimal)
- 2+ vCPU, 2GB+ RAM
- Root or sudo access
- Static IP configured

## Steps

### 1. Update system
```
apt update && apt upgrade -y
apt install -y curl wget vim net-tools gnupg2 software-properties-common apt-transport-https ca-certificates
```

### 2. Disable swap
```
swapoff -a
sed -i '/ swap / s/^/#/' /etc/fstab
```

### 3. Kernel modules and sysctl
```
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sysctl --system
```

### 4. Install containerd
```
mkdir -p /etc/containerd
apt install -y containerd
containerd config default | tee /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
systemctl restart containerd
systemctl enable containerd
```

### 5. Install Kubernetes components
```
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" | tee /etc/apt/sources.list.d/kubernetes.list
apt update
apt install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

### 6. Verify binaries
```
kubeadm version
kubectl version --client
containerd --version
```

### 7. Reboot (recommended)
```
reboot
```
