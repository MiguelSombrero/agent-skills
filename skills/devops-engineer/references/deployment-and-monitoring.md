# Deployment strategies and delivery signals

Upstream: [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Argo Rollouts](https://argoproj.github.io/argo-rollouts/).

## Strategies (choose by risk and traffic shape)

- **Rolling update:** default for Deployments; tune surge/unavailable for zero-downtime vs speed.
- **Blue/green:** two full stacks; flip Service selector or equivalent—good when you need instant cutover and quick revert.
- **Canary:** subset of pods or traffic to new version—Kubernetes Service alone does not split percentages; you need extra mechanism (multiple Deployments + weights, mesh, ingress weighting, or **Argo Rollouts**).
- **Progressive delivery:** **Argo Rollouts** `Rollout` CRD with automated promotion/pause steps—install and operate per Rollouts docs; platform install may be **cloud-platform-engineer** territory if cluster-wide.

## Rollback

- **Helm:** `helm history` / `helm rollback`.
- **GitOps:** revert Git commit or previous revision; let Argo CD sync.
- **kubectl:** `kubectl rollout undo` for Deployments—know which the team standardizes on.

## CI/CD observability and notifications

- Track **pipeline duration** and **failure rate** (GitHub Actions insights, or export to your metrics stack)—avoid bespoke curl-to-unknown-endpoint examples.
- Notify deploy outcome (Slack, Teams, PagerDuty) via official actions or webhooks; configure channels outside this skill.
