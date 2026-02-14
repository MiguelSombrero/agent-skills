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

#### Single Responsibility Principle

**Each class should have one reason to change.**

```java
// ❌ Bad - Multiple responsibilities
@RestController
@RequestMapping("/api/users")
public class UserController {

    public User createUser(UserRequest request) {
        // Validation
        if (request.email() == null || !request.email().contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }

        // Business logic
        User user = new User();
        user.setEmail(request.email());
        user.setPasswordHash(BCrypt.hashpw(request.password(), BCrypt.gensalt()));

        // Database access
        EntityManager em = entityManagerFactory.createEntityManager();
        em.persist(user);

        // Email sending
        sendWelcomeEmail(user.getEmail());

        return user;
    }
}

// ✅ Good - Separated concerns
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping
    public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest request) {
        User user = userService.createUser(request);
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(UserResponse.from(user));
    }
}

@Service
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final EmailService emailService;

    public UserService(UserRepository userRepository,
                      PasswordEncoder passwordEncoder,
                      EmailService emailService) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.emailService = emailService;
    }

    @Transactional
    public User createUser(CreateUserRequest request) {
        User user = User.builder()
            .email(request.email())
            .passwordHash(passwordEncoder.encode(request.password()))
            .name(request.name())
            .build();

        User savedUser = userRepository.save(user);
        emailService.sendWelcomeEmail(savedUser.getEmail());

        return savedUser;
    }
}
```

#### Dependency Inversion Principle

**Depend on abstractions, not concretions.**

```java
// ✅ Good - Program to interface
public interface NotificationService {
    void send(String recipient, String message);
}

@Service
public class EmailNotificationService implements NotificationService {
    @Override
    public void send(String recipient, String message) {
        // Email implementation
    }
}

@Service
public class SmsNotificationService implements NotificationService {
    @Override
    public void send(String recipient, String message) {
        // SMS implementation
    }
}

// Service depends on abstraction
@Service
public class OrderService {
    private final NotificationService notificationService;

    public OrderService(@Qualifier("emailNotificationService")
                       NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    public void placeOrder(Order order) {
        // Business logic
        notificationService.send(order.getCustomerEmail(), "Order placed");
    }
}
```

### Test-Driven Development (TDD)

**Red-Green-Refactor Cycle**

```java
// 1. RED - Write failing test first
@Test
void shouldCalculateOrderTotal() {
    // Arrange
    Order order = new Order();
    order.addItem(new OrderItem(new BigDecimal("10.00"), 2));
    order.addItem(new OrderItem(new BigDecimal("5.00"), 1));

    // Act
    BigDecimal total = order.calculateTotal();

    // Assert
    assertThat(total).isEqualByComparingTo(new BigDecimal("25.00"));
}

// 2. GREEN - Implement minimum code to pass
public class Order {
    private List<OrderItem> items = new ArrayList<>();

    public void addItem(OrderItem item) {
        items.add(item);
    }

    public BigDecimal calculateTotal() {
        return items.stream()
            .map(item -> item.price().multiply(BigDecimal.valueOf(item.quantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

// 3. REFACTOR - Improve code while keeping tests green
public class Order {
    private final List<OrderItem> items = new ArrayList<>();

    public void addItem(OrderItem item) {
        requireNonNull(item, "Item cannot be null");
        items.add(item);
    }

    public BigDecimal calculateTotal() {
        return items.stream()
            .map(OrderItem::subtotal)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

public record OrderItem(BigDecimal price, int quantity) {
    public OrderItem {
        requireNonNull(price, "Price cannot be null");
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
    }

    public BigDecimal subtotal() {
        return price.multiply(BigDecimal.valueOf(quantity));
    }
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
├── api/                          # Presentation Layer
│   ├── controller/
│   │   ├── UserController.java
│   │   └── OrderController.java
│   ├── dto/
│   │   ├── request/
│   │   │   ├── CreateUserRequest.java
│   │   │   └── UpdateUserRequest.java
│   │   └── response/
│   │       ├── UserResponse.java
│   │       └── OrderResponse.java
│   └── mapper/
│       └── UserMapper.java
│
├── application/                  # Application Layer
│   ├── service/
│   │   ├── UserService.java
│   │   └── OrderService.java
│   └── usecase/
│       └── PlaceOrderUseCase.java
│
├── domain/                       # Domain Layer
│   ├── model/
│   │   ├── User.java
│   │   ├── Order.java
│   │   └── OrderItem.java
│   ├── repository/               # Interfaces
│   │   ├── UserRepository.java
│   │   └── OrderRepository.java
│   ├── exception/
│   │   ├── UserNotFoundException.java
│   │   └── OrderValidationException.java
│   └── valueobject/
│       ├── Email.java
│       └── Money.java
│
├── infrastructure/               # Infrastructure Layer
│   ├── persistence/
│   │   ├── entity/
│   │   │   ├── UserEntity.java
│   │   │   └── OrderEntity.java
│   │   ├── repository/
│   │   │   ├── JpaUserRepository.java
│   │   │   └── JpaOrderRepository.java
│   │   └── mapper/
│   │       └── UserEntityMapper.java
│   ├── external/
│   │   ├── PaymentGatewayClient.java
│   │   └── EmailServiceClient.java
│   └── config/
│       ├── DatabaseConfig.java
│       └── SecurityConfig.java
│
└── shared/                       # Cross-cutting
    ├── exception/
    │   └── GlobalExceptionHandler.java
    ├── validation/
    │   └── EmailValidator.java
    └── util/
        └── DateUtils.java
```

