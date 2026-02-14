---
name: test-engineer
description: Creates comprehensive automated tests including unit tests, integration tests, and end-to-end tests for JavaScript and Java/Spring Boot applications. Use when the user wants to write tests, improve test coverage, add test suites, or validate application behavior. Supports Jest/Vitest, JUnit 5, Spring Boot Test, Playwright, and other developer testing frameworks. Focuses on developer-written automated tests, not manual QA testing.
---

# Test Engineer

Creates production-quality automated tests across the testing pyramid: unit tests, integration tests, and end-to-end tests. Focuses on developer-written tests that catch bugs early, ensure code quality, and enable confident refactoring.

## When to Use This Skill

Use this skill when the user:

- Asks to "write tests", "add tests", or "test this code"
- Wants to improve test coverage or test quality
- Mentions "unit test", "integration test", "e2e test", "end-to-end test"
- Wants to test APIs, databases, external services
- Asks about testing frameworks (Jest, JUnit, Playwright, TestContainers, etc.)
- Needs to test frontend interactions, user flows, or UI components
- Wants to validate business logic, edge cases, or error handling
- Mentions test automation or CI/CD testing
- Asks "how do I test..." for any application component

## Scope: Developer Tests

**In Scope** (Developer-written automated tests):

- Unit tests (isolated component testing)
- Integration tests (component interaction testing)
- API tests (REST endpoint testing)
- Database integration tests
- End-to-end tests (user journey testing)
- Contract tests (API contract validation)

**Out of Scope** (QA/Manual testing):

- Manual test cases or test plans
- Exploratory testing procedures
- Performance/load testing (unless code examples)
- Security penetration testing
- User acceptance testing (UAT)
- Manual regression test checklists

---

## Testing Pyramid

```
        ┌─────────────┐
        │     E2E     │  Slow, expensive, brittle
        │   (Few)     │  Full user journeys
        └─────────────┘
       ┌───────────────┐
       │  Integration  │  Medium speed/cost
       │    (Some)     │  Component interactions
       └───────────────┘
      ┌─────────────────┐
      │   Unit Tests    │  Fast, cheap, reliable
      │     (Many)      │  Isolated components
      └─────────────────┘
```

**Recommended Distribution:**

- **70%** Unit Tests - Fast, isolated, test business logic
- **20%** Integration Tests - Test component interactions, DB, APIs
- **10%** E2E Tests - Test critical user journeys

**Why This Matters:**

- Unit tests run in milliseconds, E2E tests in seconds/minutes
- Unit tests are stable, E2E tests can be flaky
- Unit tests pinpoint exact failures, E2E tests show broader issues
- More unit tests = faster feedback, lower maintenance

---

## Core Testing Principles

### Test Independence

**Each test must be completely independent.**

```javascript
// ❌ Bad - Tests depend on execution order
let userId;
test("creates user", async () => {
  userId = await createUser({ name: "Alice" });
  expect(userId).toBeDefined();
});

test("finds user", async () => {
  const user = await findUser(userId); // Depends on previous test!
  expect(user.name).toBe("Alice");
});

// ✅ Good - Independent tests
describe("User operations", () => {
  test("creates user", async () => {
    const userId = await createUser({ name: "Alice" });
    expect(userId).toBeDefined();
  });

  test("finds user", async () => {
    const userId = await createUser({ name: "Bob" });
    const user = await findUser(userId);
    expect(user.name).toBe("Bob");
  });
});
```

### Arrange-Act-Assert (AAA)

**Structure every test clearly.**

```javascript
test("calculates order total with discount", () => {
  // Arrange - Set up test data
  const order = new Order([
    { price: 100, quantity: 2 },
    { price: 50, quantity: 1 },
  ]);
  const discount = 0.1; // 10% discount

  // Act - Execute the behavior
  const total = order.calculateTotal(discount);

  // Assert - Verify the result
  expect(total).toBe(225); // (100*2 + 50*1) * 0.9
});
```

