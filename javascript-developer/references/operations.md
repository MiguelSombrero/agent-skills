# Performance, Anti-Patterns, and Checklist

Reference for javascript-developer skill. See [SKILL.md](../SKILL.md) for core instructions.

---

## Performance Optimization

### Caching

```typescript
// Implement simple in-memory cache
class CacheService {
  private cache = new Map<string, { value: any; expiry: number }>();
  
  set(key: string, value: any, ttlSeconds: number): void {
    this.cache.set(key, {
      value,
      expiry: Date.now() + ttlSeconds * 1000
    });
  }
  
  get<T>(key: string): T | null {
    const item = this.cache.get(key);
    
    if (!item) return null;
    
    if (Date.now() > item.expiry) {
      this.cache.delete(key);
      return null;
    }
    
    return item.value as T;
  }
  
  delete(key: string): void {
    this.cache.delete(key);
  }
  
  clear(): void {
    this.cache.clear();
  }
}

export const cacheService = new CacheService();

// Use in service
class UserService {
  async findById(id: string): Promise<User> {
    const cached = cacheService.get<User>(`user:${id}`);
    if (cached) return cached;
    
    const user = await this.userRepository.findById(id);
    
    if (!user) {
      throw new NotFoundError('User', id);
    }
    
    cacheService.set(`user:${id}`, user, 300);
    
    return user;
  }
  
  async updateUser(id: string, updates: UpdateUserDTO): Promise<User> {
    const user = await this.userRepository.update(id, updates);
    cacheService.delete(`user:${id}`);
    return user;
  }
}
```

### Database Query Optimization

```typescript
// ❌ Bad - N+1 query problem
async function getOrdersWithCustomers() {
  const orders = await prisma.order.findMany();
  
  for (const order of orders) {
    order.customer = await prisma.user.findUnique({
      where: { id: order.customerId }
    });
  }
  
  return orders;
}

// ✅ Good - Include relation
async function getOrdersWithCustomers() {
  return prisma.order.findMany({
    include: {
      customer: true,
      items: {
        include: {
          product: true
        }
      }
    }
  });
}

// ✅ Good - Select specific fields
async function getOrderSummaries() {
  return prisma.order.findMany({
    select: {
      id: true,
      total: true,
      createdAt: true,
      customer: {
        select: {
          id: true,
          name: true,
          email: true
        }
      }
    }
  });
}
```

---

## Anti-Patterns to Avoid

### Callback Hell

```typescript
// ❌ Bad - Callback hell
function processOrder(orderId, callback) {
  getOrder(orderId, (err, order) => {
    if (err) return callback(err);
    
    validateOrder(order, (err, valid) => {
      if (err) return callback(err);
      if (!valid) return callback(new Error('Invalid'));
      
      processPayment(order, (err, payment) => {
        if (err) return callback(err);
        
        updateInventory(order, (err) => {
          if (err) return callback(err);
          
          sendConfirmation(order, (err) => {
            if (err) return callback(err);
            callback(null, order);
          });
        });
      });
    });
  });
}

// ✅ Good - Async/await
async function processOrder(orderId: string): Promise<Order> {
  const order = await getOrder(orderId);
  const valid = await validateOrder(order);
  
  if (!valid) {
    throw new ValidationError('Invalid order');
  }
  
  await processPayment(order);
  await updateInventory(order);
  await sendConfirmation(order);
  
  return order;
}
```

### Prop Drilling

```typescript
// ❌ Bad - Prop drilling through many components
function App() {
  const [user, setUser] = useState(null);
  
  return <Dashboard user={user} setUser={setUser} />;
}

function Dashboard({ user, setUser }) {
  return <Sidebar user={user} setUser={setUser} />;
}

function Sidebar({ user, setUser }) {
  return <UserMenu user={user} setUser={setUser} />;
}

function UserMenu({ user, setUser }) {
  return <UserProfile user={user} setUser={setUser} />;
}

// ✅ Good - Context API
const UserContext = createContext<UserContextType | null>(null);

export function UserProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
}

export function useUser() {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within UserProvider');
  }
  return context;
}

// Usage
function UserProfile() {
  const { user, setUser } = useUser();
  // No prop drilling!
}
```

---

## Quick Reference Checklist

**Architecture:**
- [ ] Clear layered architecture (presentation, application, domain, infrastructure)
- [ ] Domain logic in entities, not services
- [ ] Dependencies point inward (DIP)
- [ ] Each module has single responsibility
- [ ] Interfaces for abstraction

**API:**
- [ ] Proper HTTP methods and status codes
- [ ] DTO validation (Zod/class-validator)
- [ ] Global error handling
- [ ] Pagination for lists
- [ ] Request/response types defined

**Frontend:**
- [ ] Component composition
- [ ] Custom hooks for logic
- [ ] Context for shared state
- [ ] Form validation
- [ ] Loading and error states

**Testing:**
- [ ] Lots of unit tests (70%+)
- [ ] Some integration tests (20%)
- [ ] Few E2E tests (10%)
- [ ] Tests are independent
- [ ] TDD approach

**TypeScript:**
- [ ] Strict mode enabled
- [ ] No `any` types
- [ ] Proper type definitions
- [ ] Type-safe API calls

**Performance:**
- [ ] Database queries optimized
- [ ] Caching where appropriate
- [ ] Code splitting
- [ ] Lazy loading

**Code Quality:**
- [ ] Descriptive names
- [ ] Short, focused functions
- [ ] No magic numbers/strings
- [ ] Async/await over callbacks
- [ ] Error handling
