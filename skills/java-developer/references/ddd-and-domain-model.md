# DDD and Rich Domain Models

Reference for java-developer skill. See [SKILL.md](../SKILL.md) for core instructions.

---

## Ubiquitous Language

Model the domain in code using terms from the business. Classes, methods, and variables should reflect domain vocabulary.

```java
// ✅ Good - Domain terminology
order.ship();
order.confirm();
order.cancel();
OrderStatus.PENDING;
OrderStatus.CONFIRMED;

// ❌ Bad - Technical or generic names
order.updateStatus("shipped");
order.setState(1);
order.doAction(Action.SHIP);
```

---

## Aggregates

An aggregate is a cluster of related entities with a root (aggregate root). Enforce invariants at aggregate boundaries; external code references only the root.

**Rules:**
- One aggregate root per aggregate
- Load and save entire aggregate through the root
- Enforce business rules inside the aggregate; reject invalid state

```java
@Entity
@Table(name = "orders")
public class Order {  // Aggregate root
    private Long customerId;
    private OrderStatus status;
    private List<OrderItem> items = new ArrayList<>();

    public static Order create(Long customerId, List<OrderItem> items) {
        if (items == null || items.isEmpty()) {
            throw new IllegalArgumentException("Order must have at least one item");
        }
        Order order = new Order();
        order.customerId = customerId;
        order.status = OrderStatus.PENDING;
        items.forEach(order::addItem);
        return order;
    }

    public void addItem(OrderItem item) {
        requireNonNull(item);
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("Cannot modify order after confirmation");
        }
        items.add(item);
    }

    public void confirm() {
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("Order already confirmed or processed");
        }
        status = OrderStatus.CONFIRMED;
    }

    public void ship() {
        if (status != OrderStatus.CONFIRMED) {
            throw new IllegalStateException("Can only ship confirmed orders");
        }
        status = OrderStatus.SHIPPED;
    }

    public void cancel() {
        if (status == OrderStatus.SHIPPED || status == OrderStatus.DELIVERED) {
            throw new IllegalStateException("Cannot cancel shipped or delivered order");
        }
        status = OrderStatus.CANCELLED;
    }
}
```

---

## Value Objects

Immutable types defined by their attributes, not identity. Validate in constructor; two value objects with same attributes are equal.

```java
// Money - immutable, validated
public record Money(BigDecimal amount, Currency currency) {
    public Money {
        Objects.requireNonNull(amount);
        Objects.requireNonNull(currency);
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
    }
}

// Email - domain concept as value object
public record Email(String value) {
    public Email {
        Objects.requireNonNull(value);
        if (!value.matches("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$")) {
            throw new IllegalArgumentException("Invalid email format: " + value);
        }
    }
}

// Usage in entity
@Entity
public class User {
    private Email email;  // Value object, not String

    public static User create(String name, Email email, String passwordHash) {
        User user = new User();
        user.name = name;
        user.email = email;
        user.passwordHash = passwordHash;
        return user;
    }
}
```

---

## Rich Domain Model

Put business logic in domain entities. Use factory methods, encapsulated state changes, and expressive behavior.

**Factory methods** instead of public constructors or builders:

```java
// ✅ Good
public static Order create(Long customerId, List<OrderItem> items) {
    Order order = new Order();
    order.customerId = customerId;
    order.status = OrderStatus.PENDING;
    items.forEach(order::addItem);
    return order;
}

// ❌ Bad - Anemic
Order order = Order.builder()
    .status(OrderStatus.PENDING)
    .items(items)
    .build();
```

**Expressive behavior** instead of setters:

```java
// ✅ Good
order.confirm();
order.ship();

// ❌ Bad
order.setStatus(OrderStatus.CONFIRMED);
order.setStatus(OrderStatus.SHIPPED);
order.setShippedAt(LocalDateTime.now());
```

**Invariant enforcement** inside the aggregate:

```java
public void addItem(OrderItem item) {
    if (status != OrderStatus.PENDING) {
        throw new IllegalStateException("Cannot add items to confirmed order");
    }
    items.add(item);
}
```

---

## Domain Events

Model significant domain occurrences explicitly. Use for decoupling, audit trails, or triggering side effects.

```java
// Domain event
public record OrderConfirmed(
    Long orderId,
    BigDecimal total,
    LocalDateTime occurredAt
) implements DomainEvent {}

// Raise in aggregate
public void confirm() {
    if (status != OrderStatus.PENDING) throw new IllegalStateException("...");
    status = OrderStatus.CONFIRMED;
    registerEvent(new OrderConfirmed(id, calculateTotal(), LocalDateTime.now()));
}

// Application service publishes; handler reacts
@TransactionalEventListener(phase = AFTER_COMMIT)
public void onOrderConfirmed(OrderConfirmed event) {
    notificationService.sendOrderConfirmation(event.orderId());
}
```

Use domain events when:
- Multiple parts of the system need to react (email, analytics, logging)
- You need an audit trail of what happened
- You want to decouple aggregates from each other

---

## Common DDD Anti-Patterns

### Anemic Domain Model

Entities with only getters/setters; business logic in services. Avoid; use rich domain models instead.

### God Service

Service classes doing everything. Split by aggregate or use case; keep services thin and delegating to domain.

### Leaking Infrastructure into Domain

Domain layer must not depend on `@Entity`, `@Table`, Spring, or database. Prefer interfaces for repositories; keep JPA annotations minimal (or use a separate persistence model if needed).

### Invariant Violations Outside Aggregate

Do not allow external code to bypass aggregate behavior:

```java
// ❌ Bad - Service mutates aggregate state directly
order.setStatus(OrderStatus.SHIPPED);

// ✅ Good - Aggregate enforces rules
order.ship();
```
