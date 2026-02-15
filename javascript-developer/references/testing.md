# Testing Strategies and Examples

Reference for javascript-developer skill. See [SKILL.md](../SKILL.md) for core instructions.

---

## Unit Tests (Lots of These!)

```typescript
// application/services/user.service.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { UserService } from './user.service';
import { UserRepository } from '@/domain/repositories/user.repository';
import { PasswordService } from './password.service';
import { EmailService } from './email.service';
import { User } from '@/domain/entities/user.entity';
import { NotFoundError } from '@/shared/errors/api.error';

// Mock dependencies
const mockUserRepository: UserRepository = {
  findById: vi.fn(),
  findByEmail: vi.fn(),
  findAll: vi.fn(),
  save: vi.fn(),
  delete: vi.fn()
};

const mockPasswordService: PasswordService = {
  hash: vi.fn(),
  compare: vi.fn()
};

const mockEmailService: EmailService = {
  sendWelcome: vi.fn(),
  sendPasswordReset: vi.fn()
};

describe('UserService', () => {
  let userService: UserService;
  
  beforeEach(() => {
    vi.clearAllMocks();
    userService = new UserService(
      mockUserRepository,
      mockPasswordService,
      mockEmailService
    );
  });
  
  describe('createUser', () => {
    it('should create user successfully', async () => {
      // Arrange
      const dto = {
        email: 'test@example.com',
        password: 'Password123',
        name: 'Test User'
      };
      
      const expectedUser = User.create(
        'test@example.com',
        'hashedPassword',
        'Test User'
      );
      
      vi.mocked(mockPasswordService.hash).mockResolvedValue('hashedPassword');
      vi.mocked(mockUserRepository.save).mockResolvedValue(expectedUser);
      vi.mocked(mockEmailService.sendWelcome).mockResolvedValue();
      
      // Act
      const result = await userService.createUser(dto);
      
      // Assert
      expect(result).toEqual(expectedUser);
      expect(mockPasswordService.hash).toHaveBeenCalledWith('Password123');
      expect(mockUserRepository.save).toHaveBeenCalled();
      expect(mockEmailService.sendWelcome).toHaveBeenCalledWith('test@example.com');
    });
    
    it('should throw error if email already exists', async () => {
      // Arrange
      const dto = {
        email: 'existing@example.com',
        password: 'Password123',
        name: 'Test User'
      };
      
      const existingUser = User.create(
        'existing@example.com',
        'hash',
        'Existing'
      );
      
      vi.mocked(mockUserRepository.findByEmail).mockResolvedValue(existingUser);
      
      // Act & Assert
      await expect(userService.createUser(dto)).rejects.toThrow(
        'Email already exists'
      );
      
      expect(mockPasswordService.hash).not.toHaveBeenCalled();
      expect(mockUserRepository.save).not.toHaveBeenCalled();
    });
  });
  
  describe('findById', () => {
    it('should return user when found', async () => {
      // Arrange
      const user = User.create('test@example.com', 'hash', 'Test');
      vi.mocked(mockUserRepository.findById).mockResolvedValue(user);
      
      // Act
      const result = await userService.findById('user-id');
      
      // Assert
      expect(result).toEqual(user);
      expect(mockUserRepository.findById).toHaveBeenCalledWith('user-id');
    });
    
    it('should throw NotFoundError when user not found', async () => {
      // Arrange
      vi.mocked(mockUserRepository.findById).mockResolvedValue(null);
      
      // Act & Assert
      await expect(userService.findById('invalid-id')).rejects.toThrow(
        NotFoundError
      );
    });
  });
});

// domain/entities/order.entity.test.ts
import { describe, it, expect } from 'vitest';
import { Order, OrderStatus } from './order.entity';
import { OrderItem } from './order-item.entity';
import { Money } from '@/domain/value-objects/money.vo';

describe('Order', () => {
  describe('create', () => {
    it('should create order with items', () => {
      // Arrange
      const items = [
        OrderItem.create('product-1', Money.create(10, 'USD'), 2),
        OrderItem.create('product-2', Money.create(5, 'USD'), 1)
      ];
      
      // Act
      const order = Order.create('customer-1', items);
      
      // Assert
      expect(order.customerId).toBe('customer-1');
      expect(order.items).toHaveLength(2);
      expect(order.status).toBe(OrderStatus.PENDING);
      expect(order.total.amount).toBe(25); // 10*2 + 5*1
    });
    
    it('should throw error when creating order without items', () => {
      // Act & Assert
      expect(() => Order.create('customer-1', [])).toThrow(
        'Order must have at least one item'
      );
    });
  });
  
  describe('ship', () => {
    it('should ship confirmed order', () => {
      // Arrange
      const items = [OrderItem.create('p1', Money.create(10, 'USD'), 1)];
      const order = Order.create('customer-1', items);
      order.confirm();
      
      // Act
      order.ship();
      
      // Assert
      expect(order.status).toBe(OrderStatus.SHIPPED);
    });
    
    it('should throw error when shipping non-confirmed order', () => {
      // Arrange
      const items = [OrderItem.create('p1', Money.create(10, 'USD'), 1)];
      const order = Order.create('customer-1', items);
      
      // Act & Assert
      expect(() => order.ship()).toThrow('Can only ship confirmed orders');
    });
  });
  
  describe('cancel', () => {
    it('should cancel pending order', () => {
      // Arrange
      const items = [OrderItem.create('p1', Money.create(10, 'USD'), 1)];
      const order = Order.create('customer-1', items);
      
      // Act
      order.cancel();
      
      // Assert
      expect(order.status).toBe(OrderStatus.CANCELLED);
    });
    
    it('should not cancel delivered order', () => {
      // Arrange
      const items = [OrderItem.create('p1', Money.create(10, 'USD'), 1)];
      const order = Order.create('customer-1', items);
      order.confirm();
      order.ship();
      order.deliver();
      
      // Act & Assert
      expect(() => order.cancel()).toThrow('Cannot cancel delivered order');
    });
  });
});
```

