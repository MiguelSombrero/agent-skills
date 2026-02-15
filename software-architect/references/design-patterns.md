Reference for software-architect skill. See [SKILL.md](../SKILL.md) for core instructions.
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

