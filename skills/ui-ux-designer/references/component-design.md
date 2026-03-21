# Component design

How to structure UI for clarity and reuse. **Code structure** (React props, hooks, tests) is covered by the **JavaScript/TypeScript** skill; this file is UX-oriented patterns only.

## Atomic design (conceptual)

- **Atoms** — smallest pieces (buttons, inputs, labels).
- **Molecules** — simple combinations (search field + button).
- **Organisms** — distinct sections (header, card with actions).
- **Templates / pages** — layout and composition without real content.
- Use as a **vocabulary**, not a rigid filing system—avoid over-splitting one-off pieces.

## Composition over configuration

- Prefer **children and small subcomponents** over giant prop lists (`variant`, `size`, twelve booleans).
- **Stable, predictable APIs**: few required props; optional slots for customization.

## Compound and slot patterns

- **Compound components** (e.g. `Tabs` + `Tab` + `Panel`) expose structure that matches mental model.
- **Slots** or named areas (`header`, `footer`, `actions`) keep layout flexible without prop explosion.

## Design systems

- **Tokens** for color, type, spacing, radii, motion—document meaning (“danger”, “surface”) not only values.
- **Documentation**: when to use each variant; accessibility notes (keyboard, labels).
- Prefer **accessible primitives** from established libraries ([Radix UI](https://www.radix-ui.com/), [React Aria](https://react-spectrum.adobe.com/react-aria/)) for complex widgets—then theme to match product.

## Related references

- Accessibility behavior: [responsive-and-accessibility.md](responsive-and-accessibility.md)
- Forms: [forms-and-interactions.md](forms-and-interactions.md)

## Further reading

- [React — Passing JSX as children](https://react.dev/learn/passing-props-to-a-component) — composition model.
