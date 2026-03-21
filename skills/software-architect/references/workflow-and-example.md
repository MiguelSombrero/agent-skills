Reference for software-architect skill. See [SKILL.md](../SKILL.md) for core instructions.
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

