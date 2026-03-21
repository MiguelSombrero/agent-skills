# k3d and secrets

Upstream: [k3d](https://k3d.io/), [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/).

## k3d (local Kubernetes)

- **Create / stop / delete:** `k3d cluster create`, `cluster stop`, `cluster start`, `cluster delete`—see CLI help.
- **Registry:** often useful to attach a **local registry** and push images there, then reference `registry.host:port/image:tag` in manifests or Helm values—exact flags in k3d docs (versioned).
- **kubeconfig:** k3d updates kubeconfig by default; use `--kubeconfig-switch` behavior from docs if you manage multiple clusters.
- **Deeper cluster tuning** (CNI, multi-server, platform components): see **cloud-platform-engineer**.

## Secrets in CI

- **GitHub Secrets** for tokens needed in workflows; reference via `${{ secrets.NAME }}`—never echo.
- Prefer **OIDC** to cloud/registry over static keys when supported ([github-actions.md](github-actions.md)).

## Secrets in GitOps

- **Sealed Secrets:** encrypt Secret manifests for a given cluster controller; commit sealed resources; cluster decrypts. Install controller and `kubeseal` CLI per [Bitnami Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets).
- **External Secrets Operator:** sync from Vault, AWS Secrets Manager, etc., into native Secrets—see [External Secrets](https://external-secrets.io/); often platform-owned SecretStore setup.

Pick one pattern per cluster and team—don’t mix unencrypted Secrets in Git with sealed/ESO without a migration plan.
