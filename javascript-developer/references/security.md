# JWT Authentication

Reference for javascript-developer skill. See [SKILL.md](../SKILL.md) for core instructions.

---

## JWT Implementation

```typescript
// application/services/auth.service.ts
import jwt from 'jsonwebtoken';
import { User } from '@/domain/entities/user.entity';
import { UserRepository } from '@/domain/repositories/user.repository';
import { PasswordService } from './password.service';
import { UnauthorizedError } from '@/shared/errors/api.error';

interface TokenPayload {
  userId: string;
  email: string;
  role: string;
}

export class AuthService {
  private readonly jwtSecret: string;
  private readonly jwtExpiresIn: string;
  
  constructor(
    private userRepository: UserRepository,
    private passwordService: PasswordService
  ) {
    this.jwtSecret = process.env.JWT_SECRET!;
    this.jwtExpiresIn = process.env.JWT_EXPIRES_IN || '7d';
  }
  
  async login(email: string, password: string): Promise<string> {
    const user = await this.userRepository.findByEmail(email);
    
    if (!user) {
      throw new UnauthorizedError('Invalid credentials');
    }
    
    const isPasswordValid = await this.passwordService.compare(
      password,
      user.passwordHash
    );
    
    if (!isPasswordValid) {
      throw new UnauthorizedError('Invalid credentials');
    }
    
    if (!user.active) {
      throw new UnauthorizedError('Account is deactivated');
    }
    
    return this.generateToken(user);
  }
  
  generateToken(user: User): string {
    const payload: TokenPayload = {
      userId: user.id,
      email: user.email,
      role: user.role
    };
    
    return jwt.sign(payload, this.jwtSecret, {
      expiresIn: this.jwtExpiresIn
    });
  }
  
  verifyToken(token: string): TokenPayload {
    try {
      return jwt.verify(token, this.jwtSecret) as TokenPayload;
    } catch (error) {
      throw new UnauthorizedError('Invalid token');
    }
  }
}

// presentation/middleware/auth.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { AuthService } from '@/application/services/auth.service';
import { UnauthorizedError } from '@/shared/errors/api.error';

export interface AuthenticatedRequest extends Request {
  user?: {
    userId: string;
    email: string;
    role: string;
  };
}

export function authenticate(authService: AuthService) {
  return (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    const authHeader = req.headers.authorization;
    
    if (!authHeader?.startsWith('Bearer ')) {
      throw new UnauthorizedError('Missing or invalid authorization header');
    }
    
    const token = authHeader.substring(7);
    
    try {
      const payload = authService.verifyToken(token);
      req.user = payload;
      next();
    } catch (error) {
      next(new UnauthorizedError('Invalid token'));
    }
  };
}

export function authorize(...roles: string[]) {
  return (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    if (!req.user) {
      throw new UnauthorizedError('Not authenticated');
    }
    
    if (!roles.includes(req.user.role)) {
      throw new UnauthorizedError('Insufficient permissions');
    }
    
    next();
  };
}
```
