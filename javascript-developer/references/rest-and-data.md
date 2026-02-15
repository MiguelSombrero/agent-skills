# REST API, Domain Layer, and Repositories

Reference for javascript-developer skill. See [SKILL.md](../SKILL.md) for core instructions.

---

## REST API with Express

### Controller Pattern

```typescript
// presentation/controllers/user.controller.ts
import { Request, Response, NextFunction } from 'express';
import { UserService } from '@/application/services/user.service';
import { CreateUserDTO } from '@/presentation/dto/create-user.dto';
import { ApiError } from '@/shared/errors/api.error';

export class UserController {
  constructor(private userService: UserService) {}
  
  async getAll(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const page = parseInt(req.query.page as string) || 1;
      const limit = parseInt(req.query.limit as string) || 20;
      
      const result = await this.userService.findAll({ page, limit });
      
      res.json({
        data: result.items,
        pagination: {
          page: result.page,
          limit: result.limit,
          total: result.total,
          totalPages: Math.ceil(result.total / result.limit)
        }
      });
    } catch (error) {
      next(error);
    }
  }
  
  async getById(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const { id } = req.params;
      const user = await this.userService.findById(id);
      res.json({ data: user });
    } catch (error) {
      next(error);
    }
  }
  
  async create(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const dto: CreateUserDTO = req.body;
      const user = await this.userService.createUser(dto);
      res.status(201).json({ data: user });
    } catch (error) {
      next(error);
    }
  }
  
  async update(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const { id } = req.params;
      const user = await this.userService.updateUser(id, req.body);
      res.json({ data: user });
    } catch (error) {
      next(error);
    }
  }
  
  async delete(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const { id } = req.params;
      await this.userService.deleteUser(id);
      res.status(204).send();
    } catch (error) {
      next(error);
    }
  }
}

// presentation/routes/users.routes.ts
import { Router } from 'express';
import { UserController } from '@/presentation/controllers/user.controller';
import { validateDTO } from '@/presentation/middleware/validation.middleware';
import { authenticate } from '@/presentation/middleware/auth.middleware';
import { CreateUserDTO } from '@/presentation/dto/create-user.dto';

export function createUserRoutes(userController: UserController): Router {
  const router = Router();
  
  router.get('/', 
    authenticate,
    (req, res, next) => userController.getAll(req, res, next)
  );
  
  router.get('/:id', 
    authenticate,
    (req, res, next) => userController.getById(req, res, next)
  );
  
  router.post('/',
    validateDTO(CreateUserDTO),
    (req, res, next) => userController.create(req, res, next)
  );
  
  router.put('/:id',
    authenticate,
    (req, res, next) => userController.update(req, res, next)
  );
  
  router.delete('/:id',
    authenticate,
    (req, res, next) => userController.delete(req, res, next)
  );
  
  return router;
}
```

### DTOs and Validation

```typescript
// presentation/dto/create-user.dto.ts
import { z } from 'zod';

export const CreateUserSchema = z.object({
  email: z.string().email('Invalid email format'),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain uppercase letter')
    .regex(/[a-z]/, 'Password must contain lowercase letter')
    .regex(/[0-9]/, 'Password must contain number'),
  name: z.string().min(2).max(100),
  role: z.enum(['user', 'admin']).optional().default('user')
});

export type CreateUserDTO = z.infer<typeof CreateUserSchema>;

// presentation/middleware/validation.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { z, ZodError } from 'zod';
import { ValidationError } from '@/shared/errors/validation.error';

export function validateDTO<T extends z.ZodType>(schema: T) {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      req.body = schema.parse(req.body);
      next();
    } catch (error) {
      if (error instanceof ZodError) {
        const errors = error.errors.reduce((acc, err) => {
          const path = err.path.join('.');
          acc[path] = err.message;
          return acc;
        }, {} as Record<string, string>);
        
        next(new ValidationError('Validation failed', errors));
      } else {
        next(error);
      }
    }
  };
}
```

### Error Handling

