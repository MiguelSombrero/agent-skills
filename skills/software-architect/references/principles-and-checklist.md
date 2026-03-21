Reference for software-architect skill. See [SKILL.md](../SKILL.md) for core instructions.
---

## Final Principles

1. **Favor composition over inheritance**
2. **Program to interfaces, not implementations**
3. **Keep it simple** - Don't over-engineer
4. **Make it work, make it right, make it fast** - In that order
5. **Test early and often**
6. **Security is not optional**
7. **Document decisions, not just code**
8. **Optimize for readability** - Code is read more than written
9. **Fail fast** - Validate early, throw errors early
10. **Monitor everything** - You can't improve what you don't measure

---

## Quick Reference Checklist

When reviewing/designing code, check:

**Architecture**

- [ ] Clear separation of concerns
- [ ] Appropriate use of design patterns
- [ ] Loose coupling, high cohesion
- [ ] SOLID principles followed
- [ ] Dependencies point inward

**Security**

- [ ] Input validation at boundaries
- [ ] Parameterized queries (no SQL injection)
- [ ] Output encoding (no XSS)
- [ ] Authentication & authorization
- [ ] Secrets not in code
- [ ] Rate limiting on APIs

**Performance**

- [ ] Appropriate caching strategy
- [ ] Database queries optimized
- [ ] No N+1 query problems
- [ ] Async for I/O operations
- [ ] Background jobs for slow operations

**Reliability**

- [ ] Comprehensive error handling
- [ ] Retry logic for transient failures
- [ ] Circuit breakers for external services
- [ ] Graceful degradation
- [ ] Health checks implemented

**Observability**

- [ ] Structured logging
- [ ] Metrics collection
- [ ] Distributed tracing
- [ ] Alerts configured

**Maintainability**

- [ ] Clear, self-documenting code
- [ ] Comprehensive tests
- [ ] Documentation updated
- [ ] No magic numbers/strings
- [ ] Consistent naming conventions

**Scalability**

- [ ] Stateless where possible
- [ ] Horizontal scaling capability
- [ ] Database indexes on foreign keys
- [ ] Pagination for large datasets
- [ ] Async processing for heavy loads
