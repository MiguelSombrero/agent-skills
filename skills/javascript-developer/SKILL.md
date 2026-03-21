---
name: javascript-developer
description: Senior TypeScript/JavaScript engineer for full-stack web apps. Covers React, Node/Express, Next.js App Router, Zod validation, Prisma, TanStack Query, Vitest, React Testing Library, Playwright, layered/hex-style structure, JWT auth, and web security basics. Use when building or refactoring TS/JS APIs, React UIs, Next.js routes, tests, or data access—or when the user mentions Vite, Vitest, Zod, Prisma, Express, Playwright, REST, hooks, or Server Components.
---

# JavaScript Developer

TypeScript-first full-stack work: clear boundaries, validation at the edge, tests that prove behavior. The YAML `description` lists when to apply this skill.

## Scope

**In scope:** React (hooks, composition, client vs server boundaries), Node/Express APIs, Next.js (Route Handlers, Server Components, `'use client'`), Zod (or project validator), Prisma or similar ORM, TanStack Query, Vitest/Jest, RTL, Playwright, auth (JWT and cookie/session patterns), performance and error handling.

**Out of scope:** Mobile native, games, Electron, pre-ES6 legacy stacks. For CI/GitOps use devops-engineer; for deep visual/UX systems use ui-ux-designer.

**Stack:** Follow the repo’s versions (Node, React, Next, TypeScript). Default posture: `strict` TypeScript, async/await, validate untrusted input at boundaries.

## Core principles

- **SOLID:** Thin HTTP layer; application services orchestrate; domain holds invariants; depend on interfaces (ports) in domain/application, implement in infrastructure.
- **TDD:** Red → green → refactor; Arrange–Act–Assert; prefer fast unit tests for pure logic, fewer integration tests for HTTP/DB, minimal E2E for critical paths.
- **Domain:** Behavior on entities/value objects; factory/static constructors; reject invalid state inside the model—not only in controllers.

## Architecture

Dependencies point **inward**: presentation → application → domain ← infrastructure.

```
┌─────────────────────────────────────────┐
│  Presentation  routes, handlers, DTOs     │
├─────────────────────────────────────────┤
│  Application   use cases, services      │
├─────────────────────────────────────────┤
│  Domain        entities, VOs, ports     │
├─────────────────────────────────────────┤
│  Infrastructure DB, HTTP clients, impl  │
└─────────────────────────────────────────┘
```

Typical layout (adapt to the project): `presentation/` / `app/` for HTTP UI and API; `application/` for use cases; `domain/` for models + repository interfaces; `infrastructure/` for Prisma/adapters; `shared/` for errors and utilities.

## Which reference to open

| Task | File |
|------|------|
| Express routes, Zod, errors, domain entities, Prisma repositories | [references/rest-and-data.md](references/rest-and-data.md) |
| JWT, cookies, XSS/CSRF, secrets | [references/security.md](references/security.md) |
| Vitest, RTL, Playwright | [references/testing.md](references/testing.md) |
| React, TanStack Query, forms, Next.js | [references/frontend.md](references/frontend.md) |
| Perf, anti-patterns, checklist | [references/operations.md](references/operations.md) |

**Cross-stack testing mindset:** [../test-engineer/SKILL.md](../test-engineer/SKILL.md)

Read references **on demand** by task—do not load all of them for every question.