### Domain Model (Rich Domain)

```java
// ✅ Good - Rich domain model with behavior
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();

    @Column(nullable = false)
    private BigDecimal total;

    private LocalDateTime placedAt;
    private LocalDateTime shippedAt;

    // Factory method
    public static Order create(List<OrderItem> items) {
        Order order = new Order();
        order.status = OrderStatus.PENDING;
        order.placedAt = LocalDateTime.now();
        items.forEach(order::addItem);
        return order;
    }

    // Business logic in domain
    public void addItem(OrderItem item) {
        requireNonNull(item, "Item cannot be null");
        items.add(item);
        item.setOrder(this);
        recalculateTotal();
    }

    public void removeItem(OrderItem item) {
        items.remove(item);
        item.setOrder(null);
        recalculateTotal();
    }

    public void ship() {
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("Can only ship pending orders");
        }
        status = OrderStatus.SHIPPED;
        shippedAt = LocalDateTime.now();
    }

    public void cancel() {
        if (status == OrderStatus.DELIVERED) {
            throw new IllegalStateException("Cannot cancel delivered order");
        }
        status = OrderStatus.CANCELLED;
    }

    public boolean canBeCancelled() {
        return status != OrderStatus.DELIVERED && status != OrderStatus.CANCELLED;
    }

    private void recalculateTotal() {
        total = items.stream()
            .map(OrderItem::getSubtotal)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    // No setters that bypass business logic
    // Getters omitted for brevity
}

public enum OrderStatus {
    PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
}
```

---

## REST API Design

### Controller Best Practices

```java
@RestController
@RequestMapping("/api/v1/users")
@Validated
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public ResponseEntity<Page<UserResponse>> getAllUsers(
            @PageableDefault(size = 20, sort = "createdAt", direction = Sort.Direction.DESC)
            Pageable pageable) {
        Page<User> users = userService.findAll(pageable);
        Page<UserResponse> response = users.map(UserResponse::from);
        return ResponseEntity.ok(response);
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUserById(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(UserResponse.from(user));
    }

    @PostMapping
    public ResponseEntity<UserResponse> createUser(
            @Valid @RequestBody CreateUserRequest request) {
        User user = userService.createUser(request);

        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(user.getId())
            .toUri();

        return ResponseEntity.created(location)
            .body(UserResponse.from(user));
    }

    @PutMapping("/{id}")
    public ResponseEntity<UserResponse> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UpdateUserRequest request) {
        User user = userService.updateUser(id, request);
        return ResponseEntity.ok(UserResponse.from(user));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }

    @PatchMapping("/{id}/activate")
    public ResponseEntity<Void> activateUser(@PathVariable Long id) {
        userService.activateUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

### DTOs and Validation

```java
// Request DTO
public record CreateUserRequest(
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be between 2 and 100 characters")
    String name,

    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    String email,

    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    @Pattern(regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d).*$",
             message = "Password must contain uppercase, lowercase and digit")
    String password
) {
    // Custom validation
    public CreateUserRequest {
        if (email != null && email.contains("+")) {
            throw new IllegalArgumentException("Email cannot contain '+' character");
        }
    }
}

