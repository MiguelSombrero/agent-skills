# Security Review Checklist

Reference for java-developer skill. See [SKILL.md](../SKILL.md) for core instructions.

---

## Review Scope

| Category | What to Check |
| -------- | ------------- |
| **Injection risks** | SQL injection (raw queries, string concatenation in JPQL/native queries), command injection, LDAP injection, log injection |
| **Input validation** | REST endpoint inputs validated (`@Valid`, `@NotNull`, `@Size`, etc.)? Domain objects validated at construction time? |
| **Sensitive data exposure** | Passwords, tokens, secrets, PII logged or in API responses? Error messages leaking internal details? |
| **Insecure defaults** | Hardcoded credentials, overly permissive CORS, missing CSRF protection, disabled security features |
| **Mass assignment** | DTOs properly scoped to prevent binding of unintended fields? |
| **Access control** | If applicable, are authorization checks in place? |

---

## Severity Classification

When classifying findings by severity:

| Severity | When to Use |
| -------- | ----------- |
| **BLOCKER** | Injection vulnerability, exposed credentials/secrets, missing authentication on sensitive endpoint |
| **WARNING** | Missing input validation, overly verbose error responses, missing CORS/CSRF configuration |
| **SUGGESTION** | Defensive coding improvements, additional validation, security header recommendations |
