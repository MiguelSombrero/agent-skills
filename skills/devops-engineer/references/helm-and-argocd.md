# Helm and Argo CD

Upstream: [Helm](https://helm.sh/docs/), [Argo CD Applications](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/), [AppProject](https://argo-cd.readthedocs.io/en/stable/user-guide/projects/).

## Helm (application charts)

- **Scaffold** with `helm create`—do not store generated template dumps in this repo; customize chart version, `values.yaml`, and templates in your application repo.
- **Chart.yaml:** `version` (chart) vs `appVersion` (app)—keep both meaningful for rollbacks and communication.
- **Values:** per-environment files (`values-prod.yaml`) or CI `--set` for image tag only—team convention matters.
- **Workflow:** `helm lint`, `helm template` / `helm install --dry-run --debug`, then `helm upgrade --install`; `helm history` and `helm rollback` for recovery.
- **Dependencies:** declare subcharts in `Chart.yaml` when you embed databases/redis—prefer shared platform charts for infra when appropriate.

## When Helm vs plain YAML

- **Helm:** multiple services/envs, shared templates, chart reuse.
- **Kustomize or static YAML:** small footprint, fewer moving parts—both are valid; GitOps works with either.

## Argo CD (application GitOps)

- **Application:** points to a **repo URL**, **path**, **targetRevision** (branch/tag/commit), and **destination** cluster/namespace.
- **Sync policy:** automated sync + **prune** + **selfHeal** trade automation against surprise deletes—configure to team risk tolerance.
- **AppProject:** restricts **which repos** and **which clusters/namespaces** an app may use—use for tenant boundaries.
- **ignoreDifferences:** e.g. ignore `/spec/replicas` when **HPA** owns replica count—field paths in Argo CD docs.
- **Sync waves / hooks:** order resources when needed (DB before app); see Argo CD docs for annotations.

**Bootstrap:** Installing the Argo CD control plane may be platform work; **registering Applications** for your microservices is DevOps—align with **cloud-platform-engineer** if the cluster is shared.
