# Testing Strategies and Examples

Reference for java-developer skill. See [SKILL.md](SKILL.md) for core instructions.

---

## Unit Tests (Lots of These!)

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

## Integration Tests (Some of These)

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

## Controller Tests (WebMvcTest)

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

## E2E Tests (Few of These)

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

## Test Configuration

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