### Descriptive Test Names

**Test names should explain what is tested and expected outcome.**

```javascript
// ❌ Bad - Unclear names
test("test1", () => {
  /* ... */
});
test("userTest", () => {
  /* ... */
});

// ✅ Good - Clear, descriptive names
test("should create user with valid email", () => {
  /* ... */
});
test("should throw error when email is invalid", () => {
  /* ... */
});
test("should return 404 when user not found", () => {
  /* ... */
});

// Even better - BDD style
describe("OrderService", () => {
  describe("when placing an order", () => {
    it("should charge the payment method", () => {
      /* ... */
    });
    it("should send confirmation email", () => {
      /* ... */
    });
    it("should update inventory", () => {
      /* ... */
    });
  });
});
```

### Test Behavior, Not Implementation

**Tests should verify what code does, not how it does it.**

```javascript
// ❌ Bad - Testing implementation details
test("should call internal helper method", () => {
  const spy = jest.spyOn(service, "_calculateTax");
  service.processOrder(order);
  expect(spy).toHaveBeenCalled(); // Breaks when refactoring
});

// ✅ Good - Testing behavior
test("should include tax in order total", () => {
  const order = { subtotal: 100, taxRate: 0.1 };
  const result = service.processOrder(order);
  expect(result.total).toBe(110);
});
```

---

## JavaScript Testing

### Frameworks

- **Jest** - Full-featured testing framework (unit + integration)
- **Vitest** - Fast Vite-native alternative to Jest
- **Playwright** - Modern E2E testing framework
- **Cypress** - Alternative E2E framework
- **Supertest** - HTTP assertion library for API testing

### Unit Testing with Jest/Vitest

#### Basic Structure

```javascript
// calculator.test.js
import { describe, test, expect, beforeEach } from "@jest/globals";
import { Calculator } from "./calculator";

describe("Calculator", () => {
  let calculator;

  beforeEach(() => {
    calculator = new Calculator();
  });

  describe("add", () => {
    test("should add two positive numbers", () => {
      expect(calculator.add(2, 3)).toBe(5);
    });

    test("should handle negative numbers", () => {
      expect(calculator.add(-1, -2)).toBe(-3);
    });

    test("should handle zero", () => {
      expect(calculator.add(5, 0)).toBe(5);
    });

    test("should handle decimal numbers", () => {
      expect(calculator.add(1.5, 2.3)).toBeCloseTo(3.8);
    });
  });

  describe("divide", () => {
    test("should divide numbers correctly", () => {
      expect(calculator.divide(10, 2)).toBe(5);
    });

    test("should throw error on division by zero", () => {
      expect(() => calculator.divide(10, 0)).toThrow("Division by zero");
    });
  });
});
```

#### Async Testing

```javascript
// userService.test.js
import { userService } from "./userService";

describe("UserService", () => {
  test("should fetch user by id", async () => {
    const user = await userService.getUser("123");

    expect(user).toMatchObject({
      id: "123",
      name: expect.any(String),
      email: expect.stringContaining("@"),
    });
  });

  test("should throw error for invalid user id", async () => {
    await expect(userService.getUser("invalid")).rejects.toThrow(
      "User not found",
    );
  });

  test("should handle timeout", async () => {
    await expect(userService.getUser("slow-user")).resolves.toBeDefined();
  }, 10000); // 10 second timeout
});
```

#### Mocking

