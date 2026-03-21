# Security (web + auth)

Reference for [SKILL.md](../SKILL.md).

---

## JWT (Bearer) pattern

- **Issue:** After password verify, sign a payload (e.g. `sub`, `role`) with a server secret; set short-ish expiry; use HTTPS in production.
- **Verify:** Read `Authorization: Bearer <token>`, verify signature and expiry, attach claims to `req` (Express) or session context.
- **RBAC:** Check roles/permissions after authentication, on the route or handler.
- **Never** log raw tokens or put secrets in client bundles. Load signing secrets from env (e.g. `JWT_SECRET`).

## Cookie / session (when not using Bearer)

- Prefer **`httpOnly`**, **`Secure`**, **`SameSite`** appropriate to your CSRF strategy.
- **CSRF:** If using cookie sessions for APIs, use CSRF tokens or SameSite + framework defaults—do not assume “cookies are enough” for cross-site POSTs.

## OWASP-style web basics

- **XSS:** React escapes text by default; avoid `dangerouslySetInnerHTML` unless sanitized; treat all URL/query/body fields as untrusted.
- **Injection:** Parameterized DB queries (Prisma does this); never concatenate SQL/strings for execution.
- **Secrets:** Env vars on server; never commit `.env`; rotate on leak.
- **Rate limiting / abuse:** Apply on auth and sensitive routes (middleware, API gateway, or edge).
- **Dependencies:** Run `npm audit` / tooling your project uses; upgrade transitive risks per policy.

## When to go deeper

- OAuth2/OIDC flows, refresh tokens, and provider-specific hardening belong in project docs or a dedicated auth skill—keep implementations aligned with the chosen provider and legal requirements.
