# Code Quality Review Checklist

Reference for java-developer skill. See [SKILL.md](../SKILL.md) for core instructions.

---

## Review Scope

| Criterion                  | What to Check                                                                                         |
| -------------------------- | ----------------------------------------------------------------------------------------------------- |
| **Readability**            | Is the code clear and self-documenting? Are method and variable names descriptive?                    |
| **Naming**                 | Do class, method, and variable names follow Java conventions and express intent?                      |
| **Duplication**            | Is there unnecessary code duplication that should be extracted?                                       |
| **Java 25 idioms**         | Records, sealed interfaces, pattern matching, text blocks                                             |
| **Spring Boot 4.x idioms** | Constructor injection, `@RestController` patterns, proper use of `@Service`/`@Repository` stereotypes |
| **Unnecessary complexity** | Is the code as simple as it can be? Over-engineered abstractions?                                     |

---

## Severity Classification

When classifying findings by severity:

| Severity       | When to Use                                                                                               |
| -------------- | --------------------------------------------------------------------------------------------------------- |
| **BLOCKER**    | Severely unreadable code, major naming confusion that will cause maintenance issues                       |
| **WARNING**    | Significant duplication, missed opportunity for a modern Java feature that substantially improves clarity |
| **SUGGESTION** | Minor naming tweaks, style improvements, optional simplifications                                         |