## Integration Tests

```typescript
// tests/integration/users.api.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import request from 'supertest';
import { app } from '@/app';
import { prisma } from '@/infrastructure/database/prisma.client';

describe('Users API Integration Tests', () => {
  beforeAll(async () => {
    await prisma.$connect();
  });
  
  afterAll(async () => {
    await prisma.$disconnect();
  });
  
  beforeEach(async () => {
    await prisma.user.deleteMany();
  });
  
  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'Password123',
        name: 'Test User'
      };
      
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201);
      
      expect(response.body.data).toMatchObject({
        id: expect.any(String),
        email: 'test@example.com',
        name: 'Test User',
        role: 'user',
        active: true
      });
      expect(response.body.data.passwordHash).toBeUndefined();
      
      const dbUser = await prisma.user.findUnique({
        where: { email: 'test@example.com' }
      });
      expect(dbUser).toBeTruthy();
      expect(dbUser?.email).toBe('test@example.com');
    });
    
    it('should return 400 for invalid email', async () => {
      const userData = {
        email: 'invalid-email',
        password: 'Password123',
        name: 'Test User'
      };
      
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(400);
      
      expect(response.body.error).toMatchObject({
        code: 'VALIDATION_ERROR',
        fields: {
          email: expect.stringContaining('Invalid email')
        }
      });
    });
    
    it('should return 409 for duplicate email', async () => {
      const userData = {
        email: 'duplicate@example.com',
        password: 'Password123',
        name: 'Test User'
      };
      
      await request(app).post('/api/users').send(userData);
      
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(409);
      
      expect(response.body.error.message).toContain('already exists');
    });
  });
  
  describe('GET /api/users/:id', () => {
    it('should return user by id', async () => {
      const user = await prisma.user.create({
        data: {
          email: 'test@example.com',
          passwordHash: 'hash',
          name: 'Test User',
          role: 'user'
        }
      });
      
      const response = await request(app)
        .get(`/api/users/${user.id}`)
        .expect(200);
      
      expect(response.body.data).toMatchObject({
        id: user.id,
        email: 'test@example.com',
        name: 'Test User'
      });
    });
    
    it('should return 404 for non-existent user', async () => {
      const response = await request(app)
        .get('/api/users/non-existent-id')
        .expect(404);
      
      expect(response.body.error.code).toBe('NotFoundError');
    });
  });
});
```

