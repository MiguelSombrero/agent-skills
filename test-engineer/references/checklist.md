Reference for test-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.

---

## Quick Reference Checklist

**Before Writing Tests:**

- [ ] Understand what behavior to test
- [ ] Identify edge cases and error conditions
- [ ] Plan test data needs
- [ ] Choose appropriate test type (unit/integration/e2e)

**Writing Tests:**

- [ ] Use descriptive test names
- [ ] Follow AAA pattern (Arrange-Act-Assert)
- [ ] One logical assertion per test
- [ ] Test behavior, not implementation
- [ ] Make tests independent
- [ ] Clean up resources

**For Unit Tests:**

- [ ] Mock external dependencies
- [ ] Test happy path
- [ ] Test edge cases
- [ ] Test error handling
- [ ] Fast execution (< 100ms per test)

**For Integration Tests:**

- [ ] Use real database/services when possible
- [ ] Clean database before each test
- [ ] Test component interactions
- [ ] Verify data persistence
- [ ] Test transactions

**For E2E Tests:**

- [ ] Test critical user journeys
- [ ] Use stable selectors (data-testid)
- [ ] Wait for elements properly
- [ ] Handle authentication
- [ ] Test on multiple browsers (if needed)
- [ ] Keep tests focused and fast

**Code Quality:**

- [ ] Tests are readable and maintainable
- [ ] No hardcoded values (use constants)
- [ ] Good test data builders
- [ ] Meaningful coverage (not just high %)
- [ ] Tests run in CI/CD

---

## Anti-Patterns to Avoid

### Testing Implementation Details

```javascript
// ❌ Bad
test("should call handleSubmit", () => {
  const spy = jest.spyOn(component, "handleSubmit");
  component.render();
  expect(spy).toHaveBeenCalled();
});

// ✅ Good
test("should submit form data", () => {
  render(<Form />);
  fireEvent.click(screen.getByRole("button", { name: "Submit" }));
  expect(mockSubmit).toHaveBeenCalledWith({ name: "John" });
});
```

### Large, Unfocused Tests

```javascript
// ❌ Bad
test("user flow", async () => {
  const user = await createUser();
  expect(user.id).toBeDefined();
  const updated = await updateUser(user.id, { name: "New" });
  expect(updated.name).toBe("New");
  const deleted = await deleteUser(user.id);
  expect(deleted).toBe(true);
  // Too many concerns in one test!
});

// ✅ Good - Separate tests
test("should create user", async () => {});
test("should update user", async () => {});
test("should delete user", async () => {});
```

### Not Cleaning Up

```javascript
// ❌ Bad
test("should create file", () => {
  fs.writeFileSync("/tmp/test.txt", "data");
  expect(fs.existsSync("/tmp/test.txt")).toBe(true);
  // File left behind!
});

// ✅ Good
test("should create file", () => {
  const file = "/tmp/test.txt";
  try {
    fs.writeFileSync(file, "data");
    expect(fs.existsSync(file)).toBe(true);
  } finally {
    if (fs.existsSync(file)) fs.unlinkSync(file);
  }
});
```

### Ignoring Test Failures

```javascript
// ❌ Bad
test.skip("broken test", () => {});

// ✅ Good - Fix or document
test("feature X", () => {
  // TODO: Fix after API changes (see ticket #123)
  expect.todo("implement when ready");
});
```

---

