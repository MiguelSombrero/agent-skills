Reference for ui-ux-designer skill. See [SKILL.md](../SKILL.md) for core instructions.
---

## Anti-Patterns to Avoid

### Poor Contrast

```tsx
// ❌ Bad - Poor color contrast
<button className="bg-gray-300 text-gray-400">Click me</button>

// ✅ Good - Sufficient contrast (WCAG AA: 4.5:1)
<button className="bg-blue-600 text-white">Click me</button>
```

### Unclear Interactive Elements

```tsx
// ❌ Bad - Looks like text, acts like button
<span onClick={handleClick} className="text-blue-500">
  Delete
</span>

// ✅ Good - Clear affordances
<button
  onClick={handleClick}
  className="text-blue-500 underline hover:text-blue-700"
>
  Delete
</button>
```

### Disabled Buttons Without Explanation

```tsx
// ❌ Bad - User doesn't know why button is disabled
<button disabled>Submit</button>

// ✅ Good - Clear explanation
<Tooltip content="Please fill in all required fields">
  <button disabled={!isFormValid}>Submit</button>
</Tooltip>
```

### Modal Overuse

```tsx
// ❌ Bad - Modal for simple confirmations
<Modal>
  <p>Item deleted</p>
  <button onClick={closeModal}>OK</button>
</Modal>;

// ✅ Good - Use toast for non-critical feedback
toast.success("Item deleted");

// ✅ Good - Modal for critical actions that need attention
<Modal>
  <h2>Delete Account?</h2>
  <p>This action cannot be undone.</p>
  <button onClick={confirmDelete}>Delete</button>
  <button onClick={closeModal}>Cancel</button>
</Modal>;
```

### Overwhelming Forms

```tsx
// ❌ Bad - All fields at once
<form>
  <input placeholder="First name" />
  <input placeholder="Last name" />
  <input placeholder="Email" />
  <input placeholder="Phone" />
  <input placeholder="Address line 1" />
  <input placeholder="Address line 2" />
  <input placeholder="City" />
  <input placeholder="State" />
  <input placeholder="ZIP" />
  <input placeholder="Country" />
  <button>Submit</button>
</form>

// ✅ Good - Progressive disclosure
<form>
  <input placeholder="Email" />
  <button>Continue</button>
  {/* Show more fields after first step */}
</form>
```

---

## Quick Reference Checklist

**Usability:**

- [ ] Clear visual hierarchy
- [ ] Consistent design system
- [ ] Obvious interactive elements
- [ ] Immediate feedback for actions
- [ ] Error prevention and helpful error messages
- [ ] Easy undo/cancel options

**Accessibility:**

- [ ] Semantic HTML elements
- [ ] ARIA labels where needed
- [ ] Keyboard navigation support
- [ ] Focus indicators visible
- [ ] Color contrast meets WCAG AA (4.5:1)
- [ ] Screen reader compatible

**Performance:**

- [ ] Images optimized and lazy loaded
- [ ] Code splitting implemented
- [ ] Long lists virtualized
- [ ] Expensive operations memoized
- [ ] Core Web Vitals monitored

**Responsive:**

- [ ] Mobile-first design
- [ ] Touch targets min 44x44px
- [ ] Works on all viewport sizes
- [ ] No horizontal scrolling
- [ ] Legible text without zooming

**Forms:**

- [ ] Clear labels and placeholders
- [ ] Inline validation
- [ ] Helpful error messages
- [ ] Progress indication for multi-step
- [ ] Save/autosave functionality

**Visual Design:**

- [ ] Consistent typography scale
- [ ] Systematic color palette
- [ ] Adequate spacing
- [ ] Visual feedback for interactions
- [ ] Respect reduced motion preference

---

