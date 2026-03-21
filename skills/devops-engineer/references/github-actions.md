# GitHub Actions

Upstream: [Workflow syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions), [Contexts](https://docs.github.com/en/actions/learn-github-actions/contexts), [Reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows), [Security hardening](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions).

## Job design

- Run **lint/unit tests** before **Docker build** and registry push.
- Use **`needs:`** to chain jobs; use **`strategy.matrix`** for version matrices when it reduces duplication (official docs for syntax).
- Use **`workflow_call`** for repeated build/test patterns across repos or workflows.
- Prefer **environments** with protection rules for production.

## Caching

- Use **`actions/cache`** or **`setup-java`** / **`setup-node`** built-in caching keyed on lockfiles (`pom.xml` + wrapper hash, `package-lock.json`, etc.).
- For Docker, **GHA cache** with buildx (`cache-from` / `cache-to: type=gha`) is documented in Docker’s build-push action—avoid pasting full YAML here.

## Java / Maven (Spring-friendly)

- **`actions/setup-java`** with `distribution` (e.g. `temurin`) and `java-version` (`25` or project standard).
- Cache: `cache: maven` or explicit `~/.m2` keyed on `**/pom.xml` + wrapper.
- Verify: `mvn -B verify` (add `-DskipITs` only if the team agrees).
- **Do not** treat Node `npm ci` samples as defaults for this org’s Java services.

## OIDC (keyless cloud / registry)

- Set job permissions: at minimum `id-token: write` (and `contents: read` for checkout) when using **`aws-actions/configure-aws-credentials`**, **`google-github-actions/auth`**, or **`azure/login`** with OIDC—see each action’s README for the exact `permissions` block.
- Prefer **federated identity** over storing long-lived cloud keys in GitHub Secrets when the cloud account supports it.

## Pin third-party actions

- For supply-chain hygiene, pin **`uses:`** to a **full commit SHA** (not only `@v4` tags) on security-sensitive workflows; keep tag in a comment if helpful.
- Review **allowed actions** in org settings if applicable.

## Supply chain extras

- **SBOM:** Generate when compliance or production policy requires (e.g. Syft, CycloneDX); store as build artifact or attach to release—tooling changes often; follow current vendor docs.
- **SAST / CodeQL / Trivy:** Integrate per org standards; avoid duplicating vendor YAML here.

## Conditional runs

- Use `if:` with `github.ref`, `github.event_name`, or `github.event.pull_request.draft` to skip deploys from the wrong branch or draft PRs—syntax in GitHub docs.
