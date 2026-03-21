# Docker and Kubernetes (application workloads)

Upstream: [Dockerfile reference](https://docs.docker.com/reference/dockerfile/), [Kubernetes concepts](https://kubernetes.io/docs/concepts/), [Workload resources](https://kubernetes.io/docs/concepts/workloads/).

## Images

- **Multi-stage builds:** separate compile/build from runtime; final stage minimal base (distroless or slim) when feasible.
- **Pin base images** by digest or specific tag—avoid unqualified `latest` in production.
- **Non-root** user in the image; document writable paths (`/tmp`, app-specific) if you use read-only root FS.
- **`.dockerignore`** to keep build context small and secrets out of layers.
- **Health:** align container `HEALTHCHECK` (if used) with K8s probes—see below.

## Kubernetes (apps)

- **Deployment:** set **`strategy`** (usually `RollingUpdate`) and tune `maxSurge` / `maxUnavailable` for your SLO; defaults are in workload docs.
- **Probes:** `readiness` gates traffic; `liveness` restarts—don’t point both at the same shallow check if it causes flapping under load.
- **Resources:** always set **requests**; set **limits** unless the team explicitly runs BestEffort (rare for prod).
- **Config:** ConfigMap for non-secret config; **Secrets** for credentials—never commit raw Secret YAML to Git.
- **Service / Ingress / Gateway:** expose via the mechanism your **platform** provides (Ingress vs Gateway API); ingress annotations are controller-specific—see platform skill or controller docs.

## When to use plain manifests vs Helm

- Few static YAML files: OK for small services.
- Multiple envs and shared patterns: **Helm** or Kustomize—Helm details in [helm-and-argocd.md](helm-and-argocd.md).
