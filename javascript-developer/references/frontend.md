# React, Hooks, Forms, and Next.js

Reference for javascript-developer skill. See [SKILL.md](../SKILL.md) for core instructions.

---

## React Frontend

### Component Architecture

```typescript
// features/users/components/UserList.tsx
import { useState, useEffect } from 'react';
import { UserCard } from './UserCard';
import { Pagination } from '@/shared/components/Pagination';
import { LoadingSpinner } from '@/shared/components/LoadingSpinner';
import { ErrorMessage } from '@/shared/components/ErrorMessage';
import { useUsers } from '../hooks/useUsers';

interface UserListProps {
  onUserSelect?: (userId: string) => void;
}

export function UserList({ onUserSelect }: UserListProps) {
  const [page, setPage] = useState(1);
  const { data, isLoading, error } = useUsers({ page, limit: 20 });
  
  if (isLoading) {
    return <LoadingSpinner />;
  }
  
  if (error) {
    return <ErrorMessage message={error.message} />;
  }
  
  if (!data || data.items.length === 0) {
    return (
      <div className="text-center py-8 text-gray-500">
        No users found
      </div>
    );
  }
  
  return (
    <div className="space-y-4">
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        {data.items.map(user => (
          <UserCard
            key={user.id}
            user={user}
            onClick={() => onUserSelect?.(user.id)}
          />
        ))}
      </div>
      
      <Pagination
        currentPage={page}
        totalPages={data.pagination.totalPages}
        onPageChange={setPage}
      />
    </div>
  );
}

// features/users/components/UserCard.tsx
import { User } from '../types';

interface UserCardProps {
  user: User;
  onClick?: () => void;
}

export function UserCard({ user, onClick }: UserCardProps) {
  return (
    <div
      className="border rounded-lg p-4 hover:shadow-md transition-shadow cursor-pointer"
      onClick={onClick}
    >
      <div className="flex items-center gap-3">
        <div className="w-12 h-12 rounded-full bg-blue-500 flex items-center justify-center text-white font-semibold">
          {user.name.charAt(0).toUpperCase()}
        </div>
        <div className="flex-1">
          <h3 className="font-semibold text-lg">{user.name}</h3>
          <p className="text-sm text-gray-600">{user.email}</p>
        </div>
      </div>
      
      <div className="mt-3 flex items-center justify-between">
        <span className={`text-xs px-2 py-1 rounded ${
          user.active 
            ? 'bg-green-100 text-green-800' 
            : 'bg-gray-100 text-gray-800'
        }`}>
          {user.active ? 'Active' : 'Inactive'}
        </span>
        <span className="text-xs text-gray-500">{user.role}</span>
      </div>
    </div>
  );
}
```

### Custom Hooks

```typescript
// features/users/hooks/useUsers.ts
import { useQuery } from '@tanstack/react-query';
import { userService } from '../services/user.service';

interface UseUsersOptions {
  page?: number;
  limit?: number;
}

export function useUsers(options: UseUsersOptions = {}) {
  const { page = 1, limit = 20 } = options;
  
  return useQuery({
    queryKey: ['users', page, limit],
    queryFn: () => userService.getAll({ page, limit }),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
}

export function useUser(id: string) {
  return useQuery({
    queryKey: ['users', id],
    queryFn: () => userService.getById(id),
    enabled: !!id,
  });
}

// features/users/hooks/useCreateUser.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { userService } from '../services/user.service';
import { CreateUserDTO } from '../types';

export function useCreateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (dto: CreateUserDTO) => userService.create(dto),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

// features/users/services/user.service.ts
import { apiClient } from '@/shared/api/client';
import { User, CreateUserDTO } from '../types';

class UserService {
  private baseUrl = '/api/users';
  
  async getAll(options: { page: number; limit: number }) {
    const { data } = await apiClient.get(this.baseUrl, {
      params: options
    });
    return data;
  }
  
  async getById(id: string): Promise<User> {
    const { data } = await apiClient.get(`${this.baseUrl}/${id}`);
    return data.data;
  }
  
  async create(dto: CreateUserDTO): Promise<User> {
    const { data } = await apiClient.post(this.baseUrl, dto);
    return data.data;
  }
  
  async update(id: string, updates: Partial<User>): Promise<User> {
    const { data } = await apiClient.put(`${this.baseUrl}/${id}`, updates);
    return data.data;
  }
  
  async delete(id: string): Promise<void> {
    await apiClient.delete(`${this.baseUrl}/${id}`);
  }
}

export const userService = new UserService();
```

