Reference for test-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.

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

