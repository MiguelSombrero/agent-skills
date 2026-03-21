# Performance, pitfalls, checklist

Reference for [SKILL.md](../SKILL.md).

---

## Performance

- **Caching:** Prefer Redis or your platform’s cache for multi-instance; in-memory `Map` TTL only acceptable for single-node/dev—invalidate on writes that affect the key.
- **DB:** Avoid **N+1**—use `include`/`select` (Prisma) or joins; add indexes for real query patterns; measure before micro-optimizing.
- **Frontend:** Code-split routes/heavy panels; lazy-load large client-only modules; images: correct sizing and modern formats per stack.

## Pitfalls

- **Callback pyramids:** Use `async`/`await`; centralize error handling.
- **Prop drilling:** Context, composition, or colocated state—not six layers of the same props.
- **Anemic domain:** If every rule lives in services, entities are structs—move invariants into the model where it fits the product.

## Checklist

**Architecture**

- [ ] Clear layers; domain does not import infrastructure
- [ ] Ports (interfaces) at boundaries you swap (DB, email, HTTP clients)

**API**

- [ ] Correct HTTP verbs/status codes; pagination for lists
- [ ] Validate input at the edge; consistent error JSON

**Frontend**

- [ ] Loading and error UI for async flows
- [ ] Forms validated client + server

**Testing**

- [ ] Most coverage from fast unit tests; integration for HTTP/DB; E2E for critical journeys only
- [ ] Tests independent (no shared mutable state/order dependence)

**TypeScript**

- [ ] `strict`; avoid `any`; types for API responses where practical

**Security / ops**

- [ ] Secrets in env; no tokens in logs; dependencies reviewed on a schedule
