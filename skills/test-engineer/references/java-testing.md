Reference for test-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.

---

## Java/Spring Boot Testing

### Frameworks

- **JUnit 5** - Java testing framework
- **Mockito** - Mocking framework
- **Spring Boot Test** - Integration testing support
- **TestContainers** - Docker-based integration testing
- **MockMvc** - Spring MVC testing
- **RestAssured** - REST API testing

### Unit Testing with JUnit 5

#### Basic Structure

```java
// CalculatorTest.java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

@DisplayName("Calculator Tests")
class CalculatorTest {
    private Calculator calculator;

    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }

    @Nested
    @DisplayName("Addition Tests")
    class AdditionTests {
        @Test
        @DisplayName("Should add two positive numbers")
        void shouldAddPositiveNumbers() {
            int result = calculator.add(2, 3);
            assertEquals(5, result);
        }

        @Test
        @DisplayName("Should handle negative numbers")
        void shouldHandleNegativeNumbers() {
            assertEquals(-3, calculator.add(-1, -2));
        }

        @Test
        @DisplayName("Should handle zero")
        void shouldHandleZero() {
            assertEquals(5, calculator.add(5, 0));
        }
    }

    @Nested
    @DisplayName("Division Tests")
    class DivisionTests {
        @Test
        void shouldDivideCorrectly() {
            assertEquals(5, calculator.divide(10, 2));
        }

        @Test
        void shouldThrowExceptionOnDivisionByZero() {
            ArithmeticException exception = assertThrows(
                ArithmeticException.class,
                () -> calculator.divide(10, 0)
            );
            assertEquals("Division by zero", exception.getMessage());
        }
    }
}
```

#### Parametrized Tests

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;

class CalculatorParametrizedTest {

    @ParameterizedTest
    @CsvSource({
        "2, 3, 5",
        "-1, -2, -3",
        "0, 0, 0",
        "100, 200, 300"
    })
    void shouldAddNumbers(int a, int b, int expected) {
        Calculator calc = new Calculator();
        assertEquals(expected, calc.add(a, b));
    }

    @ParameterizedTest
    @ValueSource(strings = {"alice@example.com", "bob@test.org", "user@domain.co.uk"})
    void shouldValidateEmails(String email) {
        EmailValidator validator = new EmailValidator();
        assertTrue(validator.isValid(email));
    }

    @ParameterizedTest
    @MethodSource("provideOrderTestData")
    void shouldCalculateOrderTotal(List<OrderItem> items, double expected) {
        Order order = new Order(items);
        assertEquals(expected, order.calculateTotal(), 0.01);
    }

    static Stream<Arguments> provideOrderTestData() {
        return Stream.of(
            Arguments.of(
                List.of(new OrderItem(10.0, 2), new OrderItem(5.0, 1)),
                25.0
            ),
            Arguments.of(
                List.of(new OrderItem(100.0, 1)),
                100.0
            )
        );
    }
}
```

#### Mocking with Mockito

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.InjectMocks;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private PaymentGateway paymentGateway;

    @Mock
    private EmailService emailService;

    @Mock
    private OrderRepository orderRepository;

    @InjectMocks
    private OrderService orderService;

    @Test
    void shouldPlaceOrderSuccessfully() {
        // Arrange
        Order order = new Order("123", 100.0);
        when(paymentGateway.charge(100.0)).thenReturn(
            new PaymentResult(true, "txn_123")
        );
        when(orderRepository.save(any(Order.class))).thenReturn(order);

        // Act
        OrderResult result = orderService.placeOrder(order);

        // Assert
        assertTrue(result.isSuccess());
        assertEquals("123", result.getOrderId());

        // Verify interactions
        verify(paymentGateway).charge(100.0);
        verify(orderRepository).save(order);
        verify(emailService).sendConfirmation(order);
    }

    @Test
    void shouldNotSendEmailIfPaymentFails() {
        // Arrange
        Order order = new Order("123", 100.0);
        when(paymentGateway.charge(100.0)).thenReturn(
            new PaymentResult(false, null)
        );

        // Act & Assert
        assertThrows(PaymentException.class, () -> {
            orderService.placeOrder(order);
        });

        verify(paymentGateway).charge(100.0);
        verify(orderRepository, never()).save(any());
        verify(emailService, never()).sendConfirmation(any());
    }

    @Test
    void shouldRetryOnTransientFailure() {
        // Arrange
        Order order = new Order("123", 100.0);
        when(paymentGateway.charge(100.0))
            .thenThrow(new NetworkException())
            .thenThrow(new NetworkException())
            .thenReturn(new PaymentResult(true, "txn_123"));

        // Act
        OrderResult result = orderService.placeOrder(order);

        // Assert
        assertTrue(result.isSuccess());
        verify(paymentGateway, times(3)).charge(100.0);
    }
}
```

### Integration Testing with Spring Boot

#### Repository Layer Tests

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;