```javascript
// orderService.test.js
import { jest } from "@jest/globals";
import { OrderService } from "./orderService";
import { PaymentGateway } from "./paymentGateway";
import { EmailService } from "./emailService";

// Mock modules
jest.mock("./paymentGateway");
jest.mock("./emailService");

describe("OrderService", () => {
  let orderService;
  let mockPaymentGateway;
  let mockEmailService;

  beforeEach(() => {
    // Create mock instances
    mockPaymentGateway = new PaymentGateway();
    mockEmailService = new EmailService();

    // Setup default mock behavior
    mockPaymentGateway.charge.mockResolvedValue({ success: true });
    mockEmailService.send.mockResolvedValue(true);

    orderService = new OrderService(mockPaymentGateway, mockEmailService);
  });

  test("should charge payment and send email", async () => {
    const order = {
      id: "123",
      total: 100,
      customerEmail: "customer@example.com",
    };

    await orderService.placeOrder(order);

    // Verify payment was charged
    expect(mockPaymentGateway.charge).toHaveBeenCalledWith(100);

    // Verify email was sent
    expect(mockEmailService.send).toHaveBeenCalledWith({
      to: "customer@example.com",
      subject: "Order Confirmation",
      orderId: "123",
    });
  });

  test("should not send email if payment fails", async () => {
    mockPaymentGateway.charge.mockRejectedValue(new Error("Payment failed"));

    const order = { id: "123", total: 100 };

    await expect(orderService.placeOrder(order)).rejects.toThrow(
      "Payment failed",
    );

    expect(mockEmailService.send).not.toHaveBeenCalled();
  });
});
```

#### Spying on Functions

```javascript
// analytics.test.js
import { trackEvent } from "./analytics";

describe("Analytics", () => {
  test("should track button click", () => {
    const consoleSpy = jest.spyOn(console, "log").mockImplementation();

    trackEvent("button_click", { buttonId: "submit" });

    expect(consoleSpy).toHaveBeenCalledWith("Event tracked:", "button_click", {
      buttonId: "submit",
    });

    consoleSpy.mockRestore();
  });
});
```

#### Testing Timers

```javascript
// scheduler.test.js
import { jest } from "@jest/globals";

describe("Scheduler", () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  test("should call callback after delay", () => {
    const callback = jest.fn();

    scheduleTask(callback, 1000);

    expect(callback).not.toHaveBeenCalled();

    jest.advanceTimersByTime(1000);

    expect(callback).toHaveBeenCalledTimes(1);
  });
});
```

### Integration Testing with Jest/Vitest

#### API Integration Tests with Supertest

```javascript
// api.integration.test.js
import request from "supertest";
import { app } from "../app";
import { db } from "../database";

describe("User API", () => {
  beforeAll(async () => {
    await db.connect();
  });

  afterAll(async () => {
    await db.disconnect();
  });

  beforeEach(async () => {
    await db.query("DELETE FROM users");
  });

  describe("POST /api/users", () => {
    test("should create a new user", async () => {
      const response = await request(app)
        .post("/api/users")
        .send({
          name: "John Doe",
          email: "john@example.com",
          password: "password123",
        })
        .expect(201);

      expect(response.body).toMatchObject({
        success: true,
        data: {
          id: expect.any(String),
          name: "John Doe",
          email: "john@example.com",
        },
      });

      // Verify user was created in database
      const user = await db.query("SELECT * FROM users WHERE email = ?", [
        "john@example.com",
      ]);
      expect(user).toHaveLength(1);
    });

    test("should return 400 for invalid email", async () => {
      const response = await request(app)
        .post("/api/users")
        .send({
          name: "John Doe",
          email: "invalid-email",
          password: "password123",
        })
        .expect(400);

      expect(response.body).toMatchObject({
        success: false,
        error: {
          code: "VALIDATION_ERROR",
          message: expect.stringContaining("email"),
        },
      });
    });

    test("should return 409 for duplicate email", async () => {
      await db.query("INSERT INTO users (email, name) VALUES (?, ?)", [
        "john@example.com",
        "Existing User",
      ]);

      await request(app)
        .post("/api/users")
        .send({
          name: "John Doe",
          email: "john@example.com",
          password: "password123",
        })
        .expect(409);
    });
  });

  describe("GET /api/users/:id", () => {
    test("should return user by id", async () => {
      const [insertResult] = await db.query(
        "INSERT INTO users (email, name) VALUES (?, ?)",
        ["jane@example.com", "Jane Doe"],
      );

      const response = await request(app)
        .get(`/api/users/${insertResult.insertId}`)
        .expect(200);

      expect(response.body.data).toMatchObject({
        id: insertResult.insertId.toString(),
        name: "Jane Doe",
        email: "jane@example.com",
      });
    });

    test("should return 404 for non-existent user", async () => {
      await request(app).get("/api/users/99999").expect(404);
    });
  });

  describe("Authentication", () => {
    test("should require authentication for protected routes", async () => {
      await request(app).get("/api/profile").expect(401);
    });

    test("should allow access with valid token", async () => {
      const token = generateTestToken({ userId: "123", role: "user" });

      await request(app)
        .get("/api/profile")
        .set("Authorization", `Bearer ${token}`)
        .expect(200);
    });
  });
});
```

