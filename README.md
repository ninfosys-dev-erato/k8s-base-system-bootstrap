# System Bootstrap

## Overview
`system-bootstrap` provides a fully reproducible, version-controlled automation flow for provisioning, securing, and maintaining a self-hosted Kubernetes cluster with Traefik ingress and NGINX edge routing.

All files under `/etc/system-bootstrap` map directly to infrastructure layers and can be applied sequentially or selectively via automation.

---

## Directory Structure
```

/etc/system-bootstrap/
├── base-config.yaml                 # Core OS/network bootstrap
├── container-runtime.yaml           # Containerd + system dependencies
├── kubeadm-control-plane-config.yaml# Control-plane init configuration
├── local-path-storage/              # Local path storage provisioner
├── traefik/                         # Ingress controller values
├── whoami/                          # Example app for validation
└── docs/                            # Full procedural documentation

````

---

## Documentation Index
| Step | File | Description |
|------|------|--------------|
| 00 | [00-overview.md](docs/00-overview.md) | Architecture summary |
| 01 | [01-host-setup.md](docs/01-host-setup.md) | Base OS configuration |
| 02 | [02-control-plane.md](docs/02-control-plane.md) | Control plane bootstrap |
| 03 | [03-worker-join.md](docs/03-worker-join.md) | Worker node registration |
| 04 | [04-addons.md](docs/04-addons.md) | Addons via Helm (CNI, metrics, storage) |
| 05 | [05-traefik-ingress.md](docs/05-traefik-ingress.md) | Traefik ingress setup + Let’s Encrypt |
| 06 | [06-sample-app.md](docs/06-sample-app.md) | Sample “whoami” deployment |
| 07 | [07-nginx-lb.md](docs/07-nginx-lb.md) | External NGINX load balancer |
| 08 | [08-validation.md](docs/08-validation.md) | Validation and smoke testing |
| 09 | [09-hardening.md](docs/09-hardening.md) | Security hardening |
| 10 | [10-routing-diagnostics.md](docs/10-routing-diagnostics.md) | Routing and diagnostics |
| 11 | maintenance.md (planned) | Upgrade and lifecycle ops |

---

## Usage
Clone into `/etc/system-bootstrap` on all hosts:
```bash
git clone <repo-url> /etc/system-bootstrap
cd /etc/system-bootstrap
````

Follow the docs sequentially from `00` to `10`.
Each file defines **repeatable, idempotent commands**.
Store environment-specific overrides in `/etc/system-bootstrap/env/`.

---

## Operational Principles

* Everything is **GitOps-aligned**.
* Each `.yaml` is a **single source of truth** for one system layer.
* Every change is traceable and version-controlled.
* Docs are executable — every code block can run directly on target hosts.

---

## Maintenance

Regularly:

```bash
apt update && apt upgrade -y
helm repo update
kubectl get nodes -o wide
kubectl get pods -A
```

Cert renewals are automated via Traefik (DNS/HTTP ACME).
For Cloudflare DNS automation, update credentials in `traefik/values.yaml`.

---

## Authoring Notes

* Each new component gets its own subdirectory and `values.yaml`.
* Document every manual change in a matching `docs/NN-*.md` file.
* Use consistent naming: lower-case, hyphenated, declarative.

---

## License

Internal use only. Confidential infrastructure baseline.