import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldSaveAndRetrieveUser() {
        // Arrange
        User user = new User();
        user.setName("John Doe");
        user.setEmail("john@example.com");

        // Act
        User saved = userRepository.save(user);
        entityManager.flush();

        User found = userRepository.findById(saved.getId()).orElse(null);

        // Assert
        assertThat(found).isNotNull();
        assertThat(found.getName()).isEqualTo("John Doe");
        assertThat(found.getEmail()).isEqualTo("john@example.com");
    }

    @Test
    void shouldFindUserByEmail() {
        // Arrange
        User user = new User();
        user.setName("Jane Doe");
        user.setEmail("jane@example.com");
        entityManager.persist(user);
        entityManager.flush();

        // Act
        User found = userRepository.findByEmail("jane@example.com");

        // Assert
        assertThat(found).isNotNull();
        assertThat(found.getName()).isEqualTo("Jane Doe");
    }

    @Test
    void shouldReturnNullWhenUserNotFound() {
        User found = userRepository.findByEmail("nonexistent@example.com");
        assertThat(found).isNull();
    }

    @Test
    void shouldDeleteUser() {
        // Arrange
        User user = new User();
        user.setEmail("delete@example.com");
        user = entityManager.persist(user);
        Long userId = user.getId();
        entityManager.flush();

        // Act
        userRepository.deleteById(userId);

        // Assert
        User found = userRepository.findById(userId).orElse(null);
        assertThat(found).isNull();
    }
}
```

#### Service Layer Tests

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.transaction.annotation.Transactional;

import static org.mockito.Mockito.*;
import static org.assertj.core.api.Assertions.*;

@SpringBootTest
@Transactional
class UserServiceIntegrationTest {

    @Autowired
    private UserService userService;

    @Autowired
    private UserRepository userRepository;

    @MockBean
    private EmailService emailService;

    @Test
    void shouldCreateUserAndSendWelcomeEmail() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest();
        request.setName("Alice");
        request.setEmail("alice@example.com");
        request.setPassword("password123");

        // Act
        User user = userService.createUser(request);

        // Assert
        assertThat(user.getId()).isNotNull();
        assertThat(user.getName()).isEqualTo("Alice");

        // Verify user in database
        User dbUser = userRepository.findById(user.getId()).orElse(null);
        assertThat(dbUser).isNotNull();

        // Verify email sent
        verify(emailService).sendWelcomeEmail("alice@example.com");
    }

    @Test
    void shouldThrowExceptionForDuplicateEmail() {
        // Arrange
        User existing = new User();
        existing.setEmail("duplicate@example.com");
        userRepository.save(existing);

        CreateUserRequest request = new CreateUserRequest();
        request.setEmail("duplicate@example.com");

        // Act & Assert
        assertThatThrownBy(() -> userService.createUser(request))
            .isInstanceOf(DuplicateEmailException.class)
            .hasMessageContaining("already exists");
    }
}
```

#### REST Controller Tests with MockMvc

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.hamcrest.Matchers.*;

@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void shouldCreateUser() throws Exception {
        // Arrange
        User user = new User(1L, "John", "john@example.com");
        when(userService.createUser(any())).thenReturn(user);

        // Act & Assert
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                        "name": "John",
                        "email": "john@example.com",
                        "password": "password123"
                    }
                    """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.data.id", is(1)))
            .andExpect(jsonPath("$.data.name", is("John")))
            .andExpect(jsonPath("$.data.email", is("john@example.com")));
    }

    @Test
    void shouldReturn400ForInvalidInput() throws Exception {
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                        "name": "",
                        "email": "invalid-email"
                    }
                    """))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.error.code", is("VALIDATION_ERROR")));
    }

    @Test
    void shouldGetUserById() throws Exception {
        // Arrange
        User user = new User(1L, "Jane", "jane@example.com");
        when(userService.getUserById(1L)).thenReturn(user);

        // Act & Assert
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.data.name", is("Jane")))
            .andExpect(jsonPath("$.data.email", is("jane@example.com")));
    }

    @Test
    void shouldReturn404WhenUserNotFound() throws Exception {
        when(userService.getUserById(999L))
            .thenThrow(new UserNotFoundException("User not found"));

        mockMvc.perform(get("/api/users/999"))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.error.code", is("NOT_FOUND")));
    }
}
```

#### Full Integration Tests

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.*;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class UserApiIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
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
    private TestRestTemplate restTemplate;

    @Test
    void shouldCreateAndRetrieveUser() {
        // Create user
        CreateUserRequest request = new CreateUserRequest();
        request.setName("Integration Test User");
        request.setEmail("integration@example.com");
        request.setPassword("password123");

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity<CreateUserRequest> entity = new HttpEntity<>(request, headers);

        ResponseEntity<ApiResponse<User>> createResponse = restTemplate.exchange(
            "/api/users",
            HttpMethod.POST,
            entity,
            new ParameterizedTypeReference<>() {}
        );

        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(createResponse.getBody().getData().getId()).isNotNull();
        Long userId = createResponse.getBody().getData().getId();

        // Retrieve user
        ResponseEntity<ApiResponse<User>> getResponse = restTemplate.exchange(
            "/api/users/" + userId,
            HttpMethod.GET,
            null,
            new ParameterizedTypeReference<>() {}
        );

        assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(getResponse.getBody().getData().getName())
            .isEqualTo("Integration Test User");
    }
}
```

#### Testing with TestContainers

```java
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.utility.DockerImageName;

@SpringBootTest
@Testcontainers
class OrderServiceWithContainersTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @Container
    static GenericContainer<?> redis = new GenericContainer<>(
        DockerImageName.parse("redis:7-alpine")
    ).withExposedPorts(6379);

    @DynamicPropertySource
    static void properties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);

        registry.add("spring.redis.host", redis::getHost);
        registry.add("spring.redis.port", redis::getFirstMappedPort);
    }

    @Autowired
    private OrderService orderService;

    @Test
    void shouldCacheOrderResults() {
        // First call hits database
        Order order1 = orderService.getOrder(123L);

        // Second call uses cache
        Order order2 = orderService.getOrder(123L);

        assertThat(order1).isEqualTo(order2);
        // Verify cache was used (implementation specific)
    }
}
```

---

