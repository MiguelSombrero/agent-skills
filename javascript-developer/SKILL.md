---
name: javascript-developer
description: Senior JavaScript/TypeScript developer specializing in modern web applications with expertise in React, Node.js, Express, Next.js, and clean architecture. Use when building full-stack applications, implementing REST APIs, designing testable code, or refactoring for maintainability. Focuses on TDD, TypeScript best practices, and modern patterns. Can adapt examples to Next.js, Svelte+Deno, or other modern frameworks.
---

# JavaScript Developer

Builds production-ready, full-stack web applications with emphasis on clean architecture, comprehensive testing, and maintainable code. Combines deep JavaScript/TypeScript knowledge with modern frameworks and testing best practices.

## When to Use This Skill

Use this skill when the user:
- Wants to **build full-stack applications** with React and Node.js
- Needs help with **REST API** design in Express or Next.js
- Asks about **React** components, hooks, or state management
- Wants to implement **authentication** or authorization
- Needs guidance on **testing** (unit, integration, e2e)
- Asks about **application architecture** or code organization
- Wants to **refactor code** for better maintainability
- Needs help with **TypeScript** types or patterns
- Asks about **database access** with Prisma or similar
- Wants to implement **server-side rendering** with Next.js
- Needs guidance on **async patterns** or error handling
- Asks "how should I structure..." or "what's the best way in React/Node.js..."

## Scope

**In Scope:**
- React and modern React patterns (hooks, context, composition)
- Node.js and Express for backend APIs
- Next.js for SSR/SSG applications
- TypeScript best practices
- Testing strategies (Jest, Vitest, Playwright)
- Clean architecture and layered design
- Database access patterns
- Authentication and authorization
- Error handling and validation
- Performance optimization
- Modern tooling (Vite, TypeScript, ESLint, Prettier)

**Can Adapt To:**
- **Next.js** - App Router, Server Components, API Routes
- **Svelte + Deno** - SvelteKit, Deno runtime patterns
- **Astro** - Content-focused sites with islands
- **Hono** - Edge-compatible API framework

**Out of Scope:**
- Native mobile development
- Game development
- Desktop applications (Electron)
- Legacy JavaScript (pre-ES6)

**Technology Stack:**
- **TypeScript 5.x** (strict mode)
- **React 18+** with modern patterns
- **Node.js 20+** LTS
- **Jest/Vitest** for testing
- **Playwright** for E2E tests
- **Prisma** for database access

---

## Core Principles

### SOLID Principles in JavaScript

**Single Responsibility**: Each module should have one reason to change. Controllers delegate to services; services orchestrate repositories and external services.

```typescript
// ✅ Good - Separated concerns
class UserController {
  constructor(private userService: UserService) {}
  async createUser(req: Request, res: Response) {
    const dto: CreateUserDTO = req.body;
    const user = await this.userService.createUser(dto);
    return res.status(201).json(user);
  }
}

class UserService {
  constructor(
    private userRepository: UserRepository,
    private passwordService: PasswordService
  ) {}
  async createUser(dto: CreateUserDTO): Promise<User> {
    const passwordHash = await this.passwordService.hash(dto.password);
    const user = await this.userRepository.create({
      email: dto.email,
      passwordHash,
      name: dto.name
    });
    return user;
  }
}
```

**Dependency Inversion**: Depend on abstractions, not concretions. Use interfaces; inject implementations.

```typescript
// ✅ Good - Program to interface
interface NotificationService {
  send(recipient: string, message: string): Promise<void>;
}

class OrderService {
  constructor(private notificationService: NotificationService) {}
  async placeOrder(order: Order): Promise<void> {
    await this.notificationService.send(order.customerEmail, 'Order placed');
  }
}
```

### Test-Driven Development (TDD)

**Red-Green-Refactor**: Write failing test first, implement minimum code to pass, then refactor. Use Arrange-Act-Assert.

```typescript
describe('Order', () => {
  it('should calculate total with tax', () => {
    const order = new Order([
      { name: 'Item 1', price: 10, quantity: 2 },
      { name: 'Item 2', price: 5, quantity: 1 }
    ]);
    expect(order.calculateTotal(0.1)).toBe(27.5);
  });
});
```

---

## Application Architecture

### Layered Architecture

```
┌─────────────────────────────────────────┐
│         Presentation Layer              │  Routes, Controllers, DTOs
├─────────────────────────────────────────┤
│         Application Layer               │  Services, Use Cases
├─────────────────────────────────────────┤
│         Domain Layer                    │  Entities, Value Objects
├─────────────────────────────────────────┤
│         Infrastructure Layer            │  Repositories, External APIs
└─────────────────────────────────────────┘
```

### Project Structure

```
src/
├── presentation/     # Routes, controllers, dto, middleware
├── application/     # Services, use-cases
├── domain/          # Entities, value-objects, repository interfaces, errors
├── infrastructure/  # Database, external clients, config
├── shared/          # Types, utils, constants
└── app.ts
```

### Rich Domain Model

Put business logic in domain entities. Use factory methods, enforce invariants, avoid anemic models.

```typescript
export class Order {
  static create(customerId: string, items: OrderItem[]): Order {
    if (items.length === 0) throw new Error('Order must have at least one item');
    const total = items.reduce((sum, item) => sum.add(item.subtotal), Money.zero('USD'));
    return new Order(crypto.randomUUID(), customerId, items, OrderStatus.PENDING, total, new Date());
  }
  
  addItem(item: OrderItem): void {
    if (this._status !== OrderStatus.PENDING) {
      throw new Error('Cannot add items to non-pending order');
    }
    this._items.push(item);
    this.recalculateTotal();
  }
  
  ship(): void {
    if (this._status !== OrderStatus.CONFIRMED) {
      throw new Error('Can only ship confirmed orders');
    }
    this._status = OrderStatus.SHIPPED;
  }
}
```

---

## Additional Resources

For detailed patterns and examples, see these reference files:

- **REST API, domain layer, repositories**: [rest-and-data.md](references/rest-and-data.md) — Controllers, routes, DTOs with Zod, error handling, rich domain models, repository pattern with Prisma
- **JWT authentication**: [security.md](references/security.md) — AuthService, authenticate/authorize middleware
- **Testing**: [testing.md](references/testing.md) — Unit, integration, React component, E2E with Playwright
- **React, hooks, forms, Next.js**: [frontend.md](references/frontend.md) — Components, custom hooks, form handling with react-hook-form, Next.js API routes and Server Components
- **Performance, anti-patterns, checklist**: [operations.md](references/operations.md) — Caching, database optimization, callback hell, prop drilling, quick reference checklist

---

## Summary

**Key Principles:**
- Test-driven development (TDD)
- SOLID principles in JavaScript/TypeScript
- Clean, maintainable code
- Rich domain models
- Comprehensive testing
- Type safety with TypeScript
- Modern patterns and tooling

**Remember**: Write tests first, keep functions small, use TypeScript strictly, and favor composition over complexity.
