# 00 - Overview

## Purpose
End-to-end reproducible Kubernetes bootstrap for bare-metal or VM nodes.

## Components
- containerd
- kubeadm + kubelet + kubectl
- Calico CNI
- local-path-storage
- Traefik ingress with ACME
- Example app: whoami
- External LB (nginx)

## Directory layout
/etc/system-bootstrap
├── base-config.yaml
├── container-runtime.yaml
├── kubeadm-control-plane-config.yaml
├── local-path-storage/local-path-values.yaml
├── traefik/values.yaml
├── whoami/whoami.yaml
└── docs/

## Workflow
1. Host setup  
2. Control plane init  
3. Worker join  
4. Add-ons  
5. Traefik ingress  
6. Sample app  
7. External LB  
8. Validation  
9. Hardening  
