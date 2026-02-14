# Clean Code and Anti-Patterns

Reference for java-developer skill. See [SKILL.md](SKILL.md) for core instructions.

---

## Clean Code Principles

### Naming Conventions

```java
// ✅ Good - Clear, descriptive names
public class UserRegistrationService {
    private final UserRepository userRepository;
    private final EmailNotificationService emailService;

    public User registerNewUser(UserRegistrationRequest request) {
        validateUniqueEmail(request.email());
        User user = createUserFromRequest(request);
        sendWelcomeEmail(user);
        return user;
    }
}

// ❌ Bad - Unclear names
public class UsrSvc {
    private final UserRepository repo;
    private final EmailNotificationService svc;

    public User doStuff(UserRegistrationRequest req) {
        chk(req.email());
        User u = make(req);
        send(u);
        return u;
    }
}
```

### Method Length and Complexity

```java
// ❌ Bad - Long method doing too much
public Order processOrder(OrderRequest request) {
    // Validate
    if (request.items().isEmpty()) throw new IllegalArgumentException("No items");
    for (OrderItem item : request.items()) {
        if (item.quantity() <= 0) throw new IllegalArgumentException("Invalid quantity");
        if (item.price().compareTo(BigDecimal.ZERO) <= 0) throw new IllegalArgumentException("Invalid price");
    }

    // Check inventory
    for (OrderItem item : request.items()) {
        int available = inventoryRepository.getAvailableQuantity(item.productId());
        if (available < item.quantity()) throw new OutOfStockException(item.productId());
    }

    // Reserve inventory
    for (OrderItem item : request.items()) {
        inventoryRepository.reserve(item.productId(), item.quantity());
    }

    // Calculate total
    BigDecimal total = BigDecimal.ZERO;
    for (OrderItem item : request.items()) {
        total = total.add(item.price().multiply(BigDecimal.valueOf(item.quantity())));
    }

    // Process payment
    PaymentRequest payment = new PaymentRequest(request.customerId(), total);
    paymentService.process(payment);

    // Create order
    Order order = new Order();
    order.setCustomerId(request.customerId());
    order.setItems(request.items());
    order.setTotal(total);
    order.setStatus(OrderStatus.CONFIRMED);

    return orderRepository.save(order);
}

// ✅ Good - Small, focused methods
@Transactional
public Order processOrder(OrderRequest request) {
    validateOrderRequest(request);
    checkInventoryAvailability(request.items());
    reserveInventory(request.items());

    BigDecimal total = calculateTotal(request.items());
    processPayment(request.customerId(), total);

    return createAndSaveOrder(request, total);
}

private void validateOrderRequest(OrderRequest request) {
    if (request.items().isEmpty()) {
        throw new IllegalArgumentException("Order must contain at least one item");
    }
    request.items().forEach(this::validateOrderItem);
}

private void validateOrderItem(OrderItem item) {
    if (item.quantity() <= 0) {
        throw new IllegalArgumentException("Quantity must be positive");
    }
    if (item.price().compareTo(BigDecimal.ZERO) <= 0) {
        throw new IllegalArgumentException("Price must be positive");
    }
}

private void checkInventoryAvailability(List<OrderItem> items) {
    items.forEach(item -> {
        int available = inventoryRepository.getAvailableQuantity(item.productId());
        if (available < item.quantity()) {
            throw new OutOfStockException(item.productId());
        }
    });
}

private void reserveInventory(List<OrderItem> items) {
    items.forEach(item ->
        inventoryRepository.reserve(item.productId(), item.quantity())
    );
}

private BigDecimal calculateTotal(List<OrderItem> items) {
    return items.stream()
        .map(item -> item.price().multiply(BigDecimal.valueOf(item.quantity())))
        .reduce(BigDecimal.ZERO, BigDecimal::add);
}

private void processPayment(Long customerId, BigDecimal amount) {
    PaymentRequest payment = new PaymentRequest(customerId, amount);
    paymentService.process(payment);
}

private Order createAndSaveOrder(OrderRequest request, BigDecimal total) {
    Order order = Order.builder()
        .customerId(request.customerId())
        .items(request.items())
        .total(total)
        .status(OrderStatus.CONFIRMED)
        .build();

    return orderRepository.save(order);
}
```

### Avoid Magic Numbers and Strings

```java
// ❌ Bad
if (user.getAge() > 18) {
    // Allow access
}

if (order.getStatus().equals("CONFIRMED")) {
    // Process
}

// ✅ Good
public class UserConstants {
    public static final int MINIMUM_AGE = 18;
    public static final int MAXIMUM_LOGIN_ATTEMPTS = 3;
}

public enum OrderStatus {
    PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
}

if (user.getAge() > UserConstants.MINIMUM_AGE) {
    // Allow access
}

if (order.getStatus() == OrderStatus.CONFIRMED) {
    // Process
}
```