```typescript
// shared/errors/api.error.ts
export class ApiError extends Error {
  constructor(
    public statusCode: number,
    message: string,
    public isOperational: boolean = true
  ) {
    super(message);
    Object.setPrototypeOf(this, ApiError.prototype);
  }
}

export class NotFoundError extends ApiError {
  constructor(resource: string, id: string) {
    super(404, `${resource} with id ${id} not found`);
  }
}

export class ValidationError extends ApiError {
  constructor(
    message: string,
    public errors: Record<string, string>
  ) {
    super(400, message);
  }
}

export class UnauthorizedError extends ApiError {
  constructor(message: string = 'Unauthorized') {
    super(401, message);
  }
}

// presentation/middleware/error-handler.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { ApiError, ValidationError } from '@/shared/errors/api.error';
import { logger } from '@/shared/utils/logger';

export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
): void {
  logger.error('Error occurred:', {
    error: err.message,
    stack: err.stack,
    path: req.path,
    method: req.method
  });
  
  if (err instanceof ValidationError) {
    res.status(err.statusCode).json({
      error: {
        message: err.message,
        code: 'VALIDATION_ERROR',
        fields: err.errors
      }
    });
    return;
  }
  
  if (err instanceof ApiError) {
    res.status(err.statusCode).json({
      error: {
        message: err.message,
        code: err.constructor.name
      }
    });
    return;
  }
  
  // Unknown errors
  res.status(500).json({
    error: {
      message: 'Internal server error',
      code: 'INTERNAL_ERROR'
    }
  });
}
```

---

## Domain Layer

### Rich Domain Models

```typescript
// domain/entities/order.entity.ts
import { OrderItem } from './order-item.entity';
import { Money } from '@/domain/value-objects/money.vo';

export enum OrderStatus {
  PENDING = 'PENDING',
  CONFIRMED = 'CONFIRMED',
  SHIPPED = 'SHIPPED',
  DELIVERED = 'DELIVERED',
  CANCELLED = 'CANCELLED'
}

export class Order {
  private constructor(
    public readonly id: string,
    public readonly customerId: string,
    private _items: OrderItem[],
    private _status: OrderStatus,
    private _total: Money,
    public readonly createdAt: Date,
    private _shippedAt?: Date,
    private _deliveredAt?: Date
  ) {}
  
  static create(customerId: string, items: OrderItem[]): Order {
    if (items.length === 0) {
      throw new Error('Order must have at least one item');
    }
    
    const total = items.reduce(
      (sum, item) => sum.add(item.subtotal),
      Money.zero('USD')
    );
    
    return new Order(
      crypto.randomUUID(),
      customerId,
      items,
      OrderStatus.PENDING,
      total,
      new Date()
    );
  }
  
  get items(): readonly OrderItem[] {
    return this._items;
  }
  
  get status(): OrderStatus {
    return this._status;
  }
  
  get total(): Money {
    return this._total;
  }
  
  addItem(item: OrderItem): void {
    if (this._status !== OrderStatus.PENDING) {
      throw new Error('Cannot add items to non-pending order');
    }
    
    this._items.push(item);
    this.recalculateTotal();
  }
  
  removeItem(itemId: string): void {
    if (this._status !== OrderStatus.PENDING) {
      throw new Error('Cannot remove items from non-pending order');
    }
    
    this._items = this._items.filter(item => item.id !== itemId);
    this.recalculateTotal();
  }
  
  confirm(): void {
    if (this._status !== OrderStatus.PENDING) {
      throw new Error('Can only confirm pending orders');
    }
    this._status = OrderStatus.CONFIRMED;
  }
  
  ship(): void {
    if (this._status !== OrderStatus.CONFIRMED) {
      throw new Error('Can only ship confirmed orders');
    }
    this._status = OrderStatus.SHIPPED;
    this._shippedAt = new Date();
  }
  
  deliver(): void {
    if (this._status !== OrderStatus.SHIPPED) {
      throw new Error('Can only deliver shipped orders');
    }
    this._status = OrderStatus.DELIVERED;
    this._deliveredAt = new Date();
  }
  
  cancel(): void {
    if (this._status === OrderStatus.DELIVERED) {
      throw new Error('Cannot cancel delivered order');
    }
    if (this._status === OrderStatus.CANCELLED) {
      throw new Error('Order is already cancelled');
    }
    this._status = OrderStatus.CANCELLED;
  }
  
  canBeCancelled(): boolean {
    return this._status !== OrderStatus.DELIVERED && 
           this._status !== OrderStatus.CANCELLED;
  }
  
  private recalculateTotal(): void {
    this._total = this._items.reduce(
      (sum, item) => sum.add(item.subtotal),
      Money.zero('USD')
    );
  }
}

// domain/value-objects/money.vo.ts
export class Money {
  private constructor(
    public readonly amount: number,
    public readonly currency: string
  ) {
    if (amount < 0) {
      throw new Error('Amount cannot be negative');
    }
  }
  
  static create(amount: number, currency: string): Money {
    return new Money(amount, currency);
  }
  
  static zero(currency: string): Money {
    return new Money(0, currency);
  }
  
  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Cannot add money with different currencies');
    }
    return new Money(this.amount + other.amount, this.currency);
  }
  
  subtract(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Cannot subtract money with different currencies');
    }
    return new Money(this.amount - other.amount, this.currency);
  }
  
  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }
  
  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }
  
  toJSON() {
    return {
      amount: this.amount,
      currency: this.currency
    };
  }
}
```