// Response DTO
public record UserResponse(
    Long id,
    String name,
    String email,
    LocalDateTime createdAt,
    boolean active
) {
    public static UserResponse from(User user) {
        return new UserResponse(
            user.getId(),
            user.getName(),
            user.getEmail(),
            user.getCreatedAt(),
            user.isActive()
        );
    }
}

// Custom validator
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueEmailValidator.class)
public @interface UniqueEmail {
    String message() default "Email already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

@Component
public class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {
    private final UserRepository userRepository;

    public UniqueEmailValidator(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        return email != null && !userRepository.existsByEmail(email);
    }
}
```

### Error Handling

```java
// Custom exceptions
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(Long id) {
        super("User not found with id: " + id);
    }
}

public class EmailAlreadyExistsException extends RuntimeException {
    public EmailAlreadyExistsException(String email) {
        super("Email already exists: " + email);
    }
}

// Global exception handler
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        log.warn("User not found: {}", ex.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            "User not found",
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(EmailAlreadyExistsException.class)
    public ResponseEntity<ErrorResponse> handleEmailAlreadyExists(EmailAlreadyExistsException ex) {
        log.warn("Email conflict: {}", ex.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.CONFLICT.value(),
            "Email already exists",
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidationErrors(
            MethodArgumentNotValidException ex) {
        Map<String, String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                error -> error.getDefaultMessage() != null ? error.getDefaultMessage() : "Invalid value"
            ));

        ValidationErrorResponse response = new ValidationErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed",
            errors,
            LocalDateTime.now()
        );
        return ResponseEntity.badRequest().body(response);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        log.error("Unexpected error", ex);
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Internal server error",
            "An unexpected error occurred",
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}

public record ErrorResponse(
    int status,
    String error,
    String message,
    LocalDateTime timestamp
) {}

public record ValidationErrorResponse(
    int status,
    String error,
    Map<String, String> fieldErrors,
    LocalDateTime timestamp
) {}
```

---

## Spring Data JPA

### Repository Pattern

```java
// Repository interface
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);

    boolean existsByEmail(String email);

    List<User> findByActiveTrue();

    @Query("SELECT u FROM User u WHERE u.createdAt > :date")
    List<User> findUsersCreatedAfter(@Param("date") LocalDateTime date);

    @Query(value = "SELECT * FROM users u WHERE u.email LIKE :pattern", nativeQuery = true)
    List<User> findByEmailPattern(@Param("pattern") String pattern);

    @Modifying
    @Query("UPDATE User u SET u.active = :active WHERE u.id = :id")
    void updateActiveStatus(@Param("id") Long id, @Param("active") boolean active);
}

// Custom repository for complex queries
public interface UserRepositoryCustom {
    List<User> findByComplexCriteria(UserSearchCriteria criteria);
}

@Repository
public class UserRepositoryCustomImpl implements UserRepositoryCustom {
    private final EntityManager entityManager;

    public UserRepositoryCustomImpl(EntityManager entityManager) {
        this.entityManager = entityManager;
    }

    @Override
    public List<User> findByComplexCriteria(UserSearchCriteria criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> user = query.from(User.class);

        List<Predicate> predicates = new ArrayList<>();

        if (criteria.name() != null) {
            predicates.add(cb.like(user.get("name"), "%" + criteria.name() + "%"));
        }

        if (criteria.email() != null) {
            predicates.add(cb.equal(user.get("email"), criteria.email()));
        }

        if (criteria.active() != null) {
            predicates.add(cb.equal(user.get("active"), criteria.active()));
        }

        query.where(predicates.toArray(new Predicate[0]));

        return entityManager.createQuery(query).getResultList();
    }
}

public record UserSearchCriteria(String name, String email, Boolean active) {}
```

### Entity Relationships

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    // One-to-Many
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();

    // Many-to-Many
    @ManyToMany
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();

    // Helper methods
    public void addOrder(Order order) {
        orders.add(order);
        order.setUser(this);
    }

    public void removeOrder(Order order) {
        orders.remove(order);
        order.setUser(null);
    }

    public void addRole(Role role) {
        roles.add(role);
        role.getUsers().add(this);
    }

    public void removeRole(Role role) {
        roles.remove(role);
        role.getUsers().remove(this);
    }
}

@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // Many-to-One
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @Column(nullable = false)
    private BigDecimal total;

    // Avoid N+1 queries with proper fetch strategies
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<OrderItem> items = new ArrayList<>();
}

// DTO projection to avoid fetching entire entity
public interface UserProjection {
    Long getId();
    String getName();
    String getEmail();
}

public interface UserRepository extends JpaRepository<User, Long> {
    List<UserProjection> findAllProjectedBy();

    @Query("SELECT u.id as id, u.name as name, u.email as email FROM User u")
    List<UserProjection> findAllWithProjection();
}
```

