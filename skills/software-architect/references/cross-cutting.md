Reference for software-architect skill. See [SKILL.md](../SKILL.md) for core instructions.
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

