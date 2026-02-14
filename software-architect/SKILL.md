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

## Design Patterns

### Creational Patterns

#### Factory Pattern

Create objects without specifying exact class.

```typescript
interface Vehicle {
  drive(): void;
}

class Car implements Vehicle {
  drive() {
    console.log("Driving car");
  }
}

class Truck implements Vehicle {
  drive() {
    console.log("Driving truck");
  }
}

class VehicleFactory {
  static create(type: "car" | "truck"): Vehicle {
    switch (type) {
      case "car":
        return new Car();
      case "truck":
        return new Truck();
      default:
        throw new Error("Unknown vehicle type");
    }
  }
}

// Usage
const vehicle = VehicleFactory.create("car");
vehicle.drive();
```

#### Builder Pattern

Construct complex objects step by step.

```typescript
class QueryBuilder {
  private query: string[] = ["SELECT"];
  private fields: string[] = ["*"];
  private tableName: string = "";
  private conditions: string[] = [];
  private orderByClause: string = "";

  select(...fields: string[]): this {
    this.fields = fields;
    return this;
  }

  from(table: string): this {
    this.tableName = table;
    return this;
  }

  where(condition: string): this {
    this.conditions.push(condition);
    return this;
  }

  orderBy(field: string, direction: "ASC" | "DESC" = "ASC"): this {
    this.orderByClause = `${field} ${direction}`;
    return this;
  }

  build(): string {
    let sql = `SELECT ${this.fields.join(", ")} FROM ${this.tableName}`;
    if (this.conditions.length > 0) {
      sql += ` WHERE ${this.conditions.join(" AND ")}`;
    }
    if (this.orderByClause) {
      sql += ` ORDER BY ${this.orderByClause}`;
    }
    return sql;
  }
}

// Usage
const query = new QueryBuilder()
  .select("id", "name", "email")
  .from("users")
  .where("age > 18")
  .where("active = true")
  .orderBy("name", "ASC")
  .build();
```

#### Singleton Pattern

Ensure only one instance exists (use sparingly).

```typescript
class DatabaseConnection {
  private static instance: DatabaseConnection;
  private connection: any;

  private constructor() {
    // Private constructor prevents direct instantiation
    this.connection = this.createConnection();
  }

  static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }

  private createConnection() {
    // Connection logic
  }

  query(sql: string) {
    return this.connection.execute(sql);
  }
}

// Usage
const db = DatabaseConnection.getInstance();
```

**Note**: Singletons can make testing difficult. Consider dependency injection instead.

### Structural Patterns

#### Adapter Pattern

Make incompatible interfaces work together.

```typescript
// Third-party library with different interface
class LegacyPaymentGateway {
  makePayment(accountNumber: string, amount: number) {
    // Legacy implementation
  }
}

// Our application's interface
interface PaymentProcessor {
  processPayment(payment: Payment): Promise<PaymentResult>;
}

// Adapter
class LegacyPaymentAdapter implements PaymentProcessor {
  constructor(private legacyGateway: LegacyPaymentGateway) {}

  async processPayment(payment: Payment): Promise<PaymentResult> {
    // Adapt our interface to legacy interface
    this.legacyGateway.makePayment(payment.accountNumber, payment.amount);
    return { success: true, transactionId: "..." };
  }
}
```

#### Decorator Pattern

Add behavior to objects dynamically.

```typescript
interface Coffee {
  cost(): number;
  description(): string;
}

class SimpleCoffee implements Coffee {
  cost() {
    return 5;
  }
  description() {
    return "Simple coffee";
  }
}

// Decorators
class MilkDecorator implements Coffee {
  constructor(private coffee: Coffee) {}
  cost() {
    return this.coffee.cost() + 1;
  }
  description() {
    return this.coffee.description() + ", milk";
  }
}

class SugarDecorator implements Coffee {
  constructor(private coffee: Coffee) {}
  cost() {
    return this.coffee.cost() + 0.5;
  }
  description() {
    return this.coffee.description() + ", sugar";
  }
}

// Usage
let coffee: Coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
console.log(coffee.description()); // "Simple coffee, milk, sugar"
console.log(coffee.cost()); // 6.5
```

#### Facade Pattern

Provide simplified interface to complex subsystem.