### Transactions

```java
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;

    public OrderService(OrderRepository orderRepository,
                       InventoryService inventoryService,
                       PaymentService paymentService) {
        this.orderRepository = orderRepository;
        this.inventoryService = inventoryService;
        this.paymentService = paymentService;
    }

    @Transactional
    public Order placeOrder(CreateOrderRequest request) {
        // All operations in same transaction
        Order order = Order.create(request.items());

        // Reserve inventory
        inventoryService.reserveItems(order.getItems());

        // Process payment
        paymentService.processPayment(order.getTotal());

        // Save order
        return orderRepository.save(order);
    }

    @Transactional(readOnly = true)
    public Order findById(Long id) {
        return orderRepository.findById(id)
            .orElseThrow(() -> new OrderNotFoundException(id));
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logOrderEvent(Long orderId, String event) {
        // Always creates new transaction
        // Commits even if parent transaction rolls back
        OrderLog log = new OrderLog(orderId, event);
        orderLogRepository.save(log);
    }
}
```

---

## Spring Security

### Security Configuration

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf
                .ignoringRequestMatchers("/api/public/**")
            )
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.GET, "/api/users/**").hasAnyRole("USER", "ADMIN")
                .requestMatchers(HttpMethod.POST, "/api/users").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtAuthenticationConverter()))
            );

        return http.build();
    }

    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter grantedAuthoritiesConverter =
            new JwtGrantedAuthoritiesConverter();
        grantedAuthoritiesConverter.setAuthorityPrefix("ROLE_");
        grantedAuthoritiesConverter.setAuthoritiesClaimName("roles");

        JwtAuthenticationConverter jwtAuthenticationConverter =
            new JwtAuthenticationConverter();
        jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(
            grantedAuthoritiesConverter
        );

        return jwtAuthenticationConverter;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### JWT Authentication

```java
@Service
public class AuthenticationService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtTokenProvider tokenProvider;

    public AuthenticationService(UserRepository userRepository,
                                PasswordEncoder passwordEncoder,
                                JwtTokenProvider tokenProvider) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.tokenProvider = tokenProvider;
    }

    public AuthenticationResponse authenticate(AuthenticationRequest request) {
        User user = userRepository.findByEmail(request.email())
            .orElseThrow(() -> new BadCredentialsException("Invalid credentials"));

        if (!passwordEncoder.matches(request.password(), user.getPasswordHash())) {
            throw new BadCredentialsException("Invalid credentials");
        }

        String token = tokenProvider.generateToken(user);

        return new AuthenticationResponse(
            token,
            "Bearer",
            tokenProvider.getExpirationTime(),
            UserResponse.from(user)
        );
    }
}

@Component
public class JwtTokenProvider {
    private final SecretKey key;
    private final long expirationTime;

    public JwtTokenProvider(@Value("${jwt.secret}") String secret,
                           @Value("${jwt.expiration}") long expirationTime) {
        this.key = Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
        this.expirationTime = expirationTime;
    }

    public String generateToken(User user) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + expirationTime);

        return Jwts.builder()
            .setSubject(user.getId().toString())
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .claim("email", user.getEmail())
            .claim("roles", user.getRoles().stream()
                .map(Role::getName)
                .toList())
            .signWith(key)
            .compact();
    }

    public Long getUserIdFromToken(String token) {
        Claims claims = Jwts.parserBuilder()
            .setSigningKey(key)
            .build()
            .parseClaimsJws(token)
            .getBody();

        return Long.parseLong(claims.getSubject());
    }

    public long getExpirationTime() {
        return expirationTime;
    }
}
```

### Method Security

```java
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @PreAuthorize("hasRole('ADMIN')")
    public User createUser(CreateUserRequest request) {
        // Only admins can create users
        return userRepository.save(User.from(request));
    }

    @PreAuthorize("hasRole('ADMIN') or #id == authentication.principal.id")
    public User updateUser(Long id, UpdateUserRequest request) {
        // Admins can update any user, users can update themselves
        User user = findById(id);
        user.update(request);
        return userRepository.save(user);
    }

    @PostAuthorize("returnObject.email == authentication.principal.username or hasRole('ADMIN')")
    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
}
```

