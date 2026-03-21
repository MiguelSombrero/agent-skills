Reference for software-architect skill. See [SKILL.md](../SKILL.md) for core instructions.
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

