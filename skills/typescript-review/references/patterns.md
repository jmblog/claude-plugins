# TypeScript Patterns and Best Practices

This reference provides detailed patterns, examples, and anti-patterns for TypeScript development.

## Table of Contents

1. [Type Safety Patterns](#type-safety-patterns)
2. [Generic Patterns](#generic-patterns)
3. [Async Patterns](#async-patterns)
4. [Error Handling Patterns](#error-handling-patterns)
5. [Design Patterns](#design-patterns)
6. [Frontend-Specific Patterns](#frontend-specific-patterns)
7. [Backend-Specific Patterns](#backend-specific-patterns)
8. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)

---

## Type Safety Patterns

### Discriminated Unions

**Pattern**: Use tagged unions for complex state management

```typescript
// ✅ Good: Discriminated union with type guards
type LoadingState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

function handleState<T>(state: LoadingState<T>) {
  switch (state.status) {
    case 'idle':
      return 'Not started';
    case 'loading':
      return 'Loading...';
    case 'success':
      return state.data; // TypeScript knows data exists
    case 'error':
      return state.error.message; // TypeScript knows error exists
  }
}

// ❌ Bad: Boolean flags
type BadLoadingState<T> = {
  isLoading: boolean;
  isError: boolean;
  data?: T;
  error?: Error;
};
// Problem: Can have invalid states like isLoading=true AND isError=true
```

### Type Guards

**Pattern**: Use type predicates for runtime type checking

```typescript
// ✅ Good: Type predicate function
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value &&
    typeof (value as User).id === 'string' &&
    typeof (value as User).name === 'string'
  );
}

// Usage
function processData(data: unknown) {
  if (isUser(data)) {
    console.log(data.name); // TypeScript knows data is User
  }
}

// ❌ Bad: Type assertion without validation
function processDataBad(data: unknown) {
  const user = data as User; // Unsafe!
  console.log(user.name); // Runtime error if data isn't a User
}
```

### Branded Types

**Pattern**: Create nominal types for type safety

```typescript
// ✅ Good: Branded types prevent mixing similar primitives
type UserId = string & { readonly __brand: 'UserId' };
type ProductId = string & { readonly __brand: 'ProductId' };

function createUserId(id: string): UserId {
  return id as UserId;
}

function createProductId(id: string): ProductId {
  return id as ProductId;
}

function getUser(id: UserId) { /* ... */ }
function getProduct(id: ProductId) { /* ... */ }

const userId = createUserId('user-123');
const productId = createProductId('product-456');

getUser(userId); // ✅ OK
getUser(productId); // ❌ Type error - prevents mixing IDs

// ❌ Bad: Plain strings can be mixed up
function getUserBad(id: string) { /* ... */ }
function getProductBad(id: string) { /* ... */ }

getUserBad(productId); // ✅ Compiles but semantically wrong
```

### Const Assertions

**Pattern**: Use `as const` for literal types

```typescript
// ✅ Good: Const assertion preserves literal types
const ROUTES = {
  HOME: '/',
  ABOUT: '/about',
  CONTACT: '/contact',
} as const;

type Route = typeof ROUTES[keyof typeof ROUTES]; // '/' | '/about' | '/contact'

// ✅ Good: Readonly arrays with literal types
const STATUSES = ['pending', 'approved', 'rejected'] as const;
type Status = typeof STATUSES[number]; // 'pending' | 'approved' | 'rejected'

// ❌ Bad: Mutable objects lose literal types
const ROUTES_BAD = {
  HOME: '/',
  ABOUT: '/about',
};
type RouteBad = typeof ROUTES_BAD[keyof typeof ROUTES_BAD]; // string (too broad)
```

---

## Generic Patterns

### Constrained Generics

**Pattern**: Use constraints to ensure type safety

```typescript
// ✅ Good: Generic with constraints
interface Entity {
  id: string;
  createdAt: Date;
}

function findById<T extends Entity>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}

// ✅ Good: Multiple constraints
interface Nameable {
  name: string;
}

function sortByName<T extends Nameable>(items: T[]): T[] {
  return [...items].sort((a, b) => a.name.localeCompare(b.name));
}

// ❌ Bad: Unconstrained generic
function findByIdBad<T>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id); // ❌ Property 'id' doesn't exist on T
}
```

### Generic Factories

**Pattern**: Type-safe factory functions

```typescript
// ✅ Good: Generic factory with builder pattern
class QueryBuilder<T> {
  private filters: Array<(item: T) => boolean> = [];

  where(predicate: (item: T) => boolean): this {
    this.filters.push(predicate);
    return this;
  }

  execute(items: T[]): T[] {
    return items.filter(item => this.filters.every(f => f(item)));
  }
}

// Usage with type inference
const users = new QueryBuilder<User>()
  .where(u => u.age > 18)
  .where(u => u.isActive)
  .execute(allUsers);
```

### Utility Type Patterns

**Pattern**: Leverage built-in utility types

```typescript
// ✅ Good: Using utility types
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
}

// Public user data (remove sensitive fields)
type PublicUser = Omit<User, 'password'>;

// User update data (all fields optional)
type UserUpdate = Partial<User>;

// Required user creation data (subset of fields)
type UserCreate = Pick<User, 'name' | 'email' | 'password'>;

// Read-only user
type ReadonlyUser = Readonly<User>;

// Record pattern for mappings
type UserMap = Record<string, User>;

// ❌ Bad: Manually redefining types
type PublicUserBad = {
  id: string;
  name: string;
  email: string;
}; // Duplicates User definition, harder to maintain
```

---

## Async Patterns

### Promise Typing

**Pattern**: Explicit Promise types and error handling

```typescript
// ✅ Good: Explicit Promise return type
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error(`Failed to fetch user: ${response.statusText}`);
  }
  return response.json();
}

// ✅ Good: Union type for success/error
async function fetchUserSafe(id: string): Promise<User | Error> {
  try {
    return await fetchUser(id);
  } catch (error) {
    return error instanceof Error ? error : new Error('Unknown error');
  }
}

// ✅ Good: Result type pattern
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

async function fetchUserResult(id: string): Promise<Result<User>> {
  try {
    const user = await fetchUser(id);
    return { success: true, data: user };
  } catch (error) {
    return {
      success: false,
      error: error instanceof Error ? error : new Error('Unknown error')
    };
  }
}

// ❌ Bad: Implicit Promise type
async function fetchUserBad(id: string) {
  return fetch(`/api/users/${id}`).then(r => r.json());
} // Return type is Promise<any>
```

### Parallel Operations

**Pattern**: Type-safe parallel async operations

```typescript
// ✅ Good: Typed Promise.all
async function fetchUserData(userId: string) {
  const [user, posts, comments] = await Promise.all([
    fetchUser(userId),
    fetchPosts(userId),
    fetchComments(userId),
  ]);

  // TypeScript knows the types of each element
  return { user, posts, comments };
}

// ✅ Good: Promise.allSettled with type discrimination
async function fetchMultipleUsers(ids: string[]) {
  const results = await Promise.allSettled(
    ids.map(id => fetchUser(id))
  );

  const users = results
    .filter((r): r is PromiseFulfilledResult<User> => r.status === 'fulfilled')
    .map(r => r.value);

  const errors = results
    .filter((r): r is PromiseRejectedResult => r.status === 'rejected')
    .map(r => r.reason);

  return { users, errors };
}
```

---

## Error Handling Patterns

### Typed Errors

**Pattern**: Custom error classes with type safety

```typescript
// ✅ Good: Custom error hierarchy
class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

class ValidationError extends AppError {
  constructor(
    message: string,
    public readonly fields: Record<string, string>
  ) {
    super(message, 'VALIDATION_ERROR', 400);
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super(`${resource} not found: ${id}`, 'NOT_FOUND', 404);
  }
}

// ✅ Good: Type-safe error handling
async function getUser(id: string): Promise<User> {
  const user = await db.findUser(id);
  if (!user) {
    throw new NotFoundError('User', id);
  }
  return user;
}

// Usage with type narrowing
try {
  const user = await getUser(id);
} catch (error) {
  if (error instanceof ValidationError) {
    console.log('Validation errors:', error.fields);
  } else if (error instanceof NotFoundError) {
    console.log('Not found:', error.message);
  } else if (error instanceof Error) {
    console.log('Generic error:', error.message);
  }
}
```

### Result Type Pattern

**Pattern**: Functional error handling without exceptions

```typescript
// ✅ Good: Result type for error handling
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

function divide(a: number, b: number): Result<number, string> {
  if (b === 0) {
    return { ok: false, error: 'Division by zero' };
  }
  return { ok: true, value: a / b };
}

// Helper functions
function ok<T>(value: T): Result<T, never> {
  return { ok: true, value };
}

function err<E>(error: E): Result<never, E> {
  return { ok: false, error };
}

// Usage with type narrowing
const result = divide(10, 2);
if (result.ok) {
  console.log(result.value); // TypeScript knows value exists
} else {
  console.error(result.error); // TypeScript knows error exists
}
```

---

## Design Patterns

### Repository Pattern

**Pattern**: Type-safe data access layer

```typescript
// ✅ Good: Generic repository interface
interface Repository<T extends { id: string }> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  create(data: Omit<T, 'id' | 'createdAt' | 'updatedAt'>): Promise<T>;
  update(id: string, data: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

// Implementation
class UserRepository implements Repository<User> {
  async findById(id: string): Promise<User | null> {
    // Implementation
  }

  async findAll(): Promise<User[]> {
    // Implementation
  }

  async create(data: Omit<User, 'id' | 'createdAt' | 'updatedAt'>): Promise<User> {
    // Implementation
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    // Implementation
  }

  async delete(id: string): Promise<void> {
    // Implementation
  }
}
```

### Builder Pattern

**Pattern**: Fluent interface with type safety

```typescript
// ✅ Good: Type-safe builder
class HttpRequestBuilder {
  private config: {
    url?: string;
    method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
    headers?: Record<string, string>;
    body?: unknown;
  } = {};

  url(url: string): this {
    this.config.url = url;
    return this;
  }

  method(method: 'GET' | 'POST' | 'PUT' | 'DELETE'): this {
    this.config.method = method;
    return this;
  }

  header(key: string, value: string): this {
    this.config.headers = { ...this.config.headers, [key]: value };
    return this;
  }

  body<T>(data: T): this {
    this.config.body = data;
    return this;
  }

  async send<T>(): Promise<T> {
    if (!this.config.url) {
      throw new Error('URL is required');
    }
    const response = await fetch(this.config.url, {
      method: this.config.method ?? 'GET',
      headers: this.config.headers,
      body: this.config.body ? JSON.stringify(this.config.body) : undefined,
    });
    return response.json();
  }
}

// Usage
const data = await new HttpRequestBuilder()
  .url('/api/users')
  .method('POST')
  .header('Content-Type', 'application/json')
  .body({ name: 'John' })
  .send<User>();
```

---

## Frontend-Specific Patterns

### React Component Props

**Pattern**: Type-safe React components

```typescript
// ✅ Good: Explicit prop types
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
  children: React.ReactNode;
}

function Button({ variant, size = 'md', disabled, onClick, children }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled} className={`btn-${variant} btn-${size}`}>
      {children}
    </button>
  );
}

// ✅ Good: Generic component props
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>{renderItem(item, index)}</li>
      ))}
    </ul>
  );
}
```

### Form Handling

**Pattern**: Type-safe form state and validation

```typescript
// ✅ Good: Type-safe form handling
interface FormData {
  email: string;
  password: string;
  rememberMe: boolean;
}

type FormErrors = Partial<Record<keyof FormData, string>>;

function useForm<T extends Record<string, any>>(initialValues: T) {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});

  const handleChange = (field: keyof T) => (
    event: React.ChangeEvent<HTMLInputElement>
  ) => {
    setValues(prev => ({
      ...prev,
      [field]: event.target.type === 'checkbox' ? event.target.checked : event.target.value,
    }));
  };

  return { values, errors, setErrors, handleChange };
}

// Usage
const { values, errors, handleChange } = useForm<FormData>({
  email: '',
  password: '',
  rememberMe: false,
});
```

---

## Backend-Specific Patterns

### Express Request/Response Typing

**Pattern**: Type-safe Express handlers

```typescript
// ✅ Good: Typed request handlers
import { Request, Response, NextFunction } from 'express';

interface TypedRequest<
  TParams = {},
  TQuery = {},
  TBody = {}
> extends Request {
  params: TParams;
  query: TQuery;
  body: TBody;
}

// Route parameter types
interface UserParams {
  userId: string;
}

interface UserQuery {
  include?: string;
}

interface CreateUserBody {
  name: string;
  email: string;
}

// Handler with full typing
async function getUser(
  req: TypedRequest<UserParams, UserQuery>,
  res: Response
) {
  const { userId } = req.params; // Typed!
  const { include } = req.query; // Typed!

  const user = await userService.findById(userId);
  res.json(user);
}

async function createUser(
  req: TypedRequest<{}, {}, CreateUserBody>,
  res: Response
) {
  const { name, email } = req.body; // Typed!

  const user = await userService.create({ name, email });
  res.status(201).json(user);
}
```

### Middleware Typing

**Pattern**: Type-safe middleware with custom Request properties

```typescript
// ✅ Good: Extend Request type for auth middleware
declare global {
  namespace Express {
    interface Request {
      user?: User;
    }
  }
}

const authMiddleware = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const token = req.headers.authorization?.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  try {
    const user = await verifyToken(token);
    req.user = user; // Now TypeScript knows about req.user
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
};

// Usage in routes
app.get('/profile', authMiddleware, (req, res) => {
  // TypeScript knows req.user exists (but might be undefined)
  if (!req.user) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  res.json(req.user);
});
```

---

## Anti-Patterns to Avoid

### Anti-Pattern: Excessive any

```typescript
// ❌ Bad: any disables type checking
function processData(data: any) {
  return data.foo.bar.baz; // No type safety
}

// ✅ Good: Use unknown and type guards
function processData(data: unknown) {
  if (isValidData(data)) {
    return data.foo.bar.baz; // Type-safe after validation
  }
  throw new Error('Invalid data');
}
```

### Anti-Pattern: Type Assertions Over Validation

```typescript
// ❌ Bad: Unsafe type assertion
const user = JSON.parse(jsonString) as User;

// ✅ Good: Runtime validation
function parseUser(jsonString: string): User {
  const data = JSON.parse(jsonString);
  if (!isUser(data)) {
    throw new Error('Invalid user data');
  }
  return data;
}
```

### Anti-Pattern: Ignoring null/undefined

```typescript
// ❌ Bad: Non-null assertion without check
function getName(user: User | null) {
  return user!.name; // Runtime error if user is null
}

// ✅ Good: Proper null checking
function getName(user: User | null): string | null {
  return user?.name ?? null;
}
```

### Anti-Pattern: Mutable Default Parameters

```typescript
// ❌ Bad: Mutable default object
function createConfig(options: Partial<Config> = {}) {
  const defaults = { timeout: 1000, retries: 3 };
  Object.assign(defaults, options); // Mutates defaults
  return defaults;
}

// ✅ Good: Immutable defaults
function createConfig(options: Partial<Config> = {}): Config {
  return {
    timeout: 1000,
    retries: 3,
    ...options,
  };
}
```

### Anti-Pattern: Deep Type Nesting

```typescript
// ❌ Bad: Deeply nested type (slow compilation)
type DeepNested = {
  level1: {
    level2: {
      level3: {
        level4: {
          level5: string;
        };
      };
    };
  };
};

// ✅ Good: Flatter structure with type aliases
type Level4 = { level5: string };
type Level3 = { level4: Level4 };
type Level2 = { level3: Level3 };
type Level1 = { level2: Level2 };
type BetterNested = { level1: Level1 };
```