---

## Cross-Cutting Concerns

### Logging

```java
// SLF4J with Logback
@Slf4j
@Service
public class OrderService {
    private final OrderRepository orderRepository;

    public Order placeOrder(CreateOrderRequest request) {
        log.info("Placing order for user: {}", request.userId());
        log.debug("Order details: {}", request);

        try {
            Order order = processOrder(request);
            log.info("Order placed successfully: orderId={}", order.getId());
            return order;
        } catch (Exception e) {
            log.error("Failed to place order for user: {}", request.userId(), e);
            throw e;
        }
    }
}

// Structured logging with MDC
@Component
public class RequestLoggingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                   HttpServletResponse response,
                                   FilterChain filterChain) throws ServletException, IOException {
        String requestId = UUID.randomUUID().toString();
        MDC.put("requestId", requestId);
        MDC.put("method", request.getMethod());
        MDC.put("path", request.getRequestURI());

        try {
            filterChain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}

// logback-spring.xml
/*
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeMdcKeyName>requestId</includeMdcKeyName>
            <includeMdcKeyName>userId</includeMdcKeyName>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
*/
```

### Monitoring with Micrometer

```java
@Configuration
public class MetricsConfig {

    @Bean
    public MeterRegistryCustomizer<MeterRegistry> metricsCommonTags() {
        return registry -> registry.config()
            .commonTags(
                "application", "my-app",
                "environment", System.getenv("SPRING_PROFILES_ACTIVE")
            );
    }
}

@Service
@Slf4j
public class OrderService {
    private final OrderRepository orderRepository;
    private final Counter orderCounter;
    private final Timer orderProcessingTimer;

    public OrderService(OrderRepository orderRepository, MeterRegistry meterRegistry) {
        this.orderRepository = orderRepository;
        this.orderCounter = Counter.builder("orders.placed")
            .description("Number of orders placed")
            .tag("type", "online")
            .register(meterRegistry);
        this.orderProcessingTimer = Timer.builder("orders.processing.time")
            .description("Order processing time")
            .register(meterRegistry);
    }

    public Order placeOrder(CreateOrderRequest request) {
        return orderProcessingTimer.record(() -> {
            Order order = processOrder(request);
            orderCounter.increment();
            return order;
        });
    }
}

// Expose metrics
// application.yml
/*
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
*/
```

### Distributed Tracing

```java
// Spring Boot 3+ with Micrometer Tracing
// Dependencies: micrometer-tracing-bridge-brave, zipkin-reporter-brave

@Configuration
public class TracingConfig {

    @Bean
    public Sampler defaultSampler() {
        return Sampler.ALWAYS_SAMPLE;
    }
}

@Service
public class OrderService {
    private final Tracer tracer;

    public OrderService(Tracer tracer) {
        this.tracer = tracer;
    }

    public Order placeOrder(CreateOrderRequest request) {
        Span span = tracer.nextSpan().name("place-order").start();
        span.tag("userId", request.userId().toString());
        span.tag("itemCount", String.valueOf(request.items().size()));

        try (Tracer.SpanInScope ws = tracer.withSpan(span)) {
            return processOrder(request);
        } catch (Exception e) {
            span.tag("error", e.getMessage());
            throw e;
        } finally {
            span.end();
        }
    }
}

// application.yml
/*
management:
  tracing:
    sampling:
      probability: 1.0
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans
*/
```

### Actuator Health Checks

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    private final DataSource dataSource;

    public DatabaseHealthIndicator(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public Health health() {
        try (Connection connection = dataSource.getConnection()) {
            if (connection.isValid(1)) {
                return Health.up()
                    .withDetail("database", "PostgreSQL")
                    .withDetail("validationQuery", "SELECT 1")
                    .build();
            }
        } catch (SQLException e) {
            return Health.down()
                .withException(e)
                .build();
        }

        return Health.down().build();
    }
}

// Custom health endpoint
@RestController
@RequestMapping("/api/health")
public class CustomHealthController {
    private final HealthEndpoint healthEndpoint;

    public CustomHealthController(HealthEndpoint healthEndpoint) {
        this.healthEndpoint = healthEndpoint;
    }

