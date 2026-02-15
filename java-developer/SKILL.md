---
name: java-developer
description: Senior Java developer specializing in Spring Boot applications with expertise in clean architecture, testing, and cloud-native patterns. Use when developing REST APIs, implementing Spring Security, designing testable code, refactoring for maintainability, or working with Spring Data JPA. Focuses on TDD, SOLID principles, and modern Spring Boot best practices with latest Java and Spring Boot versions.
---

# Java Developer

Builds production-ready, cloud-native Spring Boot applications with emphasis on clean architecture, comprehensive testing, and maintainable code. Combines deep Spring Framework knowledge with architectural principles and testing best practices.

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
        User user = User.builder()
            .email(request.email())
            .passwordHash(passwordEncoder.encode(request.password()))
            .name(request.name())
            .build();
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
    Order order = new Order();
    order.addItem(new OrderItem(new BigDecimal("10.00"), 2));
    order.addItem(new OrderItem(new BigDecimal("5.00"), 1));
    assertThat(order.calculateTotal()).isEqualByComparingTo(new BigDecimal("25.00"));
}
```

---

## Application Architecture

### Layered Architecture

```
┌─────────────────────────────────────────┐
│         Presentation Layer              │  Controllers, DTOs
├─────────────────────────────────────────┤
│         Application Layer               │  Services, Use Cases
├─────────────────────────────────────────┤
│         Domain Layer                    │  Entities, Value Objects
├─────────────────────────────────────────┤
│         Infrastructure Layer            │  Repositories, External APIs
└─────────────────────────────────────────┘
```

### Package Structure

```
com.example.app/
├── api/controller/, api/dto/, api/mapper/
├── application/service/, application/usecase/
├── domain/model/, domain/repository/, domain/exception/, domain/valueobject/
├── infrastructure/persistence/, infrastructure/external/, infrastructure/config/
└── shared/exception/, shared/validation/
```

### Rich Domain Model

Put business logic in domain entities, not services. Use factory methods, enforce invariants, avoid anemic models.

```java
@Entity
@Table(name = "orders")
public class Order {
    private OrderStatus status;
    private List<OrderItem> items = new ArrayList<>();

    public static Order create(List<OrderItem> items) {
        Order order = new Order();
        order.status = OrderStatus.PENDING;
        items.forEach(order::addItem);
        return order;
    }

    public void addItem(OrderItem item) {
        requireNonNull(item);
        items.add(item);
        recalculateTotal();
    }

    public void ship() {
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("Can only ship pending orders");
        }
        status = OrderStatus.SHIPPED;
    }
}
```

---

## Additional Resources

For detailed patterns and examples, see these reference files:

- **REST API, DTOs, Spring Data JPA**: [rest-and-data.md](references/rest-and-data.md) — Controllers, validation, error handling, repositories, entity relationships, transactions
- **Spring Security and JWT**: [security.md](references/security.md) — Security config, JWT auth, method security
- **Testing**: [testing.md](references/testing.md) — Unit, integration, controller, E2E tests with TestContainers
- **Cross-cutting, config, performance**: [operations.md](references/operations.md) — Logging, monitoring, tracing, health, application.yml, caching, async
- **Clean code and anti-patterns**: [patterns.md](references/patterns.md) — Naming, method design, anemic domain, god service, transaction anti-patterns, checklist

---

## Summary

**Key Principles:**

- Test-driven development (TDD)
- SOLID principles
- Clean, maintainable code
- Rich domain models
- Comprehensive testing
- Security by default
- Performance optimization
- Cloud-native patterns

**Remember**: Write tests first, keep methods small, make domain models rich with behavior, and always favor maintainability over cleverness.