### Form Handling

```typescript
// features/users/components/CreateUserForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useCreateUser } from '../hooks/useCreateUser';
import { Button } from '@/shared/components/Button';
import { Input } from '@/shared/components/Input';

const createUserSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  name: z.string().min(2, 'Name must be at least 2 characters'),
  role: z.enum(['user', 'admin']).default('user')
});

type CreateUserForm = z.infer<typeof createUserSchema>;

interface CreateUserFormProps {
  onSuccess?: () => void;
}

export function CreateUserForm({ onSuccess }: CreateUserFormProps) {
  const createUser = useCreateUser();
  
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting }
  } = useForm<CreateUserForm>({
    resolver: zodResolver(createUserSchema)
  });
  
  const onSubmit = async (data: CreateUserForm) => {
    try {
      await createUser.mutateAsync(data);
      onSuccess?.();
    } catch (error) {
      console.error('Failed to create user:', error);
    }
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <Input
        label="Email"
        type="email"
        {...register('email')}
        error={errors.email?.message}
      />
      
      <Input
        label="Password"
        type="password"
        {...register('password')}
        error={errors.password?.message}
      />
      
      <Input
        label="Name"
        {...register('name')}
        error={errors.name?.message}
      />
      
      <div>
        <label className="block text-sm font-medium mb-1">Role</label>
        <select
          {...register('role')}
          className="w-full border rounded px-3 py-2"
        >
          <option value="user">User</option>
          <option value="admin">Admin</option>
        </select>
      </div>
      
      {createUser.isError && (
        <div className="text-red-600 text-sm">
          {createUser.error.message}
        </div>
      )}
      
      <Button
        type="submit"
        disabled={isSubmitting || createUser.isPending}
        className="w-full"
      >
        {createUser.isPending ? 'Creating...' : 'Create User'}
      </Button>
    </form>
  );
}
```

---

## Next.js Adaptation

### API Routes (App Router)

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';
import { userService } from '@/services/user.service';

const createUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().min(2)
});

export async function GET(request: NextRequest) {
  try {
    const searchParams = request.nextUrl.searchParams;
    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '20');
    
    const result = await userService.findAll({ page, limit });
    
    return NextResponse.json({
      data: result.items,
      pagination: {
        page: result.page,
        limit: result.limit,
        total: result.total,
        totalPages: Math.ceil(result.total / result.limit)
      }
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const dto = createUserSchema.parse(body);
    
    const user = await userService.createUser(dto);
    
    return NextResponse.json(
      { data: user },
      { status: 201 }
    );
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation error', details: error.errors },
        { status: 400 }
      );
    }
    
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

// app/api/users/[id]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const user = await userService.findById(params.id);
    return NextResponse.json({ data: user });
  } catch (error) {
    if (error instanceof NotFoundError) {
      return NextResponse.json(
        { error: 'User not found' },
        { status: 404 }
      );
    }
    throw error;
  }
}
```

### Server Components

```typescript
// app/users/page.tsx (Server Component)
import { userService } from '@/services/user.service';
import { UserList } from '@/components/UserList';

interface PageProps {
  searchParams: {
    page?: string;
  };
}

export default async function UsersPage({ searchParams }: PageProps) {
  const page = parseInt(searchParams.page || '1');
  const users = await userService.findAll({ page, limit: 20 });
  
  return (
    <div className="container mx-auto py-8">
      <h1 className="text-3xl font-bold mb-6">Users</h1>
      <UserList initialData={users} />
    </div>
  );
}

// components/UserList.tsx (Client Component)
'use client';

import { useState } from 'react';
import { UserCard } from './UserCard';
import { Pagination } from './Pagination';

interface UserListProps {
  initialData: PaginatedResult<User>;
}

export function UserList({ initialData }: UserListProps) {
  const [data, setData] = useState(initialData);
  const [isLoading, setIsLoading] = useState(false);
  
  const loadPage = async (page: number) => {
    setIsLoading(true);
    const response = await fetch(`/api/users?page=${page}`);
    const newData = await response.json();
    setData(newData);
    setIsLoading(false);
  };
  
  return (
    <div>
      {isLoading ? (
        <div>Loading...</div>
      ) : (
        <>
          <div className="grid grid-cols-3 gap-4">
            {data.data.map(user => (
              <UserCard key={user.id} user={user} />
            ))}
          </div>
          
          <Pagination
            currentPage={data.pagination.page}
            totalPages={data.pagination.totalPages}
            onPageChange={loadPage}
          />
        </>
      )}
    </div>
  );
}
```
