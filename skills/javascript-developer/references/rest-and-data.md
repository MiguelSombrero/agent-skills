# REST API, domain, data

Reference for [SKILL.md](../SKILL.md).

---

## Contents

1. [Express HTTP layer](#express-http-layer)
2. [Validation (Zod)](#validation-zod)
3. [Errors](#errors)
4. [Domain model](#domain-model)
5. [Repositories + Prisma](#repositories--prisma)

---

## Express HTTP layer

- **Controllers / route handlers:** Parse params/query/body; call one application service method; map result to HTTP status + JSON; delegate failures via `next(error)` (or framework equivalent).
- **Routes:** Register middleware order: auth → validation → handler; keep route files thin—wire dependencies once (composition root / DI).

## Validation (Zod)

- Define **schemas** for body/query where input enters; `z.infer<typeof Schema>` for DTO types.
- Middleware: `parse`/`safeParse`; on `ZodError`, map to **400** with field-level messages—don’t leak stack traces to clients.

## Errors

- Typed hierarchy: e.g. validation (400), not found (404), conflict (409), unauthorized (401), forbidden (403); unknown → **500** with generic message, full detail in logs only.
- Central **error middleware** last in the chain; log `err`, `req.path`, correlation id if present.

## Domain model

- **Entities:** Private constructors + `static create` / `reconstitute` from persistence; methods enforce state transitions (`confirm`, `cancel`, etc.).
- **Value objects:** Immutable; validate in constructor (`Money`, `Email`).
- **No** Prisma types in domain—map at infrastructure edge.

## Repositories + Prisma

- **Interface** in domain (`UserRepository`): `findById`, `save`, etc.
- **Implementation** in infrastructure: Prisma calls; **`toDomain` / `toPersistence`** mappers; `include`/`select` to avoid N+1; pagination with `skip`/`take` + `count` when listing.

---

## One minimal vertical slice (mental model)

Not copy-paste boilerplate—shape to the project:

`POST /resource` → validate DTO → `service.create(dto)` → `repository.save(domainEntity)` → **201** + DTO response; errors bubble to global handler.
