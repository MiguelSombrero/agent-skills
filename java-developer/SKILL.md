---
name: java-developer
description: Senior Java developer specializing in Spring Boot applications with Domain-Driven Design (DDD) and rich domain models. Expertise in clean architecture, testing, and cloud-native patterns. Use when developing REST APIs, implementing Spring Security, designing domain models, structuring aggregates and value objects, refactoring for maintainability, or working with Spring Data JPA. Focuses on DDD, rich domain models, TDD, SOLID principles, and modern Spring Boot best practices with latest Java and Spring Boot versions.
---

# Java Developer

Builds production-ready, cloud-native Spring Boot applications with emphasis on Domain-Driven Design, rich domain models, clean architecture, comprehensive testing, and maintainable code. Combines deep Spring Framework knowledge with DDD tactical patterns and testing best practices.

## When to Use This Skill

Use this skill when the user:

- Wants to **build REST APIs** with Spring Boot
- Needs help with **Spring Boot application structure** or architecture
- Asks about **Spring Data JPA**, repositories, or database access
- Wants to implement **Spring Security** or authentication/authorization
- Needs guidance on **testing** (unit, integration, e2e) in Spring Boot
- Asks about **cross-cutting concerns** (logging, monitoring, tracing)
- Wants to **refactor code** for better maintainability
- Needs help with **configuration management** or profiles
- Asks about **error handling** or exception strategies
- Wants to implement **validation** or data binding
- Needs guidance on **dependency injection** or Spring beans
- Asks "how should I structure..." or "what's the best way in Spring Boot..."
- Wants to apply **Domain-Driven Design (DDD)** or **rich domain models**
- Wants to apply **SOLID principles** or design patterns
- Needs help with **Spring Boot starters** or auto-configuration

## Scope

**In Scope:**

- Spring Boot application development
- REST API design and implementation
- Spring Data JPA and database access
- Spring Security and authentication
- Comprehensive testing strategies
- Clean architecture and layered design
- Dependency injection and IoC
- Configuration and profiles
- Cross-cutting concerns (logging, monitoring, tracing)
- Error handling and validation
- Performance optimization
- Cloud-native patterns

**Out of Scope:**

- Frontend development (React, Angular, etc.)
- DevOps and infrastructure (use devops-engineer skill)
- Non-Spring Java frameworks
- Android development
- Desktop applications

**Technology Stack:**

- **Java 25** (latest features)
- **Spring Boot 4.x** (latest major version)
- **JUnit 5** (Jupiter)
- **Mockito 5.x**
- **TestContainers** for integration tests
- **Maven** or **Gradle** as build tools

---

## Core Principles

### SOLID Principles in Spring Boot

**Single Responsibility**: Each class should have one reason to change. Controllers delegate to services; services orchestrate business logic and repositories.

```java
// ✅ Good - Separated concerns
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;

    @PostMapping
    public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest request) {
        User user = userService.createUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(UserResponse.from(user));
    }
}

@Service
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    @Transactional
    public User createUser(CreateUserRequest request) {
        User user = User.create(
            request.name(),
            request.email(),
            passwordEncoder.encode(request.password())
        );
        return userRepository.save(user);
    }
}
```

**Dependency Inversion**: Depend on abstractions, not concretions. Use interfaces for services; inject implementations via constructor.

```java
// ✅ Good - Program to interface
public interface NotificationService { void send(String recipient, String message); }

@Service
public class OrderService {
    private final NotificationService notificationService;
    public OrderService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }
}
```

### Test-Driven Development (TDD)

**Red-Green-Refactor**: Write failing test first, implement minimum code to pass, then refactor. Use Arrange-Act-Assert structure.

```java
@Test
void shouldCalculateOrderTotal() {
    Order order = Order.create(1L, List.of(
        new OrderItem(new BigDecimal("10.00"), 2),
        new OrderItem(new BigDecimal("5.00"), 1)
    ));
    assertThat(order.getTotal()).isEqualByComparingTo(new BigDecimal("25.00"));
}
```

