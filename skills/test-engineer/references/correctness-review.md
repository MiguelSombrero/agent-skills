# Correctness Review Checklist

Reference for test-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.

---

## Acceptance Criteria Verification

When verifying that implementation meets acceptance criteria:

- For each criterion, verify the implementation correctly addresses it
- Confirm the test actually verifies the behaviour described in its name
- Check for missing or overly weak assertions

---

## Edge Case Checks

| Category | Examples |
| -------- | -------- |
| Null inputs | `null` parameters, optional absent |
| Empty collections | Empty lists, empty maps, empty strings |
| Boundary values | Min/max integers, zero, negative numbers |
| Invalid state transitions | Illegal status changes, invalid combinations |
| Invalid input | Malformed data, wrong types |

---

## Error Handling Verification

- Correct exception types for domain vs technical errors
- Meaningful error messages (no empty or generic messages)
- Proper HTTP status codes (400, 404, 500 as appropriate)
- No sensitive data in error responses
- Domain validation at construction time for value objects

---

## Test Assertion Strength

- Assertions cover the full expected outcome, not just partial checks
- No assertions that always pass (e.g. asserting a constant)
- Test name matches what is actually verified
- Edge cases have explicit assertions

---

## Severity Tags for Findings

| Severity | When to Use |
| -------- | ----------- |
| **BLOCKER** | Acceptance criterion not met, logic error, missing critical error handling |
| **WARNING** | Edge case not covered, weak assertion, potentially incorrect behaviour under certain conditions |
| **SUGGESTION** | Minor improvement to test clarity or assertion specificity |