```typescript
// Complex subsystems
class VideoConverter {
  convert(file: File, format: string) {
    /* ... */
  }
}

class AudioExtractor {
  extract(file: File) {
    /* ... */
  }
}

class CompressionService {
  compress(data: Buffer) {
    /* ... */
  }
}

// Facade
class MediaProcessor {
  constructor(
    private converter: VideoConverter,
    private audioExtractor: AudioExtractor,
    private compressor: CompressionService,
  ) {}

  async processVideo(file: File, format: string): Promise<Buffer> {
    // Simplified interface hiding complexity
    const converted = await this.converter.convert(file, format);
    const audio = await this.audioExtractor.extract(converted);
    const compressed = await this.compressor.compress(audio);
    return compressed;
  }
}

// Usage
const processor = new MediaProcessor(
  new VideoConverter(),
  new AudioExtractor(),
  new CompressionService(),
);
await processor.processVideo(file, "mp4");
```

### Behavioral Patterns

#### Strategy Pattern

Define family of algorithms, make them interchangeable.

```typescript
interface SortStrategy {
  sort(data: number[]): number[];
}

class QuickSort implements SortStrategy {
  sort(data: number[]): number[] {
    // QuickSort implementation
    return data;
  }
}

class MergeSort implements SortStrategy {
  sort(data: number[]): number[] {
    // MergeSort implementation
    return data;
  }
}

class BubbleSort implements SortStrategy {
  sort(data: number[]): number[] {
    // BubbleSort implementation
    return data;
  }
}

class DataSorter {
  constructor(private strategy: SortStrategy) {}

  setStrategy(strategy: SortStrategy) {
    this.strategy = strategy;
  }

  sort(data: number[]): number[] {
    return this.strategy.sort(data);
  }
}

// Usage
const sorter = new DataSorter(new QuickSort());
sorter.sort([3, 1, 4, 1, 5]);

// Switch strategy at runtime
sorter.setStrategy(new MergeSort());
```

#### Observer Pattern

Define one-to-many dependency for event notifications.

```typescript
interface Observer {
  update(data: any): void;
}

class Subject {
  private observers: Observer[] = [];

  attach(observer: Observer): void {
    this.observers.push(observer);
  }

  detach(observer: Observer): void {
    const index = this.observers.indexOf(observer);
    if (index > -1) this.observers.splice(index, 1);
  }

  notify(data: any): void {
    this.observers.forEach((observer) => observer.update(data));
  }
}

// Concrete implementations
class EmailNotifier implements Observer {
  update(data: any) {
    console.log("Sending email:", data);
  }
}

class SMSNotifier implements Observer {
  update(data: any) {
    console.log("Sending SMS:", data);
  }
}

// Usage
const orderSubject = new Subject();
orderSubject.attach(new EmailNotifier());
orderSubject.attach(new SMSNotifier());

orderSubject.notify({ orderId: 123, status: "shipped" });
```

#### Command Pattern

Encapsulate requests as objects.

```typescript
interface Command {
  execute(): void;
  undo(): void;
}

class Light {
  turnOn() {
    console.log("Light on");
  }
  turnOff() {
    console.log("Light off");
  }
}

class LightOnCommand implements Command {
  constructor(private light: Light) {}
  execute() {
    this.light.turnOn();
  }
  undo() {
    this.light.turnOff();
  }
}

class LightOffCommand implements Command {
  constructor(private light: Light) {}
  execute() {
    this.light.turnOff();
  }
  undo() {
    this.light.turnOn();
  }
}

class RemoteControl {
  private history: Command[] = [];

  executeCommand(command: Command) {
    command.execute();
    this.history.push(command);
  }

  undo() {
    const command = this.history.pop();
    if (command) command.undo();
  }
}

// Usage
const light = new Light();
const remote = new RemoteControl();

remote.executeCommand(new LightOnCommand(light));
remote.executeCommand(new LightOffCommand(light));
remote.undo(); // Turns light back on
```

#### Repository Pattern

Separate data access logic from business logic.

```typescript
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
}

class UserRepository implements Repository<User> {
  constructor(private db: Database) {}

  async findById(id: string): Promise<User | null> {
    const row = await this.db.query("SELECT * FROM users WHERE id = ?", [id]);
    return row ? this.mapToUser(row) : null;
  }

  async findAll(): Promise<User[]> {
    const rows = await this.db.query("SELECT * FROM users");
    return rows.map(this.mapToUser);
  }

  async save(user: User): Promise<User> {
    if (user.id) {
      await this.db.query("UPDATE users SET name = ?, email = ? WHERE id = ?", [
        user.name,
        user.email,
        user.id,
      ]);
    } else {
      const result = await this.db.query(
        "INSERT INTO users (name, email) VALUES (?, ?)",
        [user.name, user.email],
      );
      user.id = result.insertId;
    }
    return user;
  }

  async delete(id: string): Promise<void> {
    await this.db.query("DELETE FROM users WHERE id = ?", [id]);
  }

  private mapToUser(row: any): User {
    return new User(row.id, row.name, row.email);
  }
}
```

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

