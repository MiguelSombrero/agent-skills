# Visual design

Principles for readable, consistent web UI. Implementation (CSS, components) belongs in the app codebase or the **JavaScript/TypeScript** skill—not here.

## Typography

- Use a **limited scale** (e.g. body, title, display) and **consistent weights**; avoid ad-hoc font sizes.
- **Line length**: aim ~45–75 characters per line for body text for readability (often `max-width` in `ch` or `prose`-style constraints).
- **Line height**: comfortable body text (often ~1.5); tighter for large headings.
- **Hierarchy**: one clear primary action per view; headings reflect structure (see semantic HTML in [responsive-and-accessibility.md](responsive-and-accessibility.md)).

## Color

- Prefer **semantic tokens** (e.g. `text-primary`, `surface`, `danger`) over raw hex everywhere.
- **Contrast**: meet [WCAG 2.2 contrast](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html) for text and UI components; do not rely on color alone for state—pair with text, icons, or pattern.
- **Status**: success / warning / error should be distinguishable without color-only cues.

## Spacing and layout

- Use a **spacing scale** (e.g. 4px base) and repeat the same rhythm—reduces arbitrary margins.
- **Whitespace** signals grouping: related items closer; sections separated clearly.
- **Alignment**: pick a grid or consistent edge alignment; avoid jitter.

## Visual hierarchy

- Size, weight, and color guide attention; **one focal point** per section when possible.
- De-emphasize secondary metadata (timestamps, IDs) so primary tasks stay obvious.

## Theming and dark mode (brief)

- Test **semantic tokens** in light and dark; contrast holds in both.
- Respect `prefers-color-scheme` or app-level theme toggle; avoid flashing large bright surfaces on load.

## Internationalization and RTL (brief)

- Allow **longer strings** in layouts (German, Finnish); avoid fixed narrow columns for labels.
- For RTL, use **logical properties** (`margin-inline`, `inset`) where possible; verify icons and directional cues.

## Further reading

- [WCAG 2.2](https://www.w3.org/TR/WCAG22/) — perceivable, operable, understandable, robust.
- [MDN — CSS typography and layout](https://developer.mozilla.org/en-US/docs/Learn/CSS) — implementation details.
