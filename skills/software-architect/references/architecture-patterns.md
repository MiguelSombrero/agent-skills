Reference for software-architect skill. See [SKILL.md](../SKILL.md) for core instructions.
---

## Architecture Patterns

### Layered Architecture

```
┌─────────────────────────────────┐
│     Presentation Layer          │  (Controllers, Views, API)
├─────────────────────────────────┤
│     Application Layer           │  (Use Cases, Services)
├─────────────────────────────────┤
│     Domain Layer                │  (Business Logic, Entities)
├─────────────────────────────────┤
│     Infrastructure Layer        │  (DB, External APIs, File I/O)
└─────────────────────────────────┘
```

**Example Structure:**

```typescript
// Domain Layer - Pure business logic
class Order {
  constructor(
    public id: string,
    public items: OrderItem[],
    public status: OrderStatus,
  ) {}

  calculateTotal(): number {
    return this.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0,
    );
  }

  canBeCancelled(): boolean {
    return this.status === OrderStatus.PENDING;
  }
}

// Application Layer - Orchestration
class OrderService {
  constructor(
    private orderRepo: OrderRepository,
    private paymentService: PaymentService,
    private emailService: EmailService,
  ) {}

  async placeOrder(orderData: CreateOrderDTO): Promise<Order> {
    const order = new Order(/* ... */);

    await this.paymentService.charge(order.calculateTotal());
    await this.orderRepo.save(order);
    await this.emailService.sendConfirmation(order);

    return order;
  }
}

// Presentation Layer - HTTP interface
class OrderController {
  constructor(private orderService: OrderService) {}

  async createOrder(req: Request, res: Response) {
    try {
      const order = await this.orderService.placeOrder(req.body);
      res.json({ success: true, order });
    } catch (error) {
      res.status(500).json({ success: false, error: error.message });
    }
  }
}

// Infrastructure Layer - Technical concerns
class PostgresOrderRepository implements OrderRepository {
  async save(order: Order): Promise<void> {
    // PostgreSQL-specific implementation
  }
}
```

### Hexagonal Architecture (Ports and Adapters)

```
        ┌───────────────────────────┐
        │                           │
        │    Application Core       │
        │   (Domain + Use Cases)    │
        │                           │
        └─────────┬─────────┬───────┘
                  │         │
         ┌────────┴──┐   ┌──┴────────┐
         │  Port     │   │  Port     │
         │(Interface)│   │(Interface)│
         └────┬──────┘   └──────┬────┘
              │                 │
         ┌────┴──────┐     ┌────┴──────┐
         │  Adapter  │     │  Adapter  │
         │  (HTTP)   │     │  (DB)     │
         └───────────┘     └───────────┘
```

**Example:**

```typescript
// Port (interface in core)
interface UserRepository {
  findById(id: string): Promise<User>;
  save(user: User): Promise<void>;
}

// Core domain logic depends on port, not adapter
class UserService {
  constructor(private userRepo: UserRepository) {}

  async promoteToAdmin(userId: string): Promise<void> {
    const user = await this.userRepo.findById(userId);
    user.role = "admin";
    await this.userRepo.save(user);
  }
}

// Adapter implements port
class MongoUserRepository implements UserRepository {
  async findById(id: string): Promise<User> {
    const doc = await this.collection.findOne({ _id: id });
    return this.mapToUser(doc);
  }

  async save(user: User): Promise<void> {
    await this.collection.updateOne(
      { _id: user.id },
      { $set: { role: user.role } },
    );
  }
}

// Can swap adapters without changing core
class PostgresUserRepository implements UserRepository {
  // Different implementation, same interface
}
```

### Clean Architecture

```
┌─────────────────────────────────────┐
│         Frameworks & Drivers        │  External interfaces
├─────────────────────────────────────┤
│        Interface Adapters           │  Controllers, Presenters
├─────────────────────────────────────┤
│          Use Cases                  │  Application business rules
├─────────────────────────────────────┤
│          Entities                   │  Enterprise business rules
└─────────────────────────────────────┘
         Dependencies point inward →
```

**Key Rules:**

1. Dependencies point inward (toward business logic)
2. Inner layers don't know about outer layers
3. Entities contain enterprise-wide business rules
4. Use cases contain application-specific business rules

### Event-Driven Architecture

```typescript
// Event
interface OrderPlacedEvent {
  type: "ORDER_PLACED";
  orderId: string;
  customerId: string;
  timestamp: Date;
}

// Event bus
class EventBus {
  private handlers: Map<string, Function[]> = new Map();

  subscribe(eventType: string, handler: Function): void {
    if (!this.handlers.has(eventType)) {
      this.handlers.set(eventType, []);
    }
    this.handlers.get(eventType)!.push(handler);
  }

  publish(event: any): void {
    const handlers = this.handlers.get(event.type) || [];
    handlers.forEach((handler) => handler(event));
  }
}

// Handlers
class EmailNotificationHandler {
  handle(event: OrderPlacedEvent): void {
    console.log(`Sending email for order ${event.orderId}`);
  }
}

class InventoryHandler {
  handle(event: OrderPlacedEvent): void {
    console.log(`Updating inventory for order ${event.orderId}`);
  }
}

// Usage
const eventBus = new EventBus();
eventBus.subscribe("ORDER_PLACED", new EmailNotificationHandler().handle);
eventBus.subscribe("ORDER_PLACED", new InventoryHandler().handle);

eventBus.publish({
  type: "ORDER_PLACED",
  orderId: "123",
  customerId: "456",
  timestamp: new Date(),
});
```

### Microservices Patterns

#### API Gateway

Single entry point for all clients, handles routing, auth, rate limiting.

#### Service Registry

Services register themselves, others discover them dynamically.

#### Circuit Breaker

Prevent cascading failures when a service is down.

```typescript
class CircuitBreaker {
  private failures = 0;
  private lastFailTime: Date | null = null;
  private state: "CLOSED" | "OPEN" | "HALF_OPEN" = "CLOSED";

  constructor(
    private threshold: number = 5,
    private timeout: number = 60000, // 1 minute
  ) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === "OPEN") {
      if (Date.now() - this.lastFailTime!.getTime() > this.timeout) {
        this.state = "HALF_OPEN";
      } else {
        throw new Error("Circuit breaker is OPEN");
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess(): void {
    this.failures = 0;
    this.state = "CLOSED";
  }

  private onFailure(): void {
    this.failures++;
    this.lastFailTime = new Date();

    if (this.failures >= this.threshold) {
      this.state = "OPEN";
    }
  }
}

// Usage
const breaker = new CircuitBreaker();
try {
  const data = await breaker.execute(() => externalApiCall());
} catch (error) {
  // Handle error or use fallback
}
```

---