## Cross-Cutting Concerns

### Security

#### Input Validation

**Always validate and sanitize user input.**

```typescript
// ✅ Good - Validation at boundary
class CreateUserDTO {
  @IsEmail()
  email: string;

  @MinLength(8)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
  password: string;

  @IsString()
  @MinLength(2)
  @MaxLength(50)
  name: string;
}

class UserController {
  async createUser(@Body() dto: CreateUserDTO) {
    // Input already validated by framework
    const user = await this.userService.create(dto);
    return user;
  }
}
```

#### SQL Injection Prevention

**Always use parameterized queries.**

```typescript
// ❌ Bad - SQL injection vulnerability
const query = `SELECT * FROM users WHERE email = '${email}'`;

// ✅ Good - Parameterized query
const query = "SELECT * FROM users WHERE email = ?";
const result = await db.query(query, [email]);

// ✅ Good - ORM
const user = await User.findOne({ where: { email } });
```

#### XSS Prevention

**Escape output, validate input, use Content Security Policy.**

```typescript
// ✅ Good - Escape HTML
function escapeHtml(unsafe: string): string {
  return unsafe
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");
}

// Use framework's built-in escaping
// React automatically escapes by default
<div>{ userProvidedContent } < /div>; / / Safe in React;

// Set CSP headers
app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
    },
  }),
);
```

#### Authentication & Authorization

```typescript
// Authentication - Verify identity
class AuthService {
  async login(email: string, password: string): Promise<TokenPair> {
    const user = await this.userRepo.findByEmail(email);
    if (!user) throw new UnauthorizedException();

    const isValid = await bcrypt.compare(password, user.passwordHash);
    if (!isValid) throw new UnauthorizedException();

    return this.generateTokens(user);
  }

  private generateTokens(user: User): TokenPair {
    const accessToken = jwt.sign(
      { userId: user.id, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: "15m" },
    );

    const refreshToken = jwt.sign(
      { userId: user.id },
      process.env.REFRESH_SECRET,
      { expiresIn: "7d" },
    );

    return { accessToken, refreshToken };
  }
}

// Authorization - Verify permissions
function requireRole(...roles: string[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: "Not authenticated" });
    }

    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: "Insufficient permissions" });
    }

    next();
  };
}

// Usage
app.delete("/users/:id", requireRole("admin"), deleteUser);
```

#### Secrets Management

**Never commit secrets to version control.**

```typescript
// ❌ Bad
const API_KEY = 'sk-1234567890abcdef';

// ✅ Good - Environment variables
const API_KEY = process.env.API_KEY;

// ✅ Good - Secret management service
const secrets = await secretsManager.getSecrets();
const API_KEY = secrets.API_KEY;

// .env file (in .gitignore)
API_KEY=sk-1234567890abcdef
DATABASE_URL=postgresql://user:pass@localhost/db
```

#### Rate Limiting

```typescript
import rateLimit from "express-rate-limit";

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: "Too many requests from this IP",
});

app.use("/api/", limiter);

// Stricter for sensitive endpoints
const strictLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
});

app.use("/api/auth/login", strictLimiter);
```

### Performance

#### Caching Strategy

```typescript
// In-memory cache
class CacheService {
  private cache = new Map<string, { value: any; expiry: number }>();

  set(key: string, value: any, ttlSeconds: number): void {
    this.cache.set(key, {
      value,
      expiry: Date.now() + ttlSeconds * 1000,
    });
  }

  get(key: string): any | null {
    const item = this.cache.get(key);
    if (!item) return null;

    if (Date.now() > item.expiry) {
      this.cache.delete(key);
      return null;
    }

    return item.value;
  }

  delete(key: string): void {
    this.cache.delete(key);
  }
}

// Usage with cache-aside pattern
class UserService {
  constructor(
    private userRepo: UserRepository,
    private cache: CacheService,
  ) {}

  async getUser(id: string): Promise<User> {
    // Try cache first
    const cached = this.cache.get(`user:${id}`);
    if (cached) return cached;

    // Cache miss - fetch from DB
    const user = await this.userRepo.findById(id);

    // Store in cache
    this.cache.set(`user:${id}`, user, 300); // 5 minutes

    return user;
  }

  async updateUser(id: string, data: Partial<User>): Promise<User> {
    const user = await this.userRepo.update(id, data);

    // Invalidate cache
    this.cache.delete(`user:${id}`);

    return user;
  }
}
```