#### Database Integration Tests

```javascript
// userRepository.integration.test.js
import { UserRepository } from "../repositories/userRepository";
import { db } from "../database";

describe("UserRepository", () => {
  let repository;

  beforeAll(async () => {
    await db.connect();
    repository = new UserRepository(db);
  });

  afterAll(async () => {
    await db.disconnect();
  });

  beforeEach(async () => {
    await db.query("TRUNCATE TABLE users");
  });

  describe("create", () => {
    test("should insert user into database", async () => {
      const user = await repository.create({
        name: "Alice",
        email: "alice@example.com",
        password: "hashed_password",
      });

      expect(user).toMatchObject({
        id: expect.any(String),
        name: "Alice",
        email: "alice@example.com",
      });

      const [dbUser] = await db.query("SELECT * FROM users WHERE id = ?", [
        user.id,
      ]);
      expect(dbUser.email).toBe("alice@example.com");
    });
  });

  describe("findByEmail", () => {
    test("should return user when email exists", async () => {
      await db.query("INSERT INTO users (email, name) VALUES (?, ?)", [
        "bob@example.com",
        "Bob",
      ]);

      const user = await repository.findByEmail("bob@example.com");

      expect(user).toMatchObject({
        name: "Bob",
        email: "bob@example.com",
      });
    });

    test("should return null when email does not exist", async () => {
      const user = await repository.findByEmail("nonexistent@example.com");
      expect(user).toBeNull();
    });
  });

  describe("update", () => {
    test("should update user fields", async () => {
      const [result] = await db.query(
        "INSERT INTO users (email, name) VALUES (?, ?)",
        ["charlie@example.com", "Charlie"],
      );
      const userId = result.insertId;

      await repository.update(userId, { name: "Charles" });

      const [updated] = await db.query("SELECT * FROM users WHERE id = ?", [
        userId,
      ]);
      expect(updated.name).toBe("Charles");
    });
  });
});
```

#### External Service Integration Tests

```javascript
// paymentService.integration.test.js
import { PaymentService } from "../services/paymentService";
import nock from "nock";

describe("PaymentService Integration", () => {
  let paymentService;

  beforeEach(() => {
    paymentService = new PaymentService({
      apiKey: "test_key",
      apiUrl: "https://api.payment-provider.com",
    });
  });

  afterEach(() => {
    nock.cleanAll();
  });

  test("should successfully charge payment", async () => {
    // Mock external API
    nock("https://api.payment-provider.com")
      .post("/charges", {
        amount: 1000,
        currency: "usd",
      })
      .reply(200, {
        id: "ch_123",
        status: "succeeded",
        amount: 1000,
      });

    const result = await paymentService.charge({
      amount: 1000,
      currency: "usd",
    });

    expect(result).toMatchObject({
      id: "ch_123",
      status: "succeeded",
    });
  });

  test("should handle payment failure", async () => {
    nock("https://api.payment-provider.com")
      .post("/charges")
      .reply(402, {
        error: {
          code: "card_declined",
          message: "Your card was declined",
        },
      });

    await expect(paymentService.charge({ amount: 1000 })).rejects.toThrow(
      "Your card was declined",
    );
  });

  test("should retry on network error", async () => {
    nock("https://api.payment-provider.com")
      .post("/charges")
      .times(2)
      .replyWithError("Network error")
      .post("/charges")
      .reply(200, { id: "ch_123", status: "succeeded" });

    const result = await paymentService.charge({ amount: 1000 });

    expect(result.status).toBe("succeeded");
  });
});
```