### Repository Pattern

```typescript
// domain/repositories/user.repository.ts (interface)
import { User } from '@/domain/entities/user.entity';

export interface UserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  findAll(options: PaginationOptions): Promise<PaginatedResult<User>>;
  save(user: User): Promise<User>;
  delete(id: string): Promise<void>;
}

export interface PaginationOptions {
  page: number;
  limit: number;
}

export interface PaginatedResult<T> {
  items: T[];
  total: number;
  page: number;
  limit: number;
}

// infrastructure/database/repositories/prisma-user.repository.ts
import { PrismaClient } from '@prisma/client';
import { UserRepository, PaginationOptions, PaginatedResult } from '@/domain/repositories/user.repository';
import { User } from '@/domain/entities/user.entity';

export class PrismaUserRepository implements UserRepository {
  constructor(private prisma: PrismaClient) {}
  
  async findById(id: string): Promise<User | null> {
    const dbUser = await this.prisma.user.findUnique({
      where: { id }
    });
    
    return dbUser ? this.toDomain(dbUser) : null;
  }
  
  async findByEmail(email: string): Promise<User | null> {
    const dbUser = await this.prisma.user.findUnique({
      where: { email }
    });
    
    return dbUser ? this.toDomain(dbUser) : null;
  }
  
  async findAll(options: PaginationOptions): Promise<PaginatedResult<User>> {
    const { page, limit } = options;
    const skip = (page - 1) * limit;
    
    const [users, total] = await Promise.all([
      this.prisma.user.findMany({
        skip,
        take: limit,
        orderBy: { createdAt: 'desc' }
      }),
      this.prisma.user.count()
    ]);
    
    return {
      items: users.map(u => this.toDomain(u)),
      total,
      page,
      limit
    };
  }
  
  async save(user: User): Promise<User> {
    const data = this.toPersistence(user);
    
    const dbUser = await this.prisma.user.upsert({
      where: { id: user.id },
      create: data,
      update: data
    });
    
    return this.toDomain(dbUser);
  }
  
  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({
      where: { id }
    });
  }
  
  private toDomain(dbUser: any): User {
    return User.reconstitute(
      dbUser.id,
      dbUser.email,
      dbUser.passwordHash,
      dbUser.name,
      dbUser.role,
      dbUser.active,
      dbUser.createdAt,
      dbUser.updatedAt
    );
  }
  
  private toPersistence(user: User): any {
    return {
      id: user.id,
      email: user.email,
      passwordHash: user.passwordHash,
      name: user.name,
      role: user.role,
      active: user.active,
      createdAt: user.createdAt,
      updatedAt: new Date()
    };
  }
}
```
