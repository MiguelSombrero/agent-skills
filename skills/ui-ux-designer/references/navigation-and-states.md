# Navigation, IA, and UI states

Information architecture and wayfinding. **Routing implementation** (React Router, Next.js) is for the **JavaScript/TypeScript** skill.

## Information architecture

- **Group** content by user tasks and mental models, not only org structure.
- **Depth vs breadth**: fewer top-level items with clear labels often beats deep mystery menus.
- **Consistent labels** across nav, page titles, and URLs where possible.

## Navigation patterns

- **Primary navigation**: persistent, highlights current section; limit items to what people use most.
- **Secondary**: subpages, filters, or tools—visually subordinate to primary.
- **Breadcrumbs**: useful for deep hierarchies; last item is current page (not always a link).
- **Tabs vs routes**: tabs for **same context** switching; full **navigation** for distinct sections—avoid hiding critical actions only under tabs.
- **Search**: visible entry point when catalog is large; empty and error states for results (see below).
- **Deep links**: URLs reflect location; back/forward behavior predictable.

## Global UI states (what users need)

**Loading**

- What is loading; optional **progress** for long tasks; **skeleton** matching layout.

**Empty**

- Why it is empty; **primary action** (e.g. “Create …”, “Import …”); helpful illustration optional, not required for clarity.

**Error**

- Plain language; **what failed**; **retry** or **support** path; preserve user data when possible.

**Offline / degraded**

- Clear “you’re offline” or “can’t reach server”; **retry**; explain what still works.

**Permission denied / unauthorized**

- Why access is missing; **log in** / **request access** / **contact admin** as appropriate—avoid raw 403 pages in product flows.

## Related references

- Forms and flows: [forms-and-interactions.md](forms-and-interactions.md)
- Accessibility (landmarks, focus): [responsive-and-accessibility.md](responsive-and-accessibility.md)
- Checklist: [antipatterns-and-checklist.md](antipatterns-and-checklist.md)