### E2E Testing with Playwright

#### Setup

```javascript
// playwright.config.js
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: "html",
  use: {
    baseURL: "http://localhost:3000",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
  },
  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
    {
      name: "firefox",
      use: { ...devices["Desktop Firefox"] },
    },
    {
      name: "webkit",
      use: { ...devices["Desktop Safari"] },
    },
  ],
  webServer: {
    command: "npm run start",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
  },
});
```

#### Basic E2E Tests

```javascript
// e2e/login.spec.js
import { test, expect } from "@playwright/test";

test.describe("User Login", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/login");
  });

  test("should login with valid credentials", async ({ page }) => {
    // Fill in login form
    await page.fill('input[name="email"]', "user@example.com");
    await page.fill('input[name="password"]', "password123");

    // Click login button
    await page.click('button[type="submit"]');

    // Wait for navigation
    await page.waitForURL("/dashboard");

    // Verify user is logged in
    await expect(page.locator("h1")).toContainText("Dashboard");
    await expect(page.locator('[data-testid="user-menu"]')).toContainText(
      "user@example.com",
    );
  });

  test("should show error with invalid credentials", async ({ page }) => {
    await page.fill('input[name="email"]', "user@example.com");
    await page.fill('input[name="password"]', "wrongpassword");
    await page.click('button[type="submit"]');

    // Error message should appear
    await expect(page.locator(".error-message")).toContainText(
      "Invalid email or password",
    );

    // Should stay on login page
    expect(page.url()).toContain("/login");
  });

  test("should validate required fields", async ({ page }) => {
    await page.click('button[type="submit"]');

    await expect(page.locator('input[name="email"]:invalid')).toBeVisible();
    await expect(page.locator('input[name="password"]:invalid')).toBeVisible();
  });
});
```

#### Complete User Journey

```javascript
// e2e/checkout.spec.js
import { test, expect } from "@playwright/test";

test.describe("E-commerce Checkout Flow", () => {
  test("should complete purchase from product to confirmation", async ({
    page,
  }) => {
    // Navigate to products page
    await page.goto("/products");

    // Select a product
    await page.click("text=Premium Widget");
    await expect(page).toHaveURL(/\/products\/\d+/);

    // Add to cart
    await page.click('button:has-text("Add to Cart")');
    await expect(page.locator(".cart-badge")).toContainText("1");

    // Go to cart
    await page.click('[data-testid="cart-icon"]');
    await expect(page).toHaveURL("/cart");

    // Verify item in cart
    await expect(page.locator(".cart-item")).toContainText("Premium Widget");

    // Proceed to checkout
    await page.click('button:has-text("Checkout")');
    await expect(page).toHaveURL("/checkout");

    // Fill shipping information
    await page.fill('input[name="fullName"]', "John Doe");
    await page.fill('input[name="address"]', "123 Main St");
    await page.fill('input[name="city"]', "New York");
    await page.fill('input[name="zipCode"]', "10001");

    // Fill payment information
    await page.fill('input[name="cardNumber"]', "4242424242424242");
    await page.fill('input[name="expiry"]', "12/25");
    await page.fill('input[name="cvv"]', "123");

    // Submit order
    await page.click('button:has-text("Place Order")');

    // Wait for confirmation page
    await page.waitForURL("/order-confirmation");

    // Verify order confirmation
    await expect(page.locator("h1")).toContainText("Order Confirmed");
    const orderNumber = await page
      .locator('[data-testid="order-number"]')
      .textContent();
    expect(orderNumber).toMatch(/^ORD-\d+$/);
  });
});
```