#### Database Optimization

```typescript
// ✅ Good - Use indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

// ✅ Good - Use select specific columns
const users = await db.query(
  'SELECT id, name, email FROM users WHERE active = true'
);

// ❌ Bad - SELECT *
const users = await db.query('SELECT * FROM users');

// ✅ Good - Use pagination
async function getUsers(page: number, pageSize: number) {
  const offset = (page - 1) * pageSize;
  return db.query(
    'SELECT * FROM users LIMIT ? OFFSET ?',
    [pageSize, offset]
  );
}

// ✅ Good - Eager loading to avoid N+1 queries
const users = await User.findAll({
  include: [{ model: Order }] // Load orders in same query
});

// ❌ Bad - N+1 query problem
const users = await User.findAll();
for (const user of users) {
  const orders = await Order.findAll({ where: { userId: user.id } });
  // This runs N queries!
}
```

#### Lazy Loading

```typescript
class User {
  private _orders: Order[] | null = null;

  async getOrders(): Promise<Order[]> {
    if (this._orders === null) {
      this._orders = await OrderRepository.findByUserId(this.id);
    }
    return this._orders;
  }
}
```

#### Background Jobs

```typescript
// Use job queue for long-running tasks
class EmailService {
  constructor(private queue: JobQueue) {}

  async sendWelcomeEmail(userId: string): Promise<void> {
    // Don't block the request
    await this.queue.add("send-email", {
      userId,
      template: "welcome",
    });
  }
}

// Worker processes job
class EmailWorker {
  async process(job: Job) {
    const { userId, template } = job.data;
    const user = await UserRepository.findById(userId);
    await this.sendEmail(user.email, template);
  }
}
```

### Observability

#### Logging

```typescript
// Structured logging
class Logger {
  info(message: string, context?: Record<string, any>): void {
    console.log(
      JSON.stringify({
        level: "info",
        message,
        timestamp: new Date().toISOString(),
        ...context,
      }),
    );
  }

  error(message: string, error: Error, context?: Record<string, any>): void {
    console.error(
      JSON.stringify({
        level: "error",
        message,
        error: {
          name: error.name,
          message: error.message,
          stack: error.stack,
        },
        timestamp: new Date().toISOString(),
        ...context,
      }),
    );
  }
}

// Usage
logger.info("User created", {
  userId: user.id,
  email: user.email,
});

logger.error("Failed to process order", error, {
  orderId: order.id,
  customerId: customer.id,
});
```

#### Metrics

```typescript
class MetricsService {
  private counters = new Map<string, number>();
  private gauges = new Map<string, number>();

  incrementCounter(name: string, value: number = 1): void {
    const current = this.counters.get(name) || 0;
    this.counters.set(name, current + value);
  }

  setGauge(name: string, value: number): void {
    this.gauges.set(name, value);
  }

  recordTiming(name: string, durationMs: number): void {
    // Record timing metric
  }
}

// Usage
metrics.incrementCounter("api.requests.total");
metrics.incrementCounter("api.requests.errors");
metrics.setGauge("api.active_connections", activeConnections);

// Timing
const start = Date.now();
await processRequest();
metrics.recordTiming("api.request.duration", Date.now() - start);
```

#### Distributed Tracing

```typescript
class TracingService {
  startSpan(name: string, parentSpanId?: string): Span {
    return {
      id: generateId(),
      parentId: parentSpanId,
      name,
      startTime: Date.now(),
      attributes: {},
    };
  }

  endSpan(span: Span): void {
    span.endTime = Date.now();
    span.duration = span.endTime - span.startTime;
    this.sendToTracing(span);
  }
}

// Usage
const span = tracing.startSpan("process-order");
span.attributes.orderId = order.id;

try {
  await processOrder(order);
  span.attributes.status = "success";
} catch (error) {
  span.attributes.status = "error";
  span.attributes.error = error.message;
  throw error;
} finally {
  tracing.endSpan(span);
}
```

#### Health Checks

