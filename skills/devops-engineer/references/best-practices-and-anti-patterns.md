# Best practices and anti-patterns

Deployment mechanics → [deployment-and-monitoring.md](deployment-and-monitoring.md). Helm and Argo CD → [helm-and-argocd.md](helm-and-argocd.md). CI details → [github-actions.md](github-actions.md).

## Do

- Order pipeline stages so **cheap failures happen first**.
- **Parameterize** images, tags, and replica counts (Helm values, Kustomize overlays, or CI env).
- Set **resource requests/limits** and meaningful **probes** for production workloads.
- **Plan rollback:** Helm revision, Git revert + sync, or `kubectl rollout undo`—document which the team uses.

## Avoid

- **Secrets in plain Git** (use Sealed Secrets, External Secrets, or secret manager integration).
- **Root in container** and **missing limits** on shared clusters.
- **Imperative prod changes** that GitOps will overwrite—fix in Git or pause sync deliberately.
- **Copy-paste YAML** from tutorials without matching your ingress class, namespace, and platform constraints.
