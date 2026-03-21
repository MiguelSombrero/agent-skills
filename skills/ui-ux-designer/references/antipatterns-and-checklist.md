# Anti-patterns and checklist

## Avoid / prefer

| Avoid | Prefer |
|--------|--------|
| Low-contrast text or controls | Contrast that meets WCAG; test in light and dark themes |
| Clickable-looking plain text / ambiguous affordances | Real `button` / `a`; visible focus; consistent interactive styling |
| Disabled actions with no explanation | Tooltip or hint: *why* disabled and *what* unlocks it |
| Modals for trivial confirmations | Toasts or inline for low-risk feedback; modals for destructive or irreversible |
| Huge single-page forms | Progressive disclosure, steps, or grouped sections |
| Jargon and error codes in UI | Plain language; actionable next step |
| Color-only status | Icon + text or pattern in addition to color |
| Layout shift when content loads | Reserve space; skeletons; explicit dimensions for media |

## Quick checklist

**Usability**

- [ ] Clear visual hierarchy; primary action obvious
- [ ] Consistent patterns and naming
- [ ] Immediate feedback for actions; system status visible
- [ ] Errors preventable where possible; recovery path clear
- [ ] Undo / cancel where appropriate

**Accessibility**

- [ ] Semantic HTML; headings in order
- [ ] Keyboard operable; focus visible; no keyboard traps
- [ ] Labels and error associations; live regions where needed
- [ ] Contrast and non-color cues for state

**Performance**

- [ ] Images sized and lazy-loaded where appropriate
- [ ] Code splitting / lazy routes for heavy views
- [ ] Long lists virtualized when needed
- [ ] Core Web Vitals considered (see [performance.md](performance.md))

**Responsive**

- [ ] Mobile-first; readable without horizontal scroll
- [ ] Touch targets adequate; no tiny hit areas
- [ ] Works across common viewport widths

**Forms**

- [ ] Labels, errors near fields, preserve input
- [ ] Multi-step progress and safe navigation

**Visual / motion**

- [ ] Systematic type, color, spacing
- [ ] Motion purposeful; reduced motion respected
