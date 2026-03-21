---
name: ui-ux-designer
description: Designs user interfaces and experiences for web apps—usability, visual hierarchy, accessibility, performance budgets, forms, navigation, IA, empty/error/offline states, design tokens, dark mode, i18n, RTL, and microcopy. Use when improving UX, layouts, components, user flows, design systems, WCAG-oriented UI, or when the user asks how something should look or behave. For React/TypeScript implementation, hooks, and tests, pair with the JavaScript/TypeScript skill.
---

# UI/UX Designer

Creates user-centered, accessible, and performant web interfaces. This skill focuses on **UX decisions, patterns, and requirements**; **implementation** belongs in the codebase and in the **JavaScript/TypeScript** skill when writing or refactoring React/Next.js code.

## When to use

- **Design or improve** components, layouts, flows, or a design system
- **Usability**, **information architecture**, **navigation**, **search**, **wayfinding**
- **Accessibility** (WCAG-oriented UI, keyboard, focus, semantics)
- **Visual design**: typography, color, spacing, hierarchy, **design tokens**, **dark mode**
- **Responsive** and **touch**-friendly layouts; **container queries** where relevant
- **Forms**: validation UX, multi-step, errors, loading
- **Product states**: empty, error, offline, **permission denied**
- **i18n / RTL**, layout resilience for translated strings
- **UI performance** as users perceive it (Core Web Vitals, skeletons, list scale)
- Questions like “what’s the best UX for …?” or “how should this screen work?”

## Scope

**In scope:** Web UI/UX for apps (especially React/TypeScript/Next.js)—patterns, requirements, checklists, and pointers to authoritative docs.

**Out of scope:** Native mobile-only platforms, games, 3D/VR, pure marketing brand strategy, business model design. (Principles often still apply cross-platform.)

## How to apply this skill

1. **Clarify the user goal** and constraints (device, urgency, accessibility needs).
2. **Recommend UX structure**: hierarchy, flow, copy intent, states (loading / empty / error).
3. **Check** accessibility and responsive requirements using [antipatterns-and-checklist.md](references/antipatterns-and-checklist.md) and [responsive-and-accessibility.md](references/responsive-and-accessibility.md).
4. **Defer implementation** to the **JavaScript/TypeScript** skill for components, hooks, state, API calls, and tests—this skill names *what* to achieve, not production code.

## Routing: which reference to read

| Topic | File |
|--------|------|
| Nielsen heuristics, personas, journeys | [core-ux-principles.md](references/core-ux-principles.md) |
| Typography, color, spacing, hierarchy, theming, i18n/RTL (brief) | [visual-design.md](references/visual-design.md) |
| Atomic design, composition, design systems | [component-design.md](references/component-design.md) |
| Responsive layout, WCAG-oriented UI, keyboard, ARIA, focus | [responsive-and-accessibility.md](references/responsive-and-accessibility.md) |
| Forms, loading, micro-interactions, destructive actions | [forms-and-interactions.md](references/forms-and-interactions.md) |
| Nav, IA, breadcrumbs, deep links, empty/error/offline/permission | [navigation-and-states.md](references/navigation-and-states.md) |
| Core Web Vitals (LCP, INP, CLS), UI perf techniques | [performance.md](references/performance.md) |
| Anti-patterns + consolidated checklist | [antipatterns-and-checklist.md](references/antipatterns-and-checklist.md) |

## Additional resources

All references live under [references/](references/):

- [core-ux-principles.md](references/core-ux-principles.md)
- [visual-design.md](references/visual-design.md)
- [component-design.md](references/component-design.md)
- [responsive-and-accessibility.md](references/responsive-and-accessibility.md)
- [forms-and-interactions.md](references/forms-and-interactions.md)
- [navigation-and-states.md](references/navigation-and-states.md)
- [performance.md](references/performance.md)
- [antipatterns-and-checklist.md](references/antipatterns-and-checklist.md)

## Summary

- Design for **real tasks**; validate assumptions.
- **Simple, consistent** patterns; clear feedback; prevent errors where possible.
- **Accessible by default**; **performant** enough for the context; **test** with users when feasible.

Great UI feels obvious: people reach their goal without fighting the interface.