```typescript
class HealthCheckService {
  async check(): Promise<HealthStatus> {
    const checks = await Promise.allSettled([
      this.checkDatabase(),
      this.checkRedis(),
      this.checkExternalAPI(),
    ]);

    const status = checks.every((c) => c.status === "fulfilled")
      ? "healthy"
      : "unhealthy";

    return {
      status,
      timestamp: new Date().toISOString(),
      checks: {
        database: checks[0].status === "fulfilled" ? "up" : "down",
        redis: checks[1].status === "fulfilled" ? "up" : "down",
        externalAPI: checks[2].status === "fulfilled" ? "up" : "down",
      },
    };
  }

  private async checkDatabase(): Promise<void> {
    await db.query("SELECT 1");
  }
}

// Endpoint
app.get("/health", async (req, res) => {
  const health = await healthCheck.check();
  const statusCode = health.status === "healthy" ? 200 : 503;
  res.status(statusCode).json(health);
});
```

### Error Handling

#### Error Hierarchy

```typescript
// Base error class
class ApplicationError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500,
    public isOperational: boolean = true,
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

// Specific errors
class ValidationError extends ApplicationError {
  constructor(
    message: string,
    public fields?: Record<string, string>,
  ) {
    super(message, "VALIDATION_ERROR", 400);
  }
}

class NotFoundError extends ApplicationError {
  constructor(resource: string, id: string) {
    super(`${resource} with id ${id} not found`, "NOT_FOUND", 404);
  }
}

class UnauthorizedError extends ApplicationError {
  constructor(message: string = "Unauthorized") {
    super(message, "UNAUTHORIZED", 401);
  }
}

class ForbiddenError extends ApplicationError {
  constructor(message: string = "Forbidden") {
    super(message, "FORBIDDEN", 403);
  }
}

// Usage
throw new NotFoundError("User", userId);
throw new ValidationError("Invalid email format", {
  email: "Must be valid email",
});
```

#### Global Error Handler

```typescript
function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction,
) {
  // Log error
  logger.error("Error occurred", err, {
    method: req.method,
    path: req.path,
    body: req.body,
  });

  // Handle known errors
  if (err instanceof ApplicationError) {
    return res.status(err.statusCode).json({
      error: {
        code: err.code,
        message: err.message,
        ...(err instanceof ValidationError && { fields: err.fields }),
      },
    });
  }

  // Handle unknown errors
  return res.status(500).json({
    error: {
      code: "INTERNAL_ERROR",
      message: "An unexpected error occurred",
    },
  });
}

app.use(errorHandler);
```

#### Retry Logic

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  delayMs: number = 1000,
): Promise<T> {
  let lastError: Error;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;

      if (attempt < maxRetries) {
        await sleep(delayMs * attempt); // Exponential backoff
        continue;
      }
    }
  }

  throw lastError!;
}

// Usage
const data = await withRetry(() => fetchFromExternalAPI(), 3, 1000);
```

---

## Database Design

### Schema Design Principles

```sql
-- ✅ Good - Normalized design
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  total_amount DECIMAL(10, 2) NOT NULL,
  status VARCHAR(20) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE order_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id UUID NOT NULL REFERENCES products(id),
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  price DECIMAL(10, 2) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for common queries
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```

### Denormalization When Appropriate

```sql
-- Sometimes denormalization improves read performance
CREATE TABLE order_summary (
  order_id UUID PRIMARY KEY REFERENCES orders(id),
  user_email VARCHAR(255), -- Denormalized from users table
  item_count INTEGER,
  total_amount DECIMAL(10, 2),
  -- Frequently accessed together, rarely updated
);

-- Keep denormalized data in sync with triggers or application logic
```

### Soft Deletes

```sql
-- Add deleted_at column instead of hard deleting
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP NULL;

-- Query active users
SELECT * FROM users WHERE deleted_at IS NULL;

-- Create index for better performance
CREATE INDEX idx_users_deleted_at ON users(deleted_at);
```

### Audit Trail

```sql
CREATE TABLE audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  table_name VARCHAR(50) NOT NULL,
  record_id UUID NOT NULL,
  action VARCHAR(20) NOT NULL, -- INSERT, UPDATE, DELETE
  old_values JSONB,
  new_values JSONB,
  changed_by UUID REFERENCES users(id),
  changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_log_table_record ON audit_log(table_name, record_id);
CREATE INDEX idx_audit_log_changed_at ON audit_log(changed_at);
```

---

## API Design

### RESTful Best Practices

```typescript
// ✅ Good - Resource-based URLs
GET    /api/users              // List users
GET    /api/users/:id          // Get specific user
POST   /api/users              // Create user
PUT    /api/users/:id          // Update user (full)
PATCH  /api/users/:id          // Update user (partial)
DELETE /api/users/:id          // Delete user

