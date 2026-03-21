# Testing

Reference for [SKILL.md](../SKILL.md).

---

## Unit (Vitest or Jest)

- Mock **ports** (repositories, clock, email) with `vi.fn()` / jest mocks; keep domain tests free of DB/network.
- Structure: **Arrange** data and mocks → **Act** one call → **Assert** return value and that collaborators were called correctly.
- Test domain rules (invalid transitions, invariants) directly on entities/value objects.

## Integration (HTTP + DB)

- Boot the real app (or minimal app factory); use **supertest** (or `fetch` against a test server) for `POST`/`GET`/`PATCH`/`DELETE`.
- Reset DB or use transactions per test so tests don’t depend on order; use project’s test DB URL.
- Assert status code, JSON shape, and side effects (row exists) when relevant.

## React (Testing Library)

- Prefer **`getByRole`**, **`getByLabelText`** over test IDs except for non-accessible widgets.
- Use **`userEvent`** over `fireEvent` for realistic interaction.
- Wrap components that need **TanStack Query** in a `QueryClientProvider` with `retry: false` in tests.

## E2E (Playwright)

- **Locators:** Prefer `getByRole`, `getByLabel`, `getByText` over CSS when possible; `data-testid` for stable hooks where roles are awkward.
- **Assertions:** `await expect(locator).toBeVisible()`; for counts use `await expect(locator).toHaveCount(n)` or `expect(await locator.count()).toBeGreaterThan(0)`—do not chain invalid matchers.
- Keep E2E flows short and focused on user-critical paths; avoid duplicating coverage already guaranteed by unit tests.

## Pyramid (rule of thumb)

- Majority **unit** (fast, cheap), some **integration** (HTTP/DB), few **E2E** (slow, flaky if overused). Adjust to team policy.
