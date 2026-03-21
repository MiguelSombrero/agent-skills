Reference for software-architect skill. See [SKILL.md](../SKILL.md) for core instructions.
---

## Decision Framework

### When Planning Changes

Ask these questions:

1. **What problem am I solving?**
   - Define the problem clearly
   - Identify success criteria
   - Consider if this is the right problem to solve

2. **What are the constraints?**
   - Time/budget limitations
   - Existing system limitations
   - Team expertise
   - Performance requirements
   - Scalability needs

3. **What are the options?**
   - List multiple approaches
   - Consider pros/cons of each
   - Evaluate against constraints

4. **What are the trade-offs?**
   - Complexity vs. simplicity
   - Performance vs. maintainability
   - Flexibility vs. simplicity
   - Build vs. buy
   - Now vs. later

5. **What's the migration path?**
   - Can we deploy incrementally?
   - Is there a rollback plan?
   - How do we handle existing data?
   - What's the timeline?

6. **How do we verify success?**
   - What metrics will we track?
   - What tests will we write?
   - How will we monitor in production?

### Trade-off Examples

```typescript
// Trade-off: Simplicity vs. Flexibility
// Simple: Hard-coded payment processor
class OrderService {
  async processPayment(order: Order) {
    return stripeAPI.charge(order.total);
  }
}

// Flexible: Strategy pattern for multiple processors
class OrderService {
  constructor(private paymentProcessor: PaymentProcessor) {}

  async processPayment(order: Order) {
    return this.paymentProcessor.charge(order.total);
  }
}
// Choose based on: Will we need multiple payment processors?

// Trade-off: Performance vs. Maintainability
// Performant but harder to maintain
const result = data
  .filter((x) => x.status === "active")
  .reduce((acc, x) => ({ ...acc, [x.id]: x }), {});

// Slower but clearer
const activeItems = data.filter((x) => x.status === "active");
const result = {};
for (const item of activeItems) {
  result[item.id] = item;
}
// Choose based on: Is this a hot path? How often is it called?
```

---

## Anti-Patterns to Avoid

### God Object

**Problem**: One class does everything

```typescript
// ❌ Bad
class Application {
  handleRequest() {}
  connectToDatabase() {}
  sendEmail() {}
  processPayment() {}
  generateReport() {}
  // ... 50 more methods
}

// ✅ Good - Separate concerns
class RequestHandler {}
class DatabaseConnection {}
class EmailService {}
class PaymentProcessor {}
class ReportGenerator {}
```

### Spaghetti Code

**Problem**: Tangled dependencies, unclear flow

```typescript
// ❌ Bad
function processOrder(order) {
  if (validateOrder(order)) {
    if (checkInventory(order)) {
      const payment = processPayment(order);
      if (payment.success) {
        updateInventory(order);
        sendEmail(order.customer);
        if (order.isPriority) {
          priorityShipping(order);
        } else {
          standardShipping(order);
        }
      }
    }
  }
}

// ✅ Good - Clear steps, early returns
function processOrder(order: Order): OrderResult {
  validateOrderOrThrow(order);
  checkInventoryOrThrow(order);

  const payment = processPayment(order);
  if (!payment.success) {
    throw new PaymentError();
  }

  updateInventory(order);
  sendConfirmationEmail(order.customer);
  scheduleShipping(order);

  return { success: true, orderId: order.id };
}
```

### Premature Optimization

**Problem**: Optimizing before identifying bottlenecks

```typescript
// ❌ Bad - Complex optimization without measuring
class CacheManager {
  private l1Cache = new Map();
  private l2Cache = new Map();
  private lruQueue = new Queue();
  // Complex multi-level caching before knowing if it's needed
}

// ✅ Good - Start simple, measure, then optimize
class CacheManager {
  private cache = new Map();

  get(key) {
    return this.cache.get(key);
  }
  set(key, value) {
    this.cache.set(key, value);
  }
}
// Measure performance, identify bottlenecks, then optimize
```

### Tight Coupling

**Problem**: Components depend directly on concrete implementations

```typescript
// ❌ Bad
class UserService {
  private db = new PostgresDatabase(); // Tight coupling!

  async getUser(id: string) {
    return this.db.query("SELECT * FROM users WHERE id = ?", [id]);
  }
}

// ✅ Good
class UserService {
  constructor(private repository: UserRepository) {}

  async getUser(id: string) {
    return this.repository.findById(id);
  }
}
```

### Magic Numbers/Strings

**Problem**: Unexplained literal values

```typescript
// ❌ Bad
if (user.status === 3) {
  // What does 3 mean?
}

setTimeout(updateData, 86400000); // What is this number?

// ✅ Good
const UserStatus = {
  ACTIVE: 1,
  INACTIVE: 2,
  SUSPENDED: 3,
} as const;

if (user.status === UserStatus.SUSPENDED) {
  // Clear meaning
}

const ONE_DAY_MS = 24 * 60 * 60 * 1000;
setTimeout(updateData, ONE_DAY_MS);
```

---

