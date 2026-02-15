---
name: software-architect
description: Plans and executes complex architectural changes with deep expertise in design patterns, best practices, security, scalability, and cross-cutting concerns. Use when the user needs to refactor systems, design new features, improve code structure, implement design patterns, address technical debt, or make architectural decisions.
---

# Software Architect

Provides senior-level architectural guidance for planning and executing complex system changes. Focuses on maintainability, scalability, security, and long-term system health while balancing pragmatic trade-offs.

## When to Use This Skill

Use this skill when the user:

- Wants to **refactor** or **restructure** existing code
- Needs to **design a new feature** or **system component**
- Asks about **design patterns**, **architecture patterns**, or **best practices**
- Wants to **improve code organization** or **reduce technical debt**
- Needs guidance on **scaling**, **performance**, or **security**
- Is planning **complex migrations** or **system changes**
- Asks "how should I structure..." or "what's the best way to..."
- Mentions architectural concerns: modularity, coupling, cohesion, separation of concerns
- Needs help with **API design**, **database schema**, or **system integration**
- Wants to evaluate **trade-offs** between different approaches

## Core Architectural Principles

### SOLID Principles

#### Single Responsibility Principle (SRP)

**"A class should have only one reason to change"**

```typescript
// ❌ Bad - Multiple responsibilities
class User {
  saveToDatabase() {
    /* DB logic */
  }
  sendEmail() {
    /* Email logic */
  }
  validateInput() {
    /* Validation logic */
  }
  generateReport() {
    /* Reporting logic */
  }
}

// ✅ Good - Single responsibility per class
class User {
  constructor(
    private validator: UserValidator,
    private repository: UserRepository,
    private emailService: EmailService,
    private reportGenerator: ReportGenerator,
  ) {}
}

class UserValidator {
  validate(user: User): ValidationResult {
    /* ... */
  }
}

class UserRepository {
  save(user: User): Promise<void> {
    /* ... */
  }
}
```

#### Open/Closed Principle (OCP)

**"Open for extension, closed for modification"**

```typescript
// ❌ Bad - Must modify existing code for new types
class PaymentProcessor {
  process(type: string, amount: number) {
    if (type === "credit") {
      /* credit logic */
    } else if (type === "debit") {
      /* debit logic */
    } else if (type === "paypal") {
      /* paypal logic */
    }
  }
}

// ✅ Good - Extend through interfaces
interface PaymentMethod {
  process(amount: number): Promise<PaymentResult>;
}

class CreditCardPayment implements PaymentMethod {
  process(amount: number): Promise<PaymentResult> {
    /* ... */
  }
}

class PayPalPayment implements PaymentMethod {
  process(amount: number): Promise<PaymentResult> {
    /* ... */
  }
}

class PaymentProcessor {
  constructor(private method: PaymentMethod) {}

  async process(amount: number) {
    return this.method.process(amount);
  }
}
```

#### Liskov Substitution Principle (LSP)

**"Subtypes must be substitutable for their base types"**

```typescript
// ❌ Bad - Square violates LSP (changing width affects height)
class Rectangle {
  constructor(
    protected width: number,
    protected height: number,
  ) {}

  setWidth(width: number) {
    this.width = width;
  }
  setHeight(height: number) {
    this.height = height;
  }
  getArea() {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  setWidth(width: number) {
    this.width = width;
    this.height = width; // Violates LSP!
  }
}

// ✅ Good - Use composition over inheritance
interface Shape {
  getArea(): number;
}

class Rectangle implements Shape {
  constructor(
    private width: number,
    private height: number,
  ) {}
  getArea() {
    return this.width * this.height;
  }
}

class Square implements Shape {
  constructor(private side: number) {}
  getArea() {
    return this.side * this.side;
  }
}
```

#### Interface Segregation Principle (ISP)

**"Clients shouldn't depend on interfaces they don't use"**

```typescript
// ❌ Bad - Fat interface forces unnecessary implementations
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}

class Robot implements Worker {
  work() {
    /* ... */
  }
  eat() {
    throw new Error("Robots don't eat");
  } // Forced to implement
  sleep() {
    throw new Error("Robots don't sleep");
  }
}

// ✅ Good - Segregated interfaces
interface Workable {
  work(): void;
}

interface Eatable {
  eat(): void;
}

interface Sleepable {
  sleep(): void;
}

class Human implements Workable, Eatable, Sleepable {
  work() {
    /* ... */
  }
  eat() {
    /* ... */
  }
  sleep() {
    /* ... */
  }
}

class Robot implements Workable {
  work() {
    /* ... */
  }
}
```

