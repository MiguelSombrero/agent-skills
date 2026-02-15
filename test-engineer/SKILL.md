---
name: test-engineer
description: Creates comprehensive automated tests including unit tests, integration tests, and end-to-end tests for JavaScript and Java/Spring Boot applications. Use when the user wants to write tests, improve test coverage, add test suites, or validate application behavior. Supports Jest/Vitest, JUnit 5, Spring Boot Test, Playwright, and other developer testing frameworks. Focuses on developer-written automated tests, not manual QA testing.
---

# Test Engineer

Creates production-quality automated tests across the testing pyramid: unit tests, integration tests, and end-to-end tests. Focuses on developer-written tests that catch bugs early, ensure code quality, and enable confident refactoring.

## When to Use This Skill

Use this skill when the user:

- Asks to "write tests", "add tests", or "test this code"
- Wants to improve test coverage or test quality
- Mentions "unit test", "integration test", "e2e test", "end-to-end test"
- Wants to test APIs, databases, external services
- Asks about testing frameworks (Jest, JUnit, Playwright, TestContainers, etc.)
- Needs to test frontend interactions, user flows, or UI components
- Wants to validate business logic, edge cases, or error handling
- Asks "how do I test..." for any application component

## Scope: Developer Tests

**In Scope** (Developer-written automated tests):

- Unit tests (isolated component testing)
- Integration tests (component interaction testing)
- API tests (REST endpoint testing)
- Database integration tests
- End-to-end tests (user journey testing)
- Contract tests (API contract validation)

**Out of Scope** (QA/Manual testing):

- Manual test cases or test plans
- Exploratory testing procedures
- Performance/load testing (unless code examples)
- Security penetration testing
- User acceptance testing (UAT)
- Manual regression test checklists

---

## Testing Pyramid

```
        ┌─────────────┐
        │     E2E     │  Slow, expensive, brittle
        │   (Few)     │  Full user journeys
        └─────────────┘
       ┌───────────────┐
       │  Integration  │  Medium speed/cost
       │    (Some)     │  Component interactions
       └───────────────┘
      ┌─────────────────┐
      │   Unit Tests    │  Fast, cheap, reliable
      │     (Many)      │  Isolated components
      └─────────────────┘
```

**Recommended Distribution:**

- **70%** Unit Tests - Fast, isolated, test business logic
- **20%** Integration Tests - Test component interactions, DB, APIs
- **10%** E2E Tests - Test critical user journeys

**Why This Matters:**

- Unit tests run in milliseconds, E2E tests in seconds/minutes
- Unit tests are stable, E2E tests can be flaky
- Unit tests pinpoint exact failures, E2E tests show broader issues
- More unit tests = faster feedback, lower maintenance

---

## Core Testing Principles

### Test Independence

**Each test must be completely independent.**

```javascript
// ❌ Bad - Tests depend on execution order
let userId;
test("creates user", async () => {
  userId = await createUser({ name: "Alice" });
  expect(userId).toBeDefined();
});

test("finds user", async () => {
  const user = await findUser(userId); // Depends on previous test!
  expect(user.name).toBe("Alice");
});

// ✅ Good - Independent tests
describe("User operations", () => {
  test("creates user", async () => {
    const userId = await createUser({ name: "Alice" });
    expect(userId).toBeDefined();
  });

  test("finds user", async () => {
    const userId = await createUser({ name: "Bob" });
    const user = await findUser(userId);
    expect(user.name).toBe("Bob");
  });
});
```

### Arrange-Act-Assert (AAA)

**Structure every test clearly.**

```javascript
test("calculates order total with discount", () => {
  // Arrange - Set up test data
  const order = new Order([
    { price: 100, quantity: 2 },
    { price: 50, quantity: 1 },
  ]);
  const discount = 0.1; // 10% discount

  // Act - Execute the behavior
  const total = order.calculateTotal(discount);

  // Assert - Verify the result
  expect(total).toBe(225); // (100*2 + 50*1) * 0.9
});
```

### Descriptive Test Names

**Test names should explain what is tested and expected outcome.**

```javascript
// ❌ Bad - Unclear names
test("test1", () => {
  /* ... */
});
test("userTest", () => {
  /* ... */
});

// ✅ Good - Clear, descriptive names
test("should create user with valid email", () => {
  /* ... */
});
test("should throw error when email is invalid", () => {
  /* ... */
});
test("should return 404 when user not found", () => {
  /* ... */
});

// Even better - BDD style
describe("OrderService", () => {
  describe("when placing an order", () => {
    it("should charge the payment method", () => {
      /* ... */
    });
    it("should send confirmation email", () => {
      /* ... */
    });
    it("should update inventory", () => {
      /* ... */
    });
  });
});
```

### Test Behavior, Not Implementation

**Tests should verify what code does, not how it does it.**

```javascript
// ❌ Bad - Testing implementation details
test("should call internal helper method", () => {
  const spy = jest.spyOn(service, "_calculateTax");
  service.processOrder(order);
  expect(spy).toHaveBeenCalled(); // Breaks when refactoring
});

// ✅ Good - Testing behavior
test("should include tax in order total", () => {
  const order = { subtotal: 100, taxRate: 0.1 };
  const result = service.processOrder(order);
  expect(result.total).toBe(110);
});
```

---

---

## Additional Resources

For detailed patterns and examples, see these reference files:

- **JavaScript testing**: [javascript-testing.md](references/javascript-testing.md) — Jest/Vitest, Supertest, Playwright, E2E
- **Java/Spring Boot testing**: [java-testing.md](references/java-testing.md) — JUnit 5, Mockito, TestContainers, MockMvc
- **Best practices**: [best-practices.md](references/best-practices.md) — Test data, coverage
- **Checklist and anti-patterns**: [checklist.md](references/checklist.md) — Quick reference, common mistakes

---

## Summary

**Testing Pyramid:**

- Write mostly unit tests (fast, focused)
- Some integration tests (component interactions)
- Few E2E tests (critical user journeys)

**Key Principles:**

- Tests should be independent
- Test behavior, not implementation
- Make tests readable and maintainable
- Fast feedback is crucial
- Meaningful coverage over high percentages

**Test Types:**

- **Unit**: Isolated components, fast, many
- **Integration**: Component interactions, medium, some
- **E2E**: User journeys, slow, few

**Remember**: Good tests give you confidence to refactor and deploy. They should catch bugs, not prevent progress.
