# Cross-Cutting, Configuration, and Performance

Reference for java-developer skill. See [SKILL.md](SKILL.md) for core instructions.

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