    @GetMapping
    public ResponseEntity<HealthComponent> health() {
        HealthComponent health = healthEndpoint.health();
        return ResponseEntity.ok(health);
    }
}
```

---

## Testing

### Unit Tests (Lots of These!)

```java
// Service unit test - mock dependencies
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private PasswordEncoder passwordEncoder;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private UserService userService;

    @Test
    void shouldCreateUserSuccessfully() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest(
            "John Doe",
            "john@example.com",
            "Password123"
        );

        User expectedUser = User.builder()
            .id(1L)
            .name(request.name())
            .email(request.email())
            .build();

        when(passwordEncoder.encode(request.password()))
            .thenReturn("encodedPassword");
        when(userRepository.save(any(User.class)))
            .thenReturn(expectedUser);

        // Act
        User result = userService.createUser(request);

        // Assert
        assertThat(result).isNotNull();
        assertThat(result.getName()).isEqualTo("John Doe");
        assertThat(result.getEmail()).isEqualTo("john@example.com");

        verify(userRepository).save(any(User.class));
        verify(emailService).sendWelcomeEmail("john@example.com");
    }

    @Test
    void shouldThrowExceptionWhenEmailAlreadyExists() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest(
            "John Doe",
            "existing@example.com",
            "Password123"
        );

        when(userRepository.existsByEmail(request.email())).thenReturn(true);

        // Act & Assert
        assertThatThrownBy(() -> userService.createUser(request))
            .isInstanceOf(EmailAlreadyExistsException.class)
            .hasMessageContaining("existing@example.com");

        verify(userRepository, never()).save(any());
        verify(emailService, never()).sendWelcomeEmail(any());
    }
}

// Domain model test - no mocks needed
class OrderTest {

    @Test
    void shouldCalculateTotalCorrectly() {
        // Arrange
        Order order = new Order();
        order.addItem(new OrderItem(new BigDecimal("10.00"), 2));
        order.addItem(new OrderItem(new BigDecimal("5.50"), 3));

        // Act
        BigDecimal total = order.getTotal();

        // Assert
        assertThat(total).isEqualByComparingTo(new BigDecimal("36.50"));
    }

    @Test
    void shouldNotAllowShippingOfCancelledOrder() {
        // Arrange
        Order order = new Order();
        order.cancel();

        // Act & Assert
        assertThatThrownBy(() -> order.ship())
            .isInstanceOf(IllegalStateException.class)
            .hasMessageContaining("Cannot ship cancelled order");
    }

    @Test
    void shouldTransitionThroughValidStates() {
        // Arrange
        Order order = new Order();

        // Act & Assert
        assertThat(order.getStatus()).isEqualTo(OrderStatus.PENDING);

        order.confirm();
        assertThat(order.getStatus()).isEqualTo(OrderStatus.CONFIRMED);

        order.ship();
        assertThat(order.getStatus()).isEqualTo(OrderStatus.SHIPPED);

        order.deliver();
        assertThat(order.getStatus()).isEqualTo(OrderStatus.DELIVERED);
    }
}
```

### Integration Tests (Some of These)

```java
// Repository integration test with TestContainers
@DataJpaTest
@Testcontainers
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class UserRepositoryIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void shouldSaveAndRetrieveUser() {
        // Arrange
        User user = User.builder()
            .name("John Doe")
            .email("john@example.com")
            .passwordHash("hashedPassword")
            .build();

        // Act
        User saved = userRepository.save(user);
        entityManager.flush();
        entityManager.clear();

        User found = userRepository.findById(saved.getId()).orElseThrow();

        // Assert
        assertThat(found.getId()).isEqualTo(saved.getId());
        assertThat(found.getName()).isEqualTo("John Doe");
        assertThat(found.getEmail()).isEqualTo("john@example.com");
    }

    @Test
    void shouldFindUserByEmail() {
        // Arrange
        User user = User.builder()
            .name("Jane Doe")
            .email("jane@example.com")
            .passwordHash("hashedPassword")
            .build();
        entityManager.persist(user);
        entityManager.flush();

        // Act
        Optional<User> found = userRepository.findByEmail("jane@example.com");

        // Assert
        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("Jane Doe");
    }

    @Test
    void shouldReturnEmptyWhenUserNotFound() {
        // Act
        Optional<User> found = userRepository.findByEmail("nonexistent@example.com");

        // Assert
        assertThat(found).isEmpty();
    }
}

