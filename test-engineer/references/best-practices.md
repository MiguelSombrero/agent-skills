Reference for test-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.

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