## React Component Tests

```typescript
// features/users/components/UserCard.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import { UserCard } from './UserCard';

describe('UserCard', () => {
  const mockUser = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
    role: 'user' as const,
    active: true,
    createdAt: new Date(),
    updatedAt: new Date()
  };
  
  it('should render user information', () => {
    render(<UserCard user={mockUser} />);
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
    expect(screen.getByText('Active')).toBeInTheDocument();
    expect(screen.getByText('user')).toBeInTheDocument();
  });
  
  it('should call onClick when clicked', () => {
    const handleClick = vi.fn();
    render(<UserCard user={mockUser} onClick={handleClick} />);
    fireEvent.click(screen.getByText('John Doe'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  it('should show inactive status for inactive user', () => {
    const inactiveUser = { ...mockUser, active: false };
    render(<UserCard user={inactiveUser} />);
    
    expect(screen.getByText('Inactive')).toBeInTheDocument();
  });
});

// features/users/components/CreateUserForm.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { CreateUserForm } from './CreateUserForm';

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false }
    }
  });
  
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};

describe('CreateUserForm', () => {
  it('should render form fields', () => {
    render(<CreateUserForm />, { wrapper: createWrapper() });
    
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/name/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /create user/i })).toBeInTheDocument();
  });
  
  it('should show validation errors', async () => {
    const user = userEvent.setup();
    render(<CreateUserForm />, { wrapper: createWrapper() });
    
    await user.click(screen.getByRole('button', { name: /create user/i }));
    
    await waitFor(() => {
      expect(screen.getByText(/invalid email/i)).toBeInTheDocument();
      expect(screen.getByText(/password must be/i)).toBeInTheDocument();
      expect(screen.getByText(/name must be/i)).toBeInTheDocument();
    });
  });
  
  it('should submit form with valid data', async () => {
    const user = userEvent.setup();
    const onSuccess = vi.fn();
    
    render(<CreateUserForm onSuccess={onSuccess} />, { 
      wrapper: createWrapper() 
    });
    
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'Password123');
    await user.type(screen.getByLabelText(/name/i), 'Test User');
    await user.click(screen.getByRole('button', { name: /create user/i }));
    
    await waitFor(() => {
      expect(onSuccess).toHaveBeenCalled();
    });
  });
});
```

## E2E Tests with Playwright

```typescript
// tests/e2e/users.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Management', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });
  
  test('should create new user', async ({ page }) => {
    await page.click('text=Create User');
    
    await page.fill('input[name="email"]', 'newuser@example.com');
    await page.fill('input[name="password"]', 'Password123');
    await page.fill('input[name="name"]', 'New User');
    
    await page.click('button:has-text("Create User")');
    
    await expect(page.locator('text=User created successfully')).toBeVisible();
    
    await page.goto('/users');
    await expect(page.locator('text=New User')).toBeVisible();
    await expect(page.locator('text=newuser@example.com')).toBeVisible();
  });
  
  test('should show validation errors', async ({ page }) => {
    await page.click('text=Create User');
    
    await page.click('button:has-text("Create User")');
    
    await expect(page.locator('text=Invalid email')).toBeVisible();
    await expect(page.locator('text=Password must be')).toBeVisible();
  });
  
  test('should display user list with pagination', async ({ page }) => {
    await page.goto('/users');
    
    const userCards = page.locator('[data-testid="user-card"]');
    await expect(userCards).toHaveCount(20);
    
    await page.click('button:has-text("Next")');
    
    await expect(page).toHaveURL(/page=2/);
    
    await expect(userCards).toHaveCount(20);
  });
  
  test('should filter users by search', async ({ page }) => {
    await page.goto('/users');
    
    await page.fill('input[placeholder="Search users..."]', 'john');
    
    const userCards = page.locator('[data-testid="user-card"]');
    await expect(userCards).toHaveCount.toBeGreaterThan(0);
    
    const userNames = await userCards.locator('h3').allTextContents();
    userNames.forEach(name => {
      expect(name.toLowerCase()).toContain('john');
    });
  });
});
```
