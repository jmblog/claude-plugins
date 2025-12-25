# Architecture Patterns and Examples

This reference provides detailed architectural patterns, real-world examples, and trade-offs.

## Table of Contents

1. [Layered Architecture](#layered-architecture)
2. [Microservices Patterns](#microservices-patterns)
3. [Event-Driven Patterns](#event-driven-patterns)
4. [Data Patterns](#data-patterns)
5. [Caching Patterns](#caching-patterns)
6. [Security Patterns](#security-patterns)
7. [Scalability Patterns](#scalability-patterns)
8. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)

---

## Layered Architecture

### Classic Three-Tier Architecture

**Pattern**: Separate presentation, business logic, and data access

```
┌─────────────────────────┐
│   Presentation Layer    │  (Controllers, Views, API)
├─────────────────────────┤
│   Business Logic Layer  │  (Services, Domain Logic)
├─────────────────────────┤
│   Data Access Layer     │  (Repositories, ORM)
└─────────────────────────┘
```

**Example Implementation:**

```typescript
// Presentation Layer (Controller)
@Controller('/users')
export class UserController {
  constructor(private userService: UserService) {}

  @Get('/:id')
  async getUser(@Param('id') id: string) {
    return this.userService.getUser(id)
  }

  @Post()
  async createUser(@Body() data: CreateUserDto) {
    return this.userService.createUser(data)
  }
}

// Business Logic Layer (Service)
export class UserService {
  constructor(private userRepository: UserRepository) {}

  async getUser(id: string): Promise<User> {
    const user = await this.userRepository.findById(id)
    if (!user) {
      throw new NotFoundException('User not found')
    }
    return user
  }

  async createUser(data: CreateUserData): Promise<User> {
    // Business logic: validate, apply rules
    if (await this.userRepository.findByEmail(data.email)) {
      throw new ConflictException('Email already exists')
    }
    return this.userRepository.create(data)
  }
}

// Data Access Layer (Repository)
export class UserRepository {
  constructor(private db: Database) {}

  async findById(id: string): Promise<User | null> {
    return this.db.users.findOne({ id })
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.db.users.findOne({ email })
  }

  async create(data: CreateUserData): Promise<User> {
    return this.db.users.create(data)
  }
}
```

**Pros:**
- Clear separation of concerns
- Easy to understand and maintain
- Testable layers
- Can replace layers independently

**Cons:**
- Can become rigid
- May lead to anemic domain models
- Layer violations can occur

**When to use:**
- Most web applications
- When team needs clear structure
- CRUD-heavy applications

---

## Microservices Patterns

### Service per Bounded Context

**Pattern**: Each microservice owns a bounded context from domain

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   User       │  │   Order      │  │   Payment    │
│   Service    │  │   Service    │  │   Service    │
├──────────────┤  ├──────────────┤  ├──────────────┤
│   User DB    │  │   Order DB   │  │   Payment DB │
└──────────────┘  └──────────────┘  └──────────────┘
```

**Example:**

```typescript
// User Service
export class UserService {
  async createUser(data: CreateUserData): Promise<User> {
    // Owns user data and logic
    const user = await this.repository.create(data)

    // Publishes event for other services
    await this.eventBus.publish('user.created', {
      userId: user.id,
      email: user.email
    })

    return user
  }
}

// Order Service (subscribes to user events)
export class OrderService {
  @Subscribe('user.created')
  async onUserCreated(event: UserCreatedEvent) {
    // Create user profile in order service
    await this.userProfileRepository.create({
      userId: event.userId,
      email: event.email
    })
  }

  async createOrder(userId: string, items: OrderItem[]) {
    // Owns order logic
    const order = await this.repository.create({ userId, items })

    // Call payment service
    await this.paymentService.processPayment(order.id, order.total)

    return order
  }
}
```

**Pros:**
- Independent deployment
- Technology flexibility
- Team autonomy
- Fault isolation

**Cons:**
- Distributed system complexity
- Data consistency challenges
- Network latency
- Operational overhead

**When to use:**
- Large teams (multiple teams)
- Need independent scaling
- Different technology requirements
- Clear domain boundaries

### API Gateway Pattern

**Pattern**: Single entry point for all client requests

```
                ┌─────────────┐
    Client ──→  │ API Gateway │
                └──────┬──────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
    ┌───▼───┐      ┌───▼───┐      ┌──▼────┐
    │ User  │      │ Order │      │Payment│
    │Service│      │Service│      │Service│
    └───────┘      └───────┘      └───────┘
```

**Example:**

```typescript
// API Gateway
export class ApiGateway {
  constructor(
    private userService: UserServiceClient,
    private orderService: OrderServiceClient,
    private authService: AuthService
  ) {}

  @Get('/api/users/:id')
  async getUser(@Param('id') id: string, @Headers() headers: Headers) {
    // Authentication
    const user = await this.authService.verify(headers.authorization)

    // Route to service
    return this.userService.getUser(id)
  }

  @Get('/api/users/:id/orders')
  async getUserOrders(@Param('id') id: string) {
    // Aggregation: combine data from multiple services
    const [user, orders] = await Promise.all([
      this.userService.getUser(id),
      this.orderService.getOrdersByUser(id)
    ])

    return {
      user,
      orders
    }
  }

  @Post('/api/orders')
  @RateLimit({ max: 100, window: '1m' })
  async createOrder(@Body() data: CreateOrderDto) {
    // Rate limiting, validation
    return this.orderService.createOrder(data)
  }
}
```

**Pros:**
- Single entry point
- Centralized auth/rate limiting
- Request aggregation
- Protocol translation

**Cons:**
- Single point of failure
- Can become bottleneck
- Added complexity

**When to use:**
- Microservices architecture
- Need centralized cross-cutting concerns
- Multiple client types

### Circuit Breaker Pattern

**Pattern**: Prevent cascading failures

```typescript
export class CircuitBreaker {
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED'
  private failureCount = 0
  private readonly threshold = 5
  private readonly timeout = 60000 // 1 minute

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() - this.openedAt > this.timeout) {
        this.state = 'HALF_OPEN'
      } else {
        throw new Error('Circuit breaker is OPEN')
      }
    }

    try {
      const result = await fn()
      this.onSuccess()
      return result
    } catch (error) {
      this.onFailure()
      throw error
    }
  }

  private onSuccess() {
    this.failureCount = 0
    this.state = 'CLOSED'
  }

  private onFailure() {
    this.failureCount++
    if (this.failureCount >= this.threshold) {
      this.state = 'OPEN'
      this.openedAt = Date.now()
    }
  }
}

// Usage
const circuitBreaker = new CircuitBreaker()

async function callExternalService() {
  return circuitBreaker.execute(async () => {
    return axios.get('https://external-api.com/data')
  })
}
```

**Pros:**
- Prevents cascading failures
- Fast failure detection
- System resilience

**Cons:**
- Added complexity
- Configuration tuning needed

**When to use:**
- Calling external services
- Microservices communication
- Unreliable dependencies

---

## Event-Driven Patterns

### Event Sourcing

**Pattern**: Store events instead of current state

```typescript
// Event definitions
type UserEvent =
  | { type: 'USER_CREATED'; data: { id: string; email: string } }
  | { type: 'EMAIL_CHANGED'; data: { newEmail: string } }
  | { type: 'USER_DELETED'; data: { deletedAt: Date } }

// Event store
export class EventStore {
  async append(streamId: string, event: UserEvent) {
    await this.db.events.insert({
      streamId,
      type: event.type,
      data: event.data,
      timestamp: new Date(),
      version: await this.getNextVersion(streamId)
    })
  }

  async getEvents(streamId: string): Promise<UserEvent[]> {
    const records = await this.db.events.find({ streamId }).sort({ version: 1 })
    return records.map(r => ({ type: r.type, data: r.data }))
  }
}

// Aggregate that replays events
export class UserAggregate {
  private id: string
  private email: string
  private deleted: boolean = false

  static async load(eventStore: EventStore, id: string): Promise<UserAggregate> {
    const user = new UserAggregate()
    const events = await eventStore.getEvents(id)

    for (const event of events) {
      user.apply(event)
    }

    return user
  }

  private apply(event: UserEvent) {
    switch (event.type) {
      case 'USER_CREATED':
        this.id = event.data.id
        this.email = event.data.email
        break
      case 'EMAIL_CHANGED':
        this.email = event.data.newEmail
        break
      case 'USER_DELETED':
        this.deleted = true
        break
    }
  }

  async changeEmail(eventStore: EventStore, newEmail: string) {
    if (this.deleted) {
      throw new Error('Cannot change email of deleted user')
    }

    const event: UserEvent = {
      type: 'EMAIL_CHANGED',
      data: { newEmail }
    }

    this.apply(event)
    await eventStore.append(this.id, event)
  }
}
```

**Pros:**
- Complete audit trail
- Can replay events
- Time travel debugging
- Event-driven architecture

**Cons:**
- Complexity
- Storage requirements
- Query complexity

**When to use:**
- Need full audit trail
- Complex domain logic
- Event-driven systems

### CQRS (Command Query Responsibility Segregation)

**Pattern**: Separate read and write models

```typescript
// Write Model (Commands)
export class UserCommandService {
  constructor(
    private eventStore: EventStore,
    private eventBus: EventBus
  ) {}

  async createUser(command: CreateUserCommand): Promise<string> {
    const userId = generateId()

    const event: UserCreatedEvent = {
      type: 'USER_CREATED',
      data: {
        id: userId,
        email: command.email,
        name: command.name
      }
    }

    await this.eventStore.append(userId, event)
    await this.eventBus.publish('user.created', event)

    return userId
  }
}

// Read Model (Queries)
export class UserQueryService {
  constructor(private readDb: Database) {}

  async getUserById(id: string): Promise<UserReadModel | null> {
    // Optimized read model
    return this.readDb.users.findOne({ id })
  }

  async searchUsers(query: string): Promise<UserReadModel[]> {
    // Can use different storage (Elasticsearch)
    return this.readDb.users.search(query)
  }
}

// Projection (updates read model from events)
export class UserProjection {
  @Subscribe('user.created')
  async onUserCreated(event: UserCreatedEvent) {
    await this.readDb.users.insert({
      id: event.data.id,
      email: event.data.email,
      name: event.data.name,
      createdAt: new Date()
    })
  }

  @Subscribe('email.changed')
  async onEmailChanged(event: EmailChangedEvent) {
    await this.readDb.users.update(
      { id: event.userId },
      { email: event.data.newEmail }
    )
  }
}
```

**Pros:**
- Optimized read and write paths
- Scalability
- Flexible read models
- Can use different databases

**Cons:**
- Eventual consistency
- Complexity
- Data duplication

**When to use:**
- Complex read requirements
- Different scaling needs for reads/writes
- Event sourcing

---

## Data Patterns

### Repository Pattern

**Pattern**: Abstract data access behind repository interface

```typescript
// Repository interface
export interface IUserRepository {
  findById(id: string): Promise<User | null>
  findByEmail(email: string): Promise<User | null>
  create(data: CreateUserData): Promise<User>
  update(id: string, data: Partial<User>): Promise<User>
  delete(id: string): Promise<void>
}

// Implementation with ORM
export class UserRepository implements IUserRepository {
  constructor(private db: Database) {}

  async findById(id: string): Promise<User | null> {
    return this.db.users.findOne({ id })
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.db.users.findOne({ email })
  }

  async create(data: CreateUserData): Promise<User> {
    return this.db.users.create(data)
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    return this.db.users.update({ id }, data)
  }

  async delete(id: string): Promise<void> {
    await this.db.users.delete({ id })
  }
}

// Service uses interface, not implementation
export class UserService {
  constructor(private userRepository: IUserRepository) {}

  async getUser(id: string): Promise<User> {
    const user = await this.userRepository.findById(id)
    if (!user) {
      throw new NotFoundException()
    }
    return user
  }
}
```

**Pros:**
- Testability (easy to mock)
- Flexibility (can swap implementations)
- Separation of concerns
- Consistent data access

**Cons:**
- Additional abstraction layer
- Boilerplate code

**When to use:**
- Most applications
- Need testability
- Multiple data sources possible

### Unit of Work Pattern

**Pattern**: Group related database operations into a transaction

```typescript
export class UnitOfWork {
  private operations: Array<() => Promise<void>> = []

  constructor(private db: Database) {}

  registerNew<T>(repository: string, entity: T) {
    this.operations.push(async () => {
      await this.db[repository].insert(entity)
    })
  }

  registerDirty<T>(repository: string, id: string, changes: Partial<T>) {
    this.operations.push(async () => {
      await this.db[repository].update({ id }, changes)
    })
  }

  registerDeleted(repository: string, id: string) {
    this.operations.push(async () => {
      await this.db[repository].delete({ id })
    })
  }

  async commit() {
    const transaction = await this.db.beginTransaction()

    try {
      for (const operation of this.operations) {
        await operation()
      }
      await transaction.commit()
      this.operations = []
    } catch (error) {
      await transaction.rollback()
      throw error
    }
  }
}

// Usage
async function transferMoney(fromId: string, toId: string, amount: number) {
  const uow = new UnitOfWork(db)

  const fromAccount = await accountRepository.findById(fromId)
  const toAccount = await accountRepository.findById(toId)

  fromAccount.balance -= amount
  toAccount.balance += amount

  uow.registerDirty('accounts', fromId, { balance: fromAccount.balance })
  uow.registerDirty('accounts', toId, { balance: toAccount.balance })

  await uow.commit() // Both updates in one transaction
}
```

---

## Caching Patterns

### Cache-Aside Pattern

**Pattern**: Application manages cache

```typescript
export class UserService {
  constructor(
    private cache: CacheService,
    private db: Database
  ) {}

  async getUser(id: string): Promise<User> {
    // Try cache first
    const cached = await this.cache.get(`user:${id}`)
    if (cached) {
      return JSON.parse(cached)
    }

    // Cache miss - fetch from database
    const user = await this.db.users.findOne({ id })
    if (!user) {
      throw new NotFoundException()
    }

    // Update cache
    await this.cache.set(`user:${id}`, JSON.stringify(user), { ttl: 3600 })

    return user
  }

  async updateUser(id: string, data: Partial<User>): Promise<User> {
    const user = await this.db.users.update({ id }, data)

    // Invalidate cache
    await this.cache.del(`user:${id}`)

    return user
  }
}
```

### Write-Through Cache

**Pattern**: Write to cache and database simultaneously

```typescript
async function updateUser(id: string, data: Partial<User>): Promise<User> {
  const user = await db.users.update({ id }, data)

  // Write to cache
  await cache.set(`user:${id}`, JSON.stringify(user))

  return user
}
```

### Write-Behind Cache

**Pattern**: Write to cache immediately, database asynchronously

```typescript
async function updateUser(id: string, data: Partial<User>): Promise<void> {
  // Update cache immediately
  await cache.set(`user:${id}`, JSON.stringify(data))

  // Queue database update
  await queue.add('update-user', { id, data })
}
```

---

## Security Patterns

### JWT Authentication

```typescript
export class AuthService {
  generateToken(user: User): string {
    return jwt.sign(
      {
        sub: user.id,
        email: user.email,
        role: user.role
      },
      process.env.JWT_SECRET,
      { expiresIn: '1h' }
    )
  }

  verifyToken(token: string): TokenPayload {
    try {
      return jwt.verify(token, process.env.JWT_SECRET)
    } catch {
      throw new UnauthorizedException('Invalid token')
    }
  }
}

// Middleware
export function authMiddleware(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1]

  if (!token) {
    return res.status(401).json({ error: 'No token provided' })
  }

  try {
    req.user = authService.verifyToken(token)
    next()
  } catch {
    res.status(401).json({ error: 'Invalid token' })
  }
}
```

---

## Anti-Patterns to Avoid

### Big Ball of Mud

```typescript
// ❌ Bad: Everything in one file, no structure
export class AppController {
  async handleRequest(req, res) {
    // Database logic
    const db = new Database()
    const user = await db.query('SELECT * FROM users WHERE id = ?', [req.params.id])

    // Business logic
    if (!user) return res.status(404).send('Not found')
    if (user.banned) return res.status(403).send('Banned')

    // Email logic
    const transporter = nodemailer.createTransport({...})
    await transporter.sendMail({...})

    // Response
    res.json(user)
  }
}

// ✅ Good: Separated layers
export class UserController {
  constructor(private userService: UserService) {}

  async getUser(req, res) {
    const user = await this.userService.getUser(req.params.id)
    res.json(user)
  }
}
```

### Distributed Monolith

```typescript
// ❌ Bad: Microservices with shared database
// Service A
await sharedDb.users.update({...})

// Service B
const user = await sharedDb.users.findOne({...})

// ✅ Good: Each service owns its data
// Service A
await userService.updateUser(...) // API call to User Service

// User Service
await this.db.users.update({...}) // Owns users table
```

### God Object

```typescript
// ❌ Bad: Object does everything
export class UserManager {
  createUser() {}
  updateUser() {}
  deleteUser() {}
  sendEmail() {}
  processPayment() {}
  generateReport() {}
  // ... 50 more methods
}

// ✅ Good: Focused services
export class UserService {
  createUser() {}
  updateUser() {}
  deleteUser() {}
}

export class EmailService {
  sendEmail() {}
}

export class PaymentService {
  processPayment() {}
}
```
