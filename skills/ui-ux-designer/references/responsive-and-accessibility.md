# Responsive design and accessibility

Requirements and pointers. **Implementation** (components, tests) belongs in the **JavaScript/TypeScript** skill.

## Responsive design

- **Mobile-first**: base styles for small screens, then enhance for larger breakpoints.
- **Touch targets**: minimum ~44×44 CSS px for interactive controls ([Understanding Target Size](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)).
- **Grids and stacks**: stack on narrow viewports; use multi-column layouts when space allows.
- **Container queries**: consider component-level breakpoints when a widget lives in many layouts ([MDN — CSS container queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries)).

## Semantic HTML

- Use **native elements** (`button`, `a`, `nav`, `main`, `h1`–`h6`, `label`) before custom roles.
- **Heading order** reflects outline; do not skip levels for styling—change styles instead.

## Keyboard and focus

- All interactive content **reachable with Tab**; logical tab order.
- **Visible focus** indicators; do not remove outline without an equivalent.
- **Escape** closes overlays; **Arrow keys** where expected for composite widgets (tabs, menus, listbox)—see [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/).
- **Modals / dialogs**: focus moves inside on open; **focus returns** on close; scroll locked appropriately; `aria-modal` / `role="dialog"` per pattern.

## ARIA

- **Prefer native HTML**; add ARIA when semantics are missing or dynamic ([Using ARIA](https://www.w3.org/WAI/ARIA/apg/practices/read-me-first/)).
- **Labels**: every control has an accessible name (`label`, `aria-label`, or `aria-labelledby`).
- **Live regions**: announce important async updates (`role="status"`, `aria-live`) without stealing focus.

## Screen readers and motion

- **Alt text** for meaningful images; empty `alt` for decorative.
- Respect **`prefers-reduced-motion`**: reduce or disable non-essential animation ([MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)).

## WCAG

- Aim for **WCAG 2.2 AA** as a common org target; verify contrast, targets, focus, labels, errors ([WCAG 2.2](https://www.w3.org/TR/WCAG22/)).

## Accessible component libraries

- Prefer **battle-tested primitives** for combobox, menu, dialog, tabs: [Radix UI](https://www.radix-ui.com/), [React Aria](https://react-spectrum.adobe.com/react-aria/), Headless UI—then apply visual design tokens.