#### Testing with Authentication

```javascript
// e2e/auth.setup.js
import { test as setup, expect } from "@playwright/test";

const authFile = "playwright/.auth/user.json";

setup("authenticate", async ({ page }) => {
  await page.goto("/login");
  await page.fill('input[name="email"]', "test@example.com");
  await page.fill('input[name="password"]', "password123");
  await page.click('button[type="submit"]');

  await page.waitForURL("/dashboard");

  // Save authentication state
  await page.context().storageState({ path: authFile });
});

// Use in tests
// e2e/profile.spec.js
import { test, expect } from "@playwright/test";

test.use({ storageState: "playwright/.auth/user.json" });

test("should update user profile", async ({ page }) => {
  await page.goto("/profile");

  await page.fill('input[name="name"]', "Updated Name");
  await page.click('button:has-text("Save")');

  await expect(page.locator(".success-message")).toContainText(
    "Profile updated",
  );
});
```

#### API Testing with Playwright

```javascript
// e2e/api.spec.js
import { test, expect } from "@playwright/test";

test.describe("API Tests", () => {
  let apiContext;

  test.beforeAll(async ({ playwright }) => {
    apiContext = await playwright.request.newContext({
      baseURL: "http://localhost:3000/api",
      extraHTTPHeaders: {
        Authorization: `Bearer ${process.env.TEST_API_TOKEN}`,
      },
    });
  });

  test.afterAll(async () => {
    await apiContext.dispose();
  });

  test("should create and retrieve user", async () => {
    // Create user
    const createResponse = await apiContext.post("/users", {
      data: {
        name: "API Test User",
        email: "apitest@example.com",
      },
    });
    expect(createResponse.ok()).toBeTruthy();

    const user = await createResponse.json();
    expect(user.data.id).toBeDefined();

    // Retrieve user
    const getResponse = await apiContext.get(`/users/${user.data.id}`);
    expect(getResponse.ok()).toBeTruthy();

    const retrievedUser = await getResponse.json();
    expect(retrievedUser.data).toMatchObject({
      name: "API Test User",
      email: "apitest@example.com",
    });
  });
});
```

#### Visual Regression Testing

```javascript
// e2e/visual.spec.js
import { test, expect } from "@playwright/test";

test("homepage should match snapshot", async ({ page }) => {
  await page.goto("/");

  // Wait for page to be fully loaded
  await page.waitForLoadState("networkidle");

  // Take screenshot and compare
  await expect(page).toHaveScreenshot("homepage.png");
});

test("should match component snapshots", async ({ page }) => {
  await page.goto("/components");

  const button = page.locator('[data-testid="primary-button"]');
  await expect(button).toHaveScreenshot("primary-button.png");
});
```

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

## Testing Best Practices

### Test Data Management

```javascript
// Use test data builders/factories
class UserBuilder {
  constructor() {
    this.data = {
      name: "Test User",
      email: `user${Date.now()}@example.com`,
      role: "user",
      active: true,
    };
  }

  withName(name) {
    this.data.name = name;
    return this;
  }

  withEmail(email) {
    this.data.email = email;
    return this;
  }

  asAdmin() {
    this.data.role = "admin";
    return this;
  }

  inactive() {
    this.data.active = false;
    return this;
  }

  build() {
    return this.data;
  }
}

// Usage in tests
test("should create admin user", () => {
  const admin = new UserBuilder().withName("Admin User").asAdmin().build();

  expect(admin.role).toBe("admin");
});
```

```java
// Java version
public class UserTestBuilder {
    private String name = "Test User";
    private String email = "test@example.com";
    private String role = "USER";
    private boolean active = true;

    public UserTestBuilder withName(String name) {
        this.name = name;
        return this;
    }

    public UserTestBuilder withEmail(String email) {
        this.email = email;
        return this;
    }

    public UserTestBuilder asAdmin() {
        this.role = "ADMIN";
        return this;
    }

    public User build() {
        User user = new User();
        user.setName(name);
        user.setEmail(email);
        user.setRole(role);
        user.setActive(active);
        return user;
    }
}
```

