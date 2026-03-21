# UI performance

User-perceived speed and **Core Web Vitals** (field metrics). **Instrumentation and framework APIs** live in app docs and the **JavaScript/TypeScript** skill.

## Core Web Vitals (Google)

Official overview: [Web Vitals](https://web.dev/articles/vitals).

| Metric | What it reflects (simplified) | “Good” (field, 75th percentile) — see web.dev for current thresholds |
|--------|-------------------------------|----------------------------------------------------------------------|
| **LCP** — Largest Contentful Paint | Main content visible | Typically **≤ 2.5 s** |
| **INP** — Interaction to Next Paint | Responsiveness to interactions (replaced FID as a CWV) | Typically **≤ 200 ms** |
| **CLS** — Cumulative Layout Shift | Visual stability | Typically **≤ 0.1** |

- **TTFB** (Time to First Byte) is useful for server/CDN diagnosis but not a Core Web Vital—see [Optimize TTFB](https://web.dev/articles/ttfb).

**Note:** **FID** (First Input Delay) was deprecated in favor of **INP** for Core Web Vitals; prefer INP in guidance ([INP](https://web.dev/articles/inp)).

## Practical techniques (names only)

- **Code splitting** — route-level and component-level lazy loading; smaller initial JS.
- **Images** — correct dimensions, modern formats, `loading="lazy"` where appropriate; LCP image prioritized ([Optimize LCP](https://web.dev/articles/lcp)).
- **Lists** — **virtualize** very long lists (e.g. `react-window`, TanStack Virtual).
- **Expensive work** — debounce/throttle search; move heavy work off main thread where possible.
- **Re-renders** — memoization and stable props where profiling shows benefit (avoid premature optimization).

## UX tie-in

- **Skeletons** and reserved space reduce perceived wait and **CLS**.
- **Fast feedback** on interaction (optimistic UI where safe) improves perceived responsiveness—aligns with **INP** goals.

## Further reading

- [web.dev — Learn performance](https://web.dev/learn/performance)
- [MDN — Performance guides](https://developer.mozilla.org/en-US/docs/Web/Performance)
