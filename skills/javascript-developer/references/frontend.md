# React, data fetching, Next.js

Reference for [SKILL.md](../SKILL.md).

---

## Structure

- **Feature folders:** `features/<area>/components`, `hooks`, `api` or `services`—avoid a single giant `components/` dump when the app grows.
- **Presentational vs container:** Keep dumb UI separate from data loading when it improves reuse and testing.

## TanStack Query

- **`queryKey`:** Include every variable that changes the result (e.g. `['users', page]`).
- **`queryFn`:** Call your API client; handle HTTP errors in one place (interceptor or wrapper).
- **Mutations:** `onSuccess` → `invalidateQueries` for affected lists/detail keys.

## Forms

- **react-hook-form** + **Zod** (`@hookform/resolvers/zod`) keeps client validation aligned with server schemas where possible.
- Surface server errors (field + global) and loading/disabled state on submit.

## Next.js (App Router)

- **Route Handlers** (`route.ts`): `GET`/`POST` like Express—validate body with Zod, call services, return `NextResponse.json`.
- **Server Components:** Default—async data fetch on server; no browser-only APIs.
- **Client components:** `'use client'` only where you need hooks, events, or browser APIs; pass serializable props from RSC.
- **Env:** Server secrets vs `NEXT_PUBLIC_*`—never expose server-only keys to the client.

## Accessibility (baseline)

- Associate **labels** with inputs; use **semantic elements** and **`getByRole`**-friendly patterns in tests.
- Keyboard: focus order, Escape for dialogs, don’t trap focus without intent.
- For visual design systems and deep UX patterns, see **ui-ux-designer** skill when relevant.
