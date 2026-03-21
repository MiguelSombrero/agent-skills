# Forms and interactions

UX-focused guidance for forms, feedback, and motion. **Validation logic and component code** belong in the **JavaScript/TypeScript** skill.

## Forms

- **Labels** visible and associated with controls; required fields marked clearly.
- **Errors**: next to the field; message says **what failed** and **how to fix**; preserve user input.
- **Timing**: on blur or after pause for format-sensitive fields; avoid shouting on every keystroke unless helpful (e.g. password rules).
- **Server authority**: client hints, server decides; map server errors to fields when possible.
- **Multi-step flows**: show **progress**; allow **back** without losing data; final **review** for high-stakes data.

## Loading and progress

- Prefer **skeletons** that match final layout over generic spinners for content-heavy views.
- **Progress** for long operations (upload, batch); allow **cancel** when safe.
- Announce busy state for assistive tech (`aria-busy`, live region) where appropriate.

## Micro-interactions

- **Feedback** within ~100ms perceived: hover/active, success toast, inline checkmark.
- **Motion** supports understanding (transition between states), not decoration—keep durations modest.
- Honor **`prefers-reduced-motion`** ([MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)).

## Destructive actions

- **Confirm** irreversible actions; make consequence explicit; primary safe action easy to choose.

## Related references

- Core heuristics: [core-ux-principles.md](core-ux-principles.md)
- Checklist: [antipatterns-and-checklist.md](antipatterns-and-checklist.md)
