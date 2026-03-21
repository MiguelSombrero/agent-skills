# Hexagonal Architecture Review

Reference for software-architect skill. See [SKILL.md](../SKILL.md) for core instructions.

---

## Dependency Rule Checklist

1. Check import statements in every file created or modified
2. **Domain** must not import from service, api, or fhir
3. **Service** must not import from api or fhir
4. Domain has zero outward dependencies
5. Application (service) depends only on domain
6. Infrastructure (api, fhir) depends on application and domain — never the reverse

---

## DDD Building Blocks Review

| Check | What to Verify |
| ----- | -------------- |
| Anemic domain model | Business logic belongs in domain entities/aggregates, not in services |
| Value object immutability | Final fields, no setters, validated at construction |
| Aggregate invariants | Aggregates enforce their own invariants; reject invalid state at boundary |
| Port placement | Ports (interfaces) defined in service/, not in api/ or fhir/ |
| Adapter separation | api/ (REST) calls service ports; fhir/ implements service ports |
| Framework in domain | Domain entities free of framework annotations (@Entity, @Table, etc.) |

---

## Severity Classification

When classifying findings by severity:

| Severity | When to Use |
| -------- | ----------- |
| **BLOCKER** | Hexagonal dependency rule violation, business logic in wrong layer, port defined in api/ or fhir/ |
| **WARNING** | Anemic domain model tendency, value object mutability, aggregate not enforcing invariants |
| **SUGGESTION** | Naming improvements, package organisation tweaks |
