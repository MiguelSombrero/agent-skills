Reference for software-architect skill. See [SKILL.md](../SKILL.md) for core instructions.
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