// Nested resources
GET    /api/users/:id/orders   // Get user's orders
POST   /api/users/:id/orders   // Create order for user

// Filtering, sorting, pagination
GET    /api/users?role=admin&sort=created_at&order=desc&page=1&limit=20

// ❌ Bad - Action-based URLs
POST   /api/createUser
POST   /api/deleteUser
GET    /api/getUserById
```

### Response Structure

```typescript
// Success response
{
  "success": true,
  "data": {
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z"
  }
}

// Error response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "fields": {
      "email": "Email is required",
      "password": "Password must be at least 8 characters"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_abc123"
  }
}

// List response with pagination
{
  "success": true,
  "data": [...],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

### Versioning

```typescript
// Option 1: URL versioning
app.use("/api/v1", v1Router);
app.use("/api/v2", v2Router);

// Option 2: Header versioning
app.use((req, res, next) => {
  const version = req.headers["api-version"] || "1";
  req.apiVersion = version;
  next();
});

// Option 3: Accept header
// Accept: application/vnd.myapi.v2+json
```

### Authentication

```typescript
// Bearer token authentication
app.use("/api", (req, res, next) => {
  const authHeader = req.headers.authorization;

  if (!authHeader?.startsWith("Bearer ")) {
    return res.status(401).json({ error: "Missing or invalid token" });
  }

  const token = authHeader.substring(7);

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET);
    req.user = payload;
    next();
  } catch (error) {
    return res.status(401).json({ error: "Invalid token" });
  }
});
```

### Rate Limiting Headers

```typescript
res.setHeader("X-RateLimit-Limit", "100");
res.setHeader("X-RateLimit-Remaining", "95");
res.setHeader("X-RateLimit-Reset", "1640995200");
```

---

## Migration Strategies

### Database Migrations

```typescript
// Migration: Add new column
export async function up(db: Database): Promise<void> {
  await db.query(`
    ALTER TABLE users 
    ADD COLUMN phone_number VARCHAR(20)
  `);
}

export async function down(db: Database): Promise<void> {
  await db.query(`
    ALTER TABLE users 
    DROP COLUMN phone_number
  `);
}

// Migration: Rename column (backward compatible)
// Step 1: Add new column
await db.query("ALTER TABLE users ADD COLUMN full_name VARCHAR(100)");

// Step 2: Copy data
await db.query("UPDATE users SET full_name = name");

// Step 3: Deploy application code that uses full_name
// (Deploy and wait)

// Step 4: Drop old column
await db.query("ALTER TABLE users DROP COLUMN name");
```

### Zero-Downtime Deployment

```typescript
// Expand-Contract pattern for breaking changes

// Phase 1: EXPAND - Add new field, support both old and new
class User {
  name: string; // Old field (deprecated)
  fullName: string; // New field

  getName(): string {
    return this.fullName || this.name; // Support both
  }
}

// Phase 2: MIGRATE - Gradually migrate data
async function migrateUserNames() {
  const users = await User.find({ fullName: null });
  for (const user of users) {
    user.fullName = user.name;
    await user.save();
  }
}

// Phase 3: CONTRACT - Remove old field
class User {
  fullName: string; // Only new field remains

  getName(): string {
    return this.fullName;
  }
}
```

### Feature Flags

```typescript
class FeatureFlags {
  private flags = new Map<string, boolean>();

  isEnabled(flagName: string, userId?: string): boolean {
    // Check user-specific override
    if (userId) {
      const userFlag = this.flags.get(`${flagName}:${userId}`);
      if (userFlag !== undefined) return userFlag;
    }

    // Check global flag
    return this.flags.get(flagName) || false;
  }

  enable(flagName: string, userId?: string): void {
    const key = userId ? `${flagName}:${userId}` : flagName;
    this.flags.set(key, true);
  }
}

// Usage
const flags = new FeatureFlags();

if (flags.isEnabled("new-checkout-flow", user.id)) {
  // Use new implementation
  return newCheckoutService.process(order);
} else {
  // Use old implementation
  return legacyCheckoutService.process(order);
}
```

---

## Code Organization

### Directory Structure (Node.js/TypeScript)

```
src/
├── domain/              # Business logic
│   ├── entities/
│   │   ├── User.ts
│   │   └── Order.ts
│   ├── value-objects/
│   │   ├── Email.ts
│   │   └── Money.ts
│   └── repositories/
│       ├── UserRepository.ts
│       └── OrderRepository.ts
│
├── application/         # Use cases
│   ├── services/
│   │   ├── UserService.ts
│   │   └── OrderService.ts
│   └── dto/
│       ├── CreateUserDTO.ts
│       └── CreateOrderDTO.ts
│
├── infrastructure/      # Technical implementations
│   ├── database/
│   │   ├── PostgresUserRepository.ts
│   │   └── migrations/
│   ├── external/
│   │   ├── PaymentGateway.ts
│   │   └── EmailService.ts
│   └── config/
│       └── database.ts
│
├── presentation/        # HTTP/API layer
│   ├── controllers/
│   │   ├── UserController.ts
│   │   └── OrderController.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   └── errorHandler.ts
│   └── routes/
│       ├── userRoutes.ts
│       └── orderRoutes.ts
│
├── shared/             # Shared utilities
│   ├── types/
│   ├── utils/
│   └── constants/
│
└── index.ts            # Application entry point
```

### Module Boundaries

```typescript
// ✅ Good - Clear module boundaries
// user/index.ts - Public API of user module
export { UserService } from "./services/UserService";
export { CreateUserDTO } from "./dto/CreateUserDTO";
export type { User } from "./entities/User";

// Other modules import from public API only
import { UserService } from "@/modules/user";

// ❌ Bad - Reaching into module internals
import { UserRepository } from "@/modules/user/repositories/UserRepository";
```

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

## Workflow for Complex Changes

### 1. Understand Current State

- Read existing code thoroughly
- Identify pain points and technical debt
- Document current architecture
- List dependencies and integrations

### 2. Define Desired State

- Clearly state the goal
- Define success criteria
- Consider non-functional requirements
- Identify constraints

### 3. Plan the Approach

- Break down into phases
- Identify risks and mitigations
- Plan for backward compatibility
- Design migration strategy
- Consider rollback plan

### 4. Design the Solution

- Choose appropriate patterns
- Define interfaces and contracts
- Plan data model changes
- Design API changes
- Consider security implications
- Plan for scalability

### 5. Implement Incrementally

- Make smallest possible changes
- Deploy frequently
- Use feature flags for risky changes
- Write tests first (TDD)
- Keep master branch deployable

### 6. Verify and Monitor

- Write comprehensive tests
- Perform code review
- Deploy to staging first
- Monitor metrics in production
- Have rollback plan ready

### 7. Document

- Update architecture docs
- Document design decisions
- Update API documentation
- Add code comments for complex logic
- Document migration procedures

---

## Example: Refactoring Legacy Code

### Before (Legacy)

```typescript
// monolithic-service.ts
export async function handleUserCreation(req: any, res: any) {
  // Validation
  if (!req.body.email || !req.body.email.includes("@")) {
    return res.status(400).send("Invalid email");
  }

  // Check duplicates
  const existing = await db.query(
    `SELECT * FROM users WHERE email = '${req.body.email}'`,
  );
  if (existing.length > 0) {
    return res.status(409).send("User exists");
  }

  // Hash password
  const hashedPassword = crypto
    .createHash("md5")
    .update(req.body.password)
    .digest("hex");

  // Insert user
  await db.query(
    `INSERT INTO users (email, password, created_at) 
     VALUES ('${req.body.email}', '${hashedPassword}', NOW())`,
  );

  // Send email
  await fetch("https://emailapi.com/send", {
    method: "POST",
    body: JSON.stringify({
      to: req.body.email,
      subject: "Welcome",
      body: "Welcome to our service!",
    }),
  });

  res.status(201).send("User created");
}
```

**Problems**:

- SQL injection vulnerability
- Weak password hashing (MD5)
- Mixed concerns (validation, DB, email)
- Hard to test
- No error handling
- No logging

### After (Refactored)

```typescript
// Domain
class User {
  constructor(
    public id: string,
    public email: Email,
    public passwordHash: string,
    public createdAt: Date,
  ) {}

  static async create(email: Email, password: Password): Promise<User> {
    const passwordHash = await bcrypt.hash(password.value, 10);
    return new User(generateId(), email, passwordHash, new Date());
  }
}

class Email {
  constructor(public readonly value: string) {
    if (!this.isValid(value)) {
      throw new ValidationError("Invalid email format");
    }
  }

  private isValid(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
}

class Password {
  constructor(public readonly value: string) {
    if (value.length < 8) {
      throw new ValidationError("Password must be at least 8 characters");
    }
  }
}

// Repository
interface UserRepository {
  findByEmail(email: Email): Promise<User | null>;
  save(user: User): Promise<void>;
}

class PostgresUserRepository implements UserRepository {
  constructor(private db: Database) {}

  async findByEmail(email: Email): Promise<User | null> {
    const result = await this.db.query("SELECT * FROM users WHERE email = $1", [
      email.value,
    ]);
    return result.rows[0] ? this.mapToUser(result.rows[0]) : null;
  }

  async save(user: User): Promise<void> {
    await this.db.query(
      "INSERT INTO users (id, email, password_hash, created_at) VALUES ($1, $2, $3, $4)",
      [user.id, user.email.value, user.passwordHash, user.createdAt],
    );
  }

  private mapToUser(row: any): User {
    return new User(
      row.id,
      new Email(row.email),
      row.password_hash,
      row.created_at,
    );
  }
}

// Service
class UserService {
  constructor(
    private userRepo: UserRepository,
    private emailService: EmailService,
    private logger: Logger,
  ) {}

  async createUser(emailStr: string, passwordStr: string): Promise<User> {
    this.logger.info("Creating user", { email: emailStr });

    // Validate
    const email = new Email(emailStr);
    const password = new Password(passwordStr);

    // Check duplicates
    const existing = await this.userRepo.findByEmail(email);
    if (existing) {
      throw new ConflictError("User already exists");
    }

    // Create user
    const user = await User.create(email, password);
    await this.userRepo.save(user);

    // Send welcome email (async, don't wait)
    this.emailService.sendWelcome(user.email).catch((error) => {
      this.logger.error("Failed to send welcome email", error, {
        userId: user.id,
      });
    });

    this.logger.info("User created successfully", { userId: user.id });
    return user;
  }
}

// Controller
class UserController {
  constructor(private userService: UserService) {}

  async createUser(req: Request, res: Response, next: NextFunction) {
    try {
      const { email, password } = req.body;
      const user = await this.userService.createUser(email, password);

      res.status(201).json({
        success: true,
        data: { id: user.id, email: user.email.value },
      });
    } catch (error) {
      next(error); // Let error handler deal with it
    }
  }
}
```

**Improvements**:

- Secure (parameterized queries, bcrypt)
- Separated concerns (layers)
- Testable (dependency injection)
- Type-safe (value objects)
- Proper error handling
- Logging
- Clear architecture

---

## Final Principles

1. **Favor composition over inheritance**
2. **Program to interfaces, not implementations**
3. **Keep it simple** - Don't over-engineer
4. **Make it work, make it right, make it fast** - In that order
5. **Test early and often**
6. **Security is not optional**
7. **Document decisions, not just code**
8. **Optimize for readability** - Code is read more than written
9. **Fail fast** - Validate early, throw errors early
10. **Monitor everything** - You can't improve what you don't measure

---

## Quick Reference Checklist

When reviewing/designing code, check:

**Architecture**

- [ ] Clear separation of concerns
- [ ] Appropriate use of design patterns
- [ ] Loose coupling, high cohesion
- [ ] SOLID principles followed
- [ ] Dependencies point inward

**Security**

- [ ] Input validation at boundaries
- [ ] Parameterized queries (no SQL injection)
- [ ] Output encoding (no XSS)
- [ ] Authentication & authorization
- [ ] Secrets not in code
- [ ] Rate limiting on APIs

**Performance**

- [ ] Appropriate caching strategy
- [ ] Database queries optimized
- [ ] No N+1 query problems
- [ ] Async for I/O operations
- [ ] Background jobs for slow operations

**Reliability**

- [ ] Comprehensive error handling
- [ ] Retry logic for transient failures
- [ ] Circuit breakers for external services
- [ ] Graceful degradation
- [ ] Health checks implemented

**Observability**

- [ ] Structured logging
- [ ] Metrics collection
- [ ] Distributed tracing
- [ ] Alerts configured

**Maintainability**

- [ ] Clear, self-documenting code
- [ ] Comprehensive tests
- [ ] Documentation updated
- [ ] No magic numbers/strings
- [ ] Consistent naming conventions

**Scalability**

- [ ] Stateless where possible
- [ ] Horizontal scaling capability
- [ ] Database indexes on foreign keys
- [ ] Pagination for large datasets
- [ ] Async processing for heavy loads