### Test Isolation

```javascript
// ✅ Good - Each test creates its own data
describe("User API", () => {
  test("should create user", async () => {
    const user = await createTestUser();
    expect(user.id).toBeDefined();
  });

  test("should update user", async () => {
    const user = await createTestUser();
    const updated = await updateUser(user.id, { name: "New Name" });
    expect(updated.name).toBe("New Name");
  });
});

// ✅ Good - Clean up after each test
afterEach(async () => {
  await cleanupDatabase();
});
```

### Meaningful Assertions

```javascript
// ❌ Bad - Generic assertion
expect(user).toBeTruthy();

// ✅ Good - Specific assertion
expect(user).toMatchObject({
  id: expect.any(String),
  name: "John Doe",
  email: "john@example.com",
  createdAt: expect.any(Date),
});

// ❌ Bad - Unclear failure message
expect(result).toBe(true);

// ✅ Good - Clear failure message
expect(result, "Payment should succeed for valid card").toBe(true);
```

### Test Organization

```javascript
// ✅ Good - Organized with describe blocks
describe("OrderService", () => {
  describe("placeOrder", () => {
    describe("when payment succeeds", () => {
      it("should save order to database", () => {});
      it("should send confirmation email", () => {});
      it("should update inventory", () => {});
    });

    describe("when payment fails", () => {
      it("should not save order", () => {});
      it("should not send email", () => {});
      it("should throw PaymentError", () => {});
    });
  });
});
```

### Avoiding Flaky Tests

```javascript
// ❌ Bad - Depends on timing
test("should update status", async () => {
  startBackgroundJob();
  await sleep(1000); // Flaky!
  expect(status).toBe("completed");
});

// ✅ Good - Wait for condition
test("should update status", async () => {
  startBackgroundJob();
  await waitFor(
    () => {
      expect(status).toBe("completed");
    },
    { timeout: 5000 },
  );
});

// ❌ Bad - Depends on system state
test("should create file", () => {
  createFile("/tmp/test.txt");
  expect(fs.existsSync("/tmp/test.txt")).toBe(true);
  // What if file already exists?
});

// ✅ Good - Use unique names
test("should create file", () => {
  const filename = `/tmp/test-${Date.now()}.txt`;
  createFile(filename);
  expect(fs.existsSync(filename)).toBe(true);
  fs.unlinkSync(filename); // Cleanup
});
```

---

## Code Coverage

### Coverage Metrics

```bash
# JavaScript - Generate coverage report
npm test -- --coverage

# View coverage report
open coverage/lcov-report/index.html
```

```bash
# Java - Maven with JaCoCo
mvn clean test jacoco:report

# View report
open target/site/jacoco/index.html
```

### Coverage Goals

```javascript
// jest.config.js
export default {
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    "./src/services/": {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90,
    },
  },
};
```

```xml
<!-- Java - pom.xml -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <configuration>
        <rules>
            <rule>
                <element>PACKAGE</element>
                <limits>
                    <limit>
                        <counter>LINE</counter>
                        <value>COVEREDRATIO</value>
                        <minimum>0.80</minimum>
                    </limit>
                </limits>
            </rule>
        </rules>
    </configuration>
</plugin>
```

**Important**: High coverage doesn't guarantee good tests. Focus on meaningful tests that catch real bugs.

---

## CI/CD Integration

### GitHub Actions - JavaScript

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm test -- --coverage

      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb

      - name: Install Playwright
        run: npx playwright install --with-deps

      - name: Run E2E tests
        run: npm run test:e2e

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

