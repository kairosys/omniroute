<h1 align="center">OmniRoute Kubernetes Deployment</h1>
<p align="center">
  <strong>Kubernetes manifests for running OmniRoute in a local kind cluster.</strong>
  <br />
  <em>OmniRoute v3.8.53 · Kubernetes · kind</em>
</p>

<p align="center">
  <a href="#quick-start"><img src="https://img.shields.io/badge/Quick_Start-4CAF50?style=for-the-badge" alt="Quick Start" /></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge" alt="MIT License" /></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white" alt="Kubernetes" />
  <img src="https://img.shields.io/badge/kind-326CE5?style=flat&logo=kubernetes&logoColor=white" alt="kind" />
</p>

## Features

- Deploys OmniRoute as a single-replica Kubernetes Deployment.
- Exposes the application through a ClusterIP Service and nginx Ingress.
- Uses `/healthz` for startup and readiness probes.
- Mounts `/app/data` for application data persistence.

## Quick Start

The commands below target the configured local cluster context, `kind-mac-studio`.

```sh
kubectl --context kind-mac-studio apply -f k8s/omniroute-deployment.yaml
kubectl --context kind-mac-studio rollout status deployment/omniroute
kubectl --context kind-mac-studio get pods -l app=omniroute
```

The Ingress uses host `omniroute.localhost` and requires an nginx Ingress controller.

## Architecture

```mermaid
flowchart LR
    Client[Client<br/>omniroute.localhost] --> Ingress[nginx Ingress]
    Ingress --> Service[ClusterIP Service<br/>port 8080]
    Service --> App[OmniRoute v3.8.53<br/>container port 20128]
    App --> Data[(Persistent hostPath<br/>/app/data)]
    classDef edge fill:#e8f0fe,stroke:#326ce5,color:#172554
    classDef app fill:#ede9fe,stroke:#7c3aed,color:#2e1065
    classDef store fill:#ecfdf5,stroke:#059669,color:#064e3b
    class Client,Ingress,Service edge
    class App app
    class Data store
```

## Configuration

| Setting | Value | Purpose |
|---|---|---|
| Image | `diegosouzapw/omniroute:3.8.53` | OmniRoute release |
| Container port | `20128` | Application HTTP port |
| Service port | `8080` | In-cluster access |
| Health endpoint | `/healthz` | Startup and readiness checks |
| `OMNIROUTE_MEMORY_MB` | `8192` | Node.js heap setting in MB |
| Memory request / limit | `5Gi` / `10Gi` | Kubernetes scheduling and memory ceiling |
| Data mount | `/mnt/workspaces/omniroute/data` → `/app/data` | Application data directory |
| Ingress host | `omniroute.localhost` | Local HTTP hostname |

Review the hostPath for the target Kubernetes node before using this manifest outside the configured local kind environment.

## Project Structure

```text
.
├── AGENTS.md                       # Contributor and repository guidance
├── k8s/
│   └── omniroute-deployment.yaml   # Service, Deployment, and Ingress
└── data/                           # Local data directory (gitignored)
```

## Operations

Inspect application logs:

```sh
kubectl --context kind-mac-studio logs deployment/omniroute
```

Check the health endpoint from inside the pod:

```sh
kubectl --context kind-mac-studio exec deployment/omniroute -- \
  node -e 'fetch("http://127.0.0.1:20128/healthz").then(r => { console.log(r.status); process.exit(r.ok ? 0 : 1) })'
```

## Contributing

Update the Kubernetes manifest, validate it with `kubectl apply --dry-run=server -f k8s/omniroute-deployment.yaml` when the cluster is available, and verify the rollout and health endpoint after deployment. Use a concise commit message such as `chore: update OmniRoute image`.

This project is licensed under the [MIT License](LICENSE).