// Service integration test
@SpringBootTest
@Testcontainers
class UserServiceIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserService userService;

    @Autowired
    private UserRepository userRepository;

    @MockBean
    private EmailService emailService;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void shouldCreateUserAndSendEmail() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest(
            "John Doe",
            "john@example.com",
            "Password123"
        );

        // Act
        User user = userService.createUser(request);

        // Assert
        assertThat(user.getId()).isNotNull();
        assertThat(user.getName()).isEqualTo("John Doe");

        // Verify user in database
        User dbUser = userRepository.findById(user.getId()).orElseThrow();
        assertThat(dbUser.getEmail()).isEqualTo("john@example.com");

        // Verify email sent
        verify(emailService).sendWelcomeEmail("john@example.com");
    }
}
```

### Controller Tests (WebMvcTest)

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void shouldCreateUser() throws Exception {
        // Arrange
        CreateUserRequest request = new CreateUserRequest(
            "John Doe",
            "john@example.com",
            "Password123"
        );

        User user = User.builder()
            .id(1L)
            .name(request.name())
            .email(request.email())
            .createdAt(LocalDateTime.now())
            .build();

        when(userService.createUser(any(CreateUserRequest.class))).thenReturn(user);

        // Act & Assert
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"))
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("John Doe"))
            .andExpect(jsonPath("$.email").value("john@example.com"));

        verify(userService).createUser(any(CreateUserRequest.class));
    }

    @Test
    void shouldReturnBadRequestForInvalidInput() throws Exception {
        // Arrange
        CreateUserRequest request = new CreateUserRequest(
            "",  // Invalid: empty name
            "invalid-email",  // Invalid: bad email format
            "123"  // Invalid: too short password
        );

        // Act & Assert
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.fieldErrors.name").exists())
            .andExpect(jsonPath("$.fieldErrors.email").exists())
            .andExpect(jsonPath("$.fieldErrors.password").exists());

        verify(userService, never()).createUser(any());
    }

    @Test
    void shouldGetUserById() throws Exception {
        // Arrange
        User user = User.builder()
            .id(1L)
            .name("John Doe")
            .email("john@example.com")
            .build();

        when(userService.findById(1L)).thenReturn(user);

        // Act & Assert
        mockMvc.perform(get("/api/v1/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("John Doe"));
    }

    @Test
    void shouldReturn404WhenUserNotFound() throws Exception {
        // Arrange
        when(userService.findById(999L))
            .thenThrow(new UserNotFoundException(999L));

        // Act & Assert
        mockMvc.perform(get("/api/v1/users/999"))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.error").value("User not found"));
    }
}
```

### E2E Tests (Few of These)

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class UserE2ETest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void shouldCreateAndRetrieveUser() {
        // Create user
        CreateUserRequest request = new CreateUserRequest(
            "John Doe",
            "john@example.com",
            "Password123"
        );

        ResponseEntity<UserResponse> createResponse = restTemplate.postForEntity(
            "/api/v1/users",
            request,
            UserResponse.class
        );

        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(createResponse.getBody()).isNotNull();
        Long userId = createResponse.getBody().id();

        // Retrieve user
        ResponseEntity<UserResponse> getResponse = restTemplate.getForEntity(
            "/api/v1/users/" + userId,
            UserResponse.class
        );

        assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(getResponse.getBody().name()).isEqualTo("John Doe");
        assertThat(getResponse.getBody().email()).isEqualTo("john@example.com");
    }

    @Test
    void shouldNotCreateUserWithDuplicateEmail() {
        // Create first user
        CreateUserRequest request = new CreateUserRequest(
            "John Doe",
            "john@example.com",
            "Password123"
        );

        restTemplate.postForEntity("/api/v1/users", request, UserResponse.class);

        // Try to create second user with same email
        ResponseEntity<ErrorResponse> response = restTemplate.postForEntity(
            "/api/v1/users",
            request,
            ErrorResponse.class
        );

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CONFLICT);
        assertThat(response.getBody().error()).contains("Email already exists");
    }
}
```

### Test Configuration

```java
// Test configuration
@TestConfiguration
public class TestConfig {

    @Bean
    @Primary
    public PasswordEncoder passwordEncoder() {
        // Use faster encoder for tests
        return new PasswordEncoder() {
            @Override
            public String encode(CharSequence rawPassword) {
                return "encoded_" + rawPassword;
            }

            @Override
            public boolean matches(CharSequence rawPassword, String encodedPassword) {
                return encodedPassword.equals("encoded_" + rawPassword);
            }
        };
    }
}

