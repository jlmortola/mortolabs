# TypeScript Conventions

## Naming

| Category | Convention | Example |
|----------|------------|---------|
| Components | PascalCase | `UserProfile` |
| Types/Interfaces | PascalCase | `ApiResponse`, `UserConfig` |
| Enums | PascalCase | `OrderStatus` |
| Variables | camelCase | `userName`, `isLoading` |
| Functions | camelCase | `fetchUserData`, `handleClick` |
| Constants | SCREAMING_SNAKE | `API_BASE_URL`, `MAX_RETRIES` |
| Files | kebab-case | `user-profile.tsx`, `api-client.ts` |

## Type Definitions

### Type vs Interface
```typescript
// Use type for simple objects, unions, intersections
type UserConfig = {
  readonly id: string;
  name: string;
  email?: string;
};

type Status = 'pending' | 'active' | 'archived';

type WithTimestamps<T> = T & {
  createdAt: Date;
  updatedAt: Date;
};

// Use interface for extendable contracts
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  save(entity: T): Promise<void>;
}
```

### Branded Types for IDs
```typescript
type UserId = string & { readonly __brand: unique symbol };
type OrderId = string & { readonly __brand: unique symbol };

// Prevents mixing IDs
function getUser(id: UserId): Promise<User>;
function getOrder(id: OrderId): Promise<Order>;

// Create branded values
const userId = '123' as UserId;
```

### Utility Types
```typescript
// Pick specific properties
type UserSummary = Pick<User, 'id' | 'name' | 'email'>;

// Omit properties
type CreateUserInput = Omit<User, 'id' | 'createdAt' | 'updatedAt'>;

// Make all optional
type UserUpdates = Partial<User>;

// Make all required
type CompleteUser = Required<User>;

// Extract from union
type ActiveStatus = Extract<Status, 'active' | 'pending'>;

// Exclude from union
type InactiveStatus = Exclude<Status, 'active'>;
```

### Function Types
```typescript
// Always specify return types for public functions
export function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

// Function type definitions
type EventHandler<T> = (event: T) => void;
type AsyncFn<T, R> = (input: T) => Promise<R>;

// Overloads for different signatures
function format(value: string): string;
function format(value: number, decimals?: number): string;
function format(value: string | number, decimals?: number): string {
  if (typeof value === 'string') return value.trim();
  return value.toFixed(decimals ?? 2);
}
```

## Type Guards

### typeof Guards
```typescript
function processValue(value: string | number) {
  if (typeof value === 'string') {
    return value.toUpperCase(); // value is string
  }
  return value * 2; // value is number
}
```

### Custom Type Guards
```typescript
type ApiError = { code: string; message: string };
type ApiSuccess<T> = { data: T };
type ApiResponse<T> = ApiError | ApiSuccess<T>;

function isApiError(response: ApiResponse<unknown>): response is ApiError {
  return 'code' in response;
}

// Usage
const response = await fetchData();
if (isApiError(response)) {
  console.error(response.code, response.message);
} else {
  console.log(response.data);
}
```

### Discriminated Unions
```typescript
type LoadingState = { status: 'loading' };
type SuccessState<T> = { status: 'success'; data: T };
type ErrorState = { status: 'error'; error: Error };
type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState;

function renderState<T>(state: AsyncState<T>) {
  switch (state.status) {
    case 'loading': return <Spinner />;
    case 'success': return <Data data={state.data} />;
    case 'error': return <Error error={state.error} />;
  }
}
```

## Result Pattern

```typescript
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

// Creating results
function ok<T>(data: T): Result<T> {
  return { success: true, data };
}

function err<E = Error>(error: E): Result<never, E> {
  return { success: false, error };
}

// Usage
async function safeParseJSON<T>(json: string): Promise<Result<T>> {
  try {
    return ok(JSON.parse(json));
  } catch (e) {
    return err(e instanceof Error ? e : new Error('Parse failed'));
  }
}

const result = await safeParseJSON<User>(data);
if (result.success) {
  console.log(result.data.name);
} else {
  console.error(result.error.message);
}
```

## Generics

### Generic Components
```typescript
type ListProps<T> = {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string;
};

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, i) => (
        <li key={keyExtractor(item)}>{renderItem(item, i)}</li>
      ))}
    </ul>
  );
}

// Usage with inference
<List 
  items={users} 
  renderItem={user => user.name} 
  keyExtractor={user => user.id} 
/>
```

### Generic Constraints
```typescript
// Must have id property
function findById<T extends { id: string }>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}

// Must be a key of object
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

## Zod Integration

```typescript
import { z } from 'zod';

// Define schema
const userSchema = z.object({
  name: z.string().min(1, 'Name required'),
  email: z.string().email('Invalid email'),
  age: z.number().int().positive().optional(),
  role: z.enum(['admin', 'user', 'guest']),
});

// Infer type from schema
type User = z.infer<typeof userSchema>;

// Validation
function validateUser(data: unknown): Result<User> {
  const result = userSchema.safeParse(data);
  if (result.success) {
    return ok(result.data);
  }
  return err(new Error(result.error.issues[0].message));
}
```

## Module Organization

```typescript
// Prefer explicit named exports
export { UserCard };
export { formatDate, formatCurrency };
export type { User, UserConfig };

// Group and re-export from index files
// components/ui/index.ts
export { Button } from './button';
export { Card } from './card';
export { Input } from './input';
export type { ButtonProps } from './button';

// Import order: external → internal → relative
import { useState, useCallback } from 'react';
import { json } from 'react-router';
import { z } from 'zod';

import { Button } from '~/components/ui';
import { useAuth } from '~/utils/hooks';

import { formatPrice } from './utils';
import type { ProductProps } from './types';
```

## Strict Mode Rules

```typescript
// tsconfig.json should include:
// "strict": true
// "noImplicitAny": true
// "strictNullChecks": true

// Always handle null/undefined
function getUser(id: string): User | null {
  return users.get(id) ?? null;
}

const user = getUser('123');
if (user) {
  console.log(user.name); // user is User, not null
}

// Non-null assertion (use sparingly)
const element = document.getElementById('app')!;

// Optional chaining
const name = user?.profile?.displayName ?? 'Anonymous';
```