---

## Anti-Patterns to Avoid

### Anemic Domain Model

```java
// ❌ Bad - Anemic model (just getters/setters)
@Entity
public class Order {
    private Long id;
    private OrderStatus status;
    private List<OrderItem> items;

    // Only getters and setters, no behavior
}

@Service
public class OrderService {
    public void shipOrder(Order order) {
        order.setStatus(OrderStatus.SHIPPED);
        order.setShippedAt(LocalDateTime.now());
    }
}

// ✅ Good - Rich domain model
@Entity
public class Order {
    private Long id;
    private OrderStatus status;
    private List<OrderItem> items;

    public void ship() {
        if (status != OrderStatus.CONFIRMED) {
            throw new IllegalStateException("Can only ship confirmed orders");
        }
        status = OrderStatus.SHIPPED;
        shippedAt = LocalDateTime.now();
    }
}

@Service
public class OrderService {
    public void shipOrder(Long orderId) {
        Order order = findById(orderId);
        order.ship(); // Business logic in domain
        orderRepository.save(order);
    }
}
```

### God Service

```java
// ❌ Bad - Service doing everything
@Service
public class UserService {
    public User createUser(CreateUserRequest request) { }
    public void deleteUser(Long id) { }
    public Order placeOrder(OrderRequest request) { }
    public void sendEmail(String to, String subject) { }
    public void processPayment(PaymentRequest request) { }
    public Report generateReport(ReportType type) { }
    // ... 50 more methods
}

// ✅ Good - Separated concerns
@Service
public class UserService {
    public User createUser(CreateUserRequest request) { }
    public void deleteUser(Long id) { }
}

@Service
public class OrderService {
    public Order placeOrder(OrderRequest request) { }
}

@Service
public class EmailService {
    public void sendEmail(String to, String subject) { }
}

@Service
public class PaymentService {
    public void processPayment(PaymentRequest request) { }
}
```

### Transaction Anti-Patterns

```java
// ❌ Bad - Transaction too large
@Transactional
public void processMonthlyBatch() {
    List<Order> orders = orderRepository.findAll(); // Could be millions
    for (Order order : orders) {
        // Long processing
    }
    // Transaction held for hours!
}

// ✅ Good - Batch processing
public void processMonthlyBatch() {
    int pageSize = 100;
    int page = 0;

    Page<Order> orderPage;
    do {
        orderPage = orderRepository.findAll(PageRequest.of(page, pageSize));
        processOrderBatch(orderPage.getContent());
        page++;
    } while (orderPage.hasNext());
}

@Transactional
private void processOrderBatch(List<Order> orders) {
    orders.forEach(this::processOrder);
}
```

---

## Quick Reference Checklist

**Architecture:**

- [ ] Clear layered architecture (presentation, application, domain, infrastructure)
- [ ] Domain logic in domain models, not services
- [ ] Dependencies point inward (DIP)
- [ ] Each class has single responsibility
- [ ] Interfaces used for abstraction

**REST API:**

- [ ] Proper HTTP methods (GET, POST, PUT, PATCH, DELETE)
- [ ] Appropriate status codes
- [ ] Request/response DTOs separate from entities
- [ ] Validation on DTOs
- [ ] Pagination for lists
- [ ] Global exception handling

**Data Access:**

- [ ] Repository interfaces in domain layer
- [ ] Proper fetch strategies (avoid N+1)
- [ ] Transactions at service layer
- [ ] Read-only transactions for queries
- [ ] Query optimization

**Security:**

- [ ] All endpoints authenticated/authorized
- [ ] Passwords encrypted
- [ ] JWT tokens properly validated
- [ ] CSRF protection enabled
- [ ] Method security annotations

**Testing:**

- [ ] Lots of unit tests (70%+)
- [ ] Some integration tests (20%)
- [ ] Few E2E tests (10%)
- [ ] Tests are independent
- [ ] Tests are fast
- [ ] Meaningful test names

**Cross-Cutting:**

- [ ] Structured logging with context
- [ ] Metrics exposed
- [ ] Health checks implemented
- [ ] Distributed tracing configured
- [ ] Error messages user-friendly

**Code Quality:**

- [ ] Descriptive names
- [ ] Short, focused methods
- [ ] No magic numbers/strings
- [ ] Proper exception handling
- [ ] Configuration externalized