// Base test class
@SpringBootTest
@Testcontainers
@ActiveProfiles("test")
public abstract class IntegrationTestBase {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
}

// Extend in tests
class UserServiceIntegrationTest extends IntegrationTestBase {
    // Test methods...
}
```

---

## Configuration Management

### Application Properties

```yaml
# application.yml
spring:
  application:
    name: my-app

  datasource:
    url: ${DATABASE_URL:jdbc:postgresql://localhost:5432/myapp}
    username: ${DATABASE_USER:postgres}
    password: ${DATABASE_PASSWORD:postgres}
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000

  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
        jdbc:
          batch_size: 20
        order_inserts: true
        order_updates: true
    show-sql: false

  jackson:
    serialization:
      write-dates-as-timestamps: false
    default-property-inclusion: non_null

server:
  port: 8080
  error:
    include-message: always
    include-binding-errors: always

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true

logging:
  level:
    root: INFO
    com.example.app: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE

---
# application-dev.yml
spring:
  config:
    activate:
      on-profile: dev

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

logging:
  level:
    root: DEBUG

---
# application-prod.yml
spring:
  config:
    activate:
      on-profile: prod

  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false

logging:
  level:
    root: WARN
    com.example.app: INFO
```

### Configuration Classes

```java
@Configuration
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private Security security = new Security();
    private Jwt jwt = new Jwt();

    public static class Security {
        private List<String> allowedOrigins = new ArrayList<>();
        private int sessionTimeout = 3600;

        // Getters and setters
    }

    public static class Jwt {
        private String secret;
        private long expiration = 86400000; // 24 hours

        // Getters and setters
    }

    // Getters and setters
}

// Enable in main application
@SpringBootApplication
@EnableConfigurationProperties(AppProperties.class)
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// Use in services
@Service
public class AuthenticationService {
    private final AppProperties appProperties;

    public AuthenticationService(AppProperties appProperties) {
        this.appProperties = appProperties;
    }

    public String generateToken(User user) {
        return Jwts.builder()
            .setSubject(user.getId().toString())
            .setExpiration(new Date(System.currentTimeMillis() +
                appProperties.getJwt().getExpiration()))
            .signWith(Keys.hmacShaKeyFor(
                appProperties.getJwt().getSecret().getBytes()))
            .compact();
    }
}
```

---

## Performance Optimization

### Query Optimization

```java
// ❌ Bad - N+1 query problem
public List<OrderDTO> getAllOrders() {
    List<Order> orders = orderRepository.findAll();
    return orders.stream()
        .map(order -> {
            User user = userRepository.findById(order.getUserId()).orElseThrow();
            return new OrderDTO(order, user);
        })
        .toList();
}

// ✅ Good - Fetch join
@Query("SELECT o FROM Order o JOIN FETCH o.user")
List<Order> findAllWithUser();

// ✅ Good - Entity graph
@EntityGraph(attributePaths = {"user", "items"})
List<Order> findAll();

// ✅ Good - DTO projection
@Query("""
    SELECT new com.example.app.dto.OrderDTO(
        o.id, o.total, o.createdAt, u.name, u.email
    )
    FROM Order o
    JOIN o.user u
""")
List<OrderDTO> findAllOrderDTOs();
```

### Caching

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("users", "orders");
    }
}

@Service
public class UserService {

    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }

    @CachePut(value = "users", key = "#result.id")
    public User updateUser(Long id, UpdateUserRequest request) {
        User user = findById(id);
        user.update(request);
        return userRepository.save(user);
    }

    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }

    @CacheEvict(value = "users", allEntries = true)
    public void clearCache() {
        // Clears all user cache entries
    }
}
```

### Async Processing

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(25);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class EmailService {

    @Async
    public CompletableFuture<Void> sendWelcomeEmail(String email) {
        // Long-running email sending
        return CompletableFuture.completedFuture(null);
    }

    @Async
    public void sendOrderConfirmation(Order order) {
        // Fire and forget
    }
}

// Use in service
@Service
public class UserService {
    private final EmailService emailService;

    public User createUser(CreateUserRequest request) {
        User user = userRepository.save(User.from(request));

        // Non-blocking email sending
        emailService.sendWelcomeEmail(user.getEmail());

        return user;
    }
}
```

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
