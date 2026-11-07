# Repository Guidelines

## Project Structure

This repository contains Kubernetes deployment configuration for a self-hosted OmniRoute instance. The manifest is `k8s/omniroute-deployment.yaml`; it defines the Service, Deployment, and Ingress. Persistent application data is mounted at `/app/data` from the configured host path. Keep deployment changes in this manifest unless the project grows additional components.

## Deploy and Verify

Use the local kind context explicitly when managing the development cluster:

```sh
kubectl --context kind-mac-studio apply -f k8s/omniroute-deployment.yaml
kubectl --context kind-mac-studio rollout status deployment/omniroute
kubectl --context kind-mac-studio get pods -l app=omniroute
```

Check container logs with `kubectl --context kind-mac-studio logs deployment/omniroute`. The startup and readiness probes use `/healthz`; verify a rollout reaches Ready before considering a deployment complete. Test the endpoint from inside the pod when needed:

```sh
kubectl --context kind-mac-studio exec deployment/omniroute -- node -e 'fetch("http://127.0.0.1:20128/healthz").then(r => { console.log(r.status); process.exit(r.ok ? 0 : 1) })'
```

## Manifest Conventions

Use two spaces for YAML indentation. Keep the Service selector, Deployment selector, and pod label aligned on `app: omniroute`. Keep the Service target port aligned with the container port (`20128`) and the Ingress backend aligned with the Service port (`8080`). Pin container images to explicit release tags, such as `diegosouzapw/omniroute:3.8.53`, rather than `latest`.

Review resource requests and limits against expected workload and node capacity. Treat changes to `/app/data`, `hostPath`, probes, and resource limits as operational changes: check their impact on persistence, scheduling, and recovery before applying them.

## Testing and Changes

There is no application source code or automated test suite in this repository. Validate YAML parsing and use Kubernetes server-side dry-run when the cluster is available. For deployment changes, apply the manifest and verify rollout status, pod events, and `/healthz`.

Use concise commits with conventional prefixes, for example `chore: update OmniRoute image`. In pull requests, describe the operational effect, list validation performed, and call out data or downtime implications.