#### Dependency Inversion Principle (DIP)

**"Depend on abstractions, not concretions"**

```typescript
// ❌ Bad - High-level module depends on low-level module
class EmailService {
  send(to: string, message: string) {
    /* SMTP implementation */
  }
}

class UserRegistration {
  private emailService = new EmailService(); // Tight coupling!

  register(user: User) {
    // ...
    this.emailService.send(user.email, "Welcome");
  }
}

// ✅ Good - Both depend on abstraction
interface MessageService {
  send(to: string, message: string): Promise<void>;
}

class EmailService implements MessageService {
  send(to: string, message: string) {
    /* SMTP */
  }
}

class SMSService implements MessageService {
  send(to: string, message: string) {
    /* SMS API */
  }
}

class UserRegistration {
  constructor(private messageService: MessageService) {}

  async register(user: User) {
    // ...
    await this.messageService.send(user.contact, "Welcome");
  }
}
```

### Other Essential Principles

#### DRY (Don't Repeat Yourself)

Eliminate duplication, but avoid premature abstraction.

```typescript
// ❌ Bad - Repeated validation logic
function createUser(data: UserData) {
  if (!data.email) throw new Error("Email required");
  if (!data.email.includes("@")) throw new Error("Invalid email");
  // ...
}

function updateUser(id: string, data: UserData) {
  if (!data.email) throw new Error("Email required");
  if (!data.email.includes("@")) throw new Error("Invalid email");
  // ...
}

// ✅ Good - Extract common logic
class UserValidator {
  validateEmail(email: string): ValidationResult {
    if (!email) return { valid: false, error: "Email required" };
    if (!email.includes("@")) return { valid: false, error: "Invalid email" };
    return { valid: true };
  }
}
```

#### KISS (Keep It Simple, Stupid)

Prefer simple solutions over clever ones.

```typescript
// ❌ Bad - Over-engineered
const sum = numbers.reduce((acc, n) => acc + n, 0);
const avg = sum / numbers.length;
const variance =
  numbers.reduce((acc, n) => acc + Math.pow(n - avg, 2), 0) / numbers.length;
const stdDev = Math.sqrt(variance);

// ✅ Good - Clear and simple (if you only need average)
const sum = numbers.reduce((acc, n) => acc + n, 0);
const average = sum / numbers.length;
```

#### YAGNI (You Aren't Gonna Need It)

Don't build features until they're actually needed.

```typescript
// ❌ Bad - Building for imaginary future requirements
class User {
  // Maybe we'll need these someday?
  alternateEmails: string[] = [];
  socialMediaLinks: Record<string, string> = {};
  preferences: Record<string, any> = {};
  metadata: Record<string, any> = {};
}

// ✅ Good - Only what's needed now
class User {
  constructor(
    public id: string,
    public email: string,
    public name: string,
  ) {}
}
// Add fields when you actually need them
```

#### Separation of Concerns

Each module should address a separate concern.

```typescript
// ✅ Good - Clear separation
// Domain Layer
class Order {
  constructor(
    public items: OrderItem[],
    public customer: Customer,
  ) {}
  calculateTotal(): number {
    /* business logic */
  }
}

// Application Layer
class OrderService {
  constructor(
    private repository: OrderRepository,
    private emailService: EmailService,
    private paymentGateway: PaymentGateway,
  ) {}

  async placeOrder(order: Order): Promise<void> {
    await this.paymentGateway.charge(order);
    await this.repository.save(order);
    await this.emailService.sendConfirmation(order);
  }
}

// Infrastructure Layer
class OrderRepository {
  async save(order: Order): Promise<void> {
    /* DB logic */
  }
}
```

---


---

## Additional Resources

For detailed patterns and examples, see these reference files:

- **Design patterns**: [design-patterns.md](references/design-patterns.md) — Creational, structural, behavioral patterns
- **Architecture patterns**: [architecture-patterns.md](references/architecture-patterns.md) — Layered, CQRS, event-driven
- **Cross-cutting concerns**: [cross-cutting.md](references/cross-cutting.md) — Logging, security, caching
- **Database and API design**: [database-and-api.md](references/database-and-api.md)
- **Migration and organization**: [migration-and-organization.md](references/migration-and-organization.md)
- **Decision framework and anti-patterns**: [decision-and-antipatterns.md](references/decision-and-antipatterns.md)
- **Workflow and example**: [workflow-and-example.md](references/workflow-and-example.md) — Refactoring workflow
- **Principles and checklist**: [principles-and-checklist.md](references/principles-and-checklist.md)

---