### GitHub Actions - Java

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: "17"
          distribution: "temurin"
          cache: maven

      - name: Run tests with Maven
        run: mvn clean verify

      - name: Generate coverage report
        run: mvn jacoco:report

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./target/site/jacoco/jacoco.xml
```

---

## Quick Reference Checklist

**Before Writing Tests:**

- [ ] Understand what behavior to test
- [ ] Identify edge cases and error conditions
- [ ] Plan test data needs
- [ ] Choose appropriate test type (unit/integration/e2e)

**Writing Tests:**

- [ ] Use descriptive test names
- [ ] Follow AAA pattern (Arrange-Act-Assert)
- [ ] One logical assertion per test
- [ ] Test behavior, not implementation
- [ ] Make tests independent
- [ ] Clean up resources

**For Unit Tests:**

- [ ] Mock external dependencies
- [ ] Test happy path
- [ ] Test edge cases
- [ ] Test error handling
- [ ] Fast execution (< 100ms per test)

**For Integration Tests:**

- [ ] Use real database/services when possible
- [ ] Clean database before each test
- [ ] Test component interactions
- [ ] Verify data persistence
- [ ] Test transactions

**For E2E Tests:**

- [ ] Test critical user journeys
- [ ] Use stable selectors (data-testid)
- [ ] Wait for elements properly
- [ ] Handle authentication
- [ ] Test on multiple browsers (if needed)
- [ ] Keep tests focused and fast

**Code Quality:**

- [ ] Tests are readable and maintainable
- [ ] No hardcoded values (use constants)
- [ ] Good test data builders
- [ ] Meaningful coverage (not just high %)
- [ ] Tests run in CI/CD

---

## Anti-Patterns to Avoid

### Testing Implementation Details

```javascript
// ❌ Bad
test("should call handleSubmit", () => {
  const spy = jest.spyOn(component, "handleSubmit");
  component.render();
  expect(spy).toHaveBeenCalled();
});

// ✅ Good
test("should submit form data", () => {
  render(<Form />);
  fireEvent.click(screen.getByRole("button", { name: "Submit" }));
  expect(mockSubmit).toHaveBeenCalledWith({ name: "John" });
});
```

### Large, Unfocused Tests

```javascript
// ❌ Bad
test("user flow", async () => {
  const user = await createUser();
  expect(user.id).toBeDefined();
  const updated = await updateUser(user.id, { name: "New" });
  expect(updated.name).toBe("New");
  const deleted = await deleteUser(user.id);
  expect(deleted).toBe(true);
  // Too many concerns in one test!
});

// ✅ Good - Separate tests
test("should create user", async () => {});
test("should update user", async () => {});
test("should delete user", async () => {});
```

### Not Cleaning Up

```javascript
// ❌ Bad
test("should create file", () => {
  fs.writeFileSync("/tmp/test.txt", "data");
  expect(fs.existsSync("/tmp/test.txt")).toBe(true);
  // File left behind!
});

// ✅ Good
test("should create file", () => {
  const file = "/tmp/test.txt";
  try {
    fs.writeFileSync(file, "data");
    expect(fs.existsSync(file)).toBe(true);
  } finally {
    if (fs.existsSync(file)) fs.unlinkSync(file);
  }
});
```

### Ignoring Test Failures

```javascript
// ❌ Bad
test.skip("broken test", () => {});

// ✅ Good - Fix or document
test("feature X", () => {
  // TODO: Fix after API changes (see ticket #123)
  expect.todo("implement when ready");
});
```

---

## Summary

**Testing Pyramid:**

- Write mostly unit tests (fast, focused)
- Some integration tests (component interactions)
- Few E2E tests (critical user journeys)

**Key Principles:**

- Tests should be independent
- Test behavior, not implementation
- Make tests readable and maintainable
- Fast feedback is crucial
- Meaningful coverage over high percentages

**Test Types:**

- **Unit**: Isolated components, fast, many
- **Integration**: Component interactions, medium, some
- **E2E**: User journeys, slow, few

**Remember**: Good tests give you confidence to refactor and deploy. They should catch bugs, not prevent progress.