### Domain-Driven Design (DDD)

Apply tactical DDD patterns to model the domain explicitly:

- **Ubiquitous Language**: Use domain terms in code (e.g., `order.ship()`, `OrderStatus.CONFIRMED`) instead of technical jargon
- **Bounded Contexts**: Delineate clear model boundaries per subdomain; avoid mixing concerns across contexts
- **Aggregates**: Cluster related entities with a root; enforce invariants at aggregate boundaries; load/save aggregate roots
- **Value Objects**: Immutable types defined by attributes (e.g., `Money`, `Email`) rather than identity; validate in constructor
- **Domain Events**: Model significant domain occurrences explicitly for decoupling and auditability

### Rich Domain Model

Put business logic and invariants in domain entities—not in services:

- Use **factory methods** (`Order.create(...)`) instead of public constructors or builders for entities
- Prefer expressive domain methods (`order.ship()`, `order.confirm()`) over procedural service logic (`order.setStatus(SHIPPED)`)
- Enforce invariants inside the aggregate; reject invalid state at the boundary
- Avoid anemic models (getters/setters only with no behavior)

For detailed DDD and rich domain patterns, see [ddd-and-domain-model.md](references/ddd-and-domain-model.md).

---

## Application Architecture

### Layered Architecture

```
┌─────────────────────────────────────────┐
│         Presentation Layer              │  Controllers, DTOs
├─────────────────────────────────────────┤
│         Application Layer               │  Services, Use Cases
├─────────────────────────────────────────┤
│         Domain Layer                    │  Aggregates, Value Objects, Domain Events
├─────────────────────────────────────────┤
│         Infrastructure Layer            │  Repositories, External APIs
└─────────────────────────────────────────┘
```

### Package Structure

The Domain Layer is the heart of the design—it must not depend on infrastructure.

```
com.example.orderservice/   # Service name implies context
├── api/
│   ├── controller/
│   ├── dto/
│   └── mapper/             # DTO ↔ Domain mappers
├── application/
│   ├── service/
│   └── port/
├── domain/
│   ├── aggregate/
│   ├── entity/             # Domain entities like Order
│   ├── valueobject/
│   ├── service/
│   ├── event/
│   └── repository/
├── infrastructure/
│   ├── persistence/
│   │   ├── entity/         # JPA entities like OrderEntity
│   │   ├── repository/
│   │   └── mapper/         # Domain ↔ JPA mappers
│   ├── external/           # External HTTP clients
│   │   ├── client/
│   │   ├── dto/
│   │   ├── mapper/         # Anti-corruption layer
│   │   └── config/
│   ├── messaging/
│   └── config/
└── shared/
    └── exception/
```

---

## Additional Resources

For detailed patterns and examples, see these reference files:

- **DDD and rich domain models**: [ddd-and-domain-model.md](references/ddd-and-domain-model.md) — Ubiquitous language, aggregates, value objects, domain events, anti-patterns
- **REST API, DTOs, Spring Data JPA**: [rest-and-data.md](references/rest-and-data.md) — Controllers, validation, error handling, repositories, entity relationships, transactions
- **Spring Security and JWT**: [security.md](references/security.md) — Security config, JWT auth, method security
- **Testing**: [testing.md](references/testing.md) — Unit, integration, controller, E2E tests with TestContainers
- **Cross-cutting, config, performance**: [operations.md](references/operations.md) — Logging, monitoring, tracing, health, application.yml, caching, async
- **Clean code and anti-patterns**: [patterns.md](references/patterns.md) — Naming, method design, anemic domain, god service, transaction anti-patterns, checklist

---

## Summary

**Key Principles:**

- Domain-Driven Design (DDD) and rich domain models
- Test-driven development (TDD)
- SOLID principles
- Clean, maintainable code
- Comprehensive testing
- Security by default
- Performance optimization
- Cloud-native patterns

**Remember**: Model the domain with ubiquitous language, make domain models rich with behavior and invariants, write tests first, keep methods small, and always favor maintainability over cleverness.
