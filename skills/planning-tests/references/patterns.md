# Test Patterns and Examples

This reference provides detailed testing patterns, real-world examples, and anti-patterns.

## Table of Contents

1. [Unit Test Patterns](#unit-test-patterns)
2. [Integration Test Patterns](#integration-test-patterns)
3. [E2E Test Patterns](#e2e-test-patterns)
4. [Mocking Patterns](#mocking-patterns)
5. [Async Testing Patterns](#async-testing-patterns)
6. [Test Data Patterns](#test-data-patterns)
7. [React Component Testing](#react-component-testing)
8. [API Testing Patterns](#api-testing-patterns)
9. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)

---

## Unit Test Patterns

### Basic Unit Test (AAA Pattern)

**Pattern**: Arrange-Act-Assert structure

```typescript
// ✅ Good: Clear AAA structure
describe('calculateDiscount', () => {
  it('should apply 10% discount for regular customers', () => {
    // Arrange
    const price = 100
    const customerType = 'regular'

    // Act
    const result = calculateDiscount(price, customerType)

    // Assert
    expect(result).toBe(90)
  })

  it('should apply 20% discount for VIP customers', () => {
    // Arrange
    const price = 100
    const customerType = 'vip'

    // Act
    const result = calculateDiscount(price, customerType)

    // Assert
    expect(result).toBe(80)
  })

  it('should return original price for unknown customer type', () => {
    // Arrange
    const price = 100
    const customerType = 'unknown'

    // Act
    const result = calculateDiscount(price, customerType)

    // Assert
    expect(result).toBe(100)
  })
})
```

### Testing Pure Functions

**Pattern**: Exhaustive testing of pure functions

```typescript
// ✅ Good: Test all branches and edge cases
describe('isValidEmail', () => {
  it('should return true for valid email', () => {
    expect(isValidEmail('test@example.com')).toBe(true)
    expect(isValidEmail('user.name+tag@example.co.uk')).toBe(true)
  })

  it('should return false for invalid email', () => {
    expect(isValidEmail('invalid')).toBe(false)
    expect(isValidEmail('missing@domain')).toBe(false)
    expect(isValidEmail('@example.com')).toBe(false)
  })

  it('should return false for empty or null', () => {
    expect(isValidEmail('')).toBe(false)
    expect(isValidEmail(null as any)).toBe(false)
    expect(isValidEmail(undefined as any)).toBe(false)
  })
})
```

### Parametrized Tests

**Pattern**: Use test.each for similar test cases

```typescript
// ✅ Good: Parametrized tests reduce duplication
describe('calculateTax', () => {
  test.each([
    [100, 'NY', 8.875],
    [100, 'CA', 7.25],
    [100, 'TX', 6.25],
    [100, 'OR', 0],
  ])('should calculate tax for %p in %s as %p', (amount, state, expected) => {
    expect(calculateTax(amount, state)).toBe(expected)
  })
})

// ❌ Bad: Repetitive tests
describe('calculateTax', () => {
  it('should calculate NY tax', () => {
    expect(calculateTax(100, 'NY')).toBe(8.875)
  })
  it('should calculate CA tax', () => {
    expect(calculateTax(100, 'CA')).toBe(7.25)
  })
  // ... repetitive
})
```

### Testing Class Methods

**Pattern**: Test public interface, mock dependencies

```typescript
class UserService {
  constructor(
    private db: Database,
    private emailService: EmailService
  ) {}

  async createUser(data: CreateUserData): Promise<User> {
    const existingUser = await this.db.users.findByEmail(data.email)
    if (existingUser) {
      throw new ConflictError('Email already exists')
    }

    const user = await this.db.users.create(data)
    await this.emailService.sendWelcomeEmail(user.email)

    return user
  }
}

// ✅ Good: Test with mocked dependencies
describe('UserService', () => {
  let userService: UserService
  let mockDb: jest.Mocked<Database>
  let mockEmailService: jest.Mocked<EmailService>

  beforeEach(() => {
    mockDb = {
      users: {
        findByEmail: jest.fn(),
        create: jest.fn(),
      },
    } as any

    mockEmailService = {
      sendWelcomeEmail: jest.fn(),
    } as any

    userService = new UserService(mockDb, mockEmailService)
  })

  describe('createUser', () => {
    it('should create user and send welcome email', async () => {
      // Arrange
      const userData = { email: 'test@example.com', name: 'Test' }
      const createdUser = { id: '123', ...userData }

      mockDb.users.findByEmail.mockResolvedValue(null)
      mockDb.users.create.mockResolvedValue(createdUser)

      // Act
      const result = await userService.createUser(userData)

      // Assert
      expect(result).toEqual(createdUser)
      expect(mockDb.users.create).toHaveBeenCalledWith(userData)
      expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalledWith(userData.email)
    })

    it('should throw ConflictError if email exists', async () => {
      // Arrange
      const userData = { email: 'existing@example.com', name: 'Test' }
      mockDb.users.findByEmail.mockResolvedValue({ id: '1', email: userData.email } as any)

      // Act & Assert
      await expect(userService.createUser(userData)).rejects.toThrow(ConflictError)
      expect(mockDb.users.create).not.toHaveBeenCalled()
      expect(mockEmailService.sendWelcomeEmail).not.toHaveBeenCalled()
    })
  })
})
```

### Testing Error Cases

**Pattern**: Comprehensive error testing

```typescript
describe('validatePassword', () => {
  it('should throw ValidationError for short password', () => {
    expect(() => validatePassword('short')).toThrow(ValidationError)
    expect(() => validatePassword('short')).toThrow('Password must be at least 8 characters')
  })

  it('should throw ValidationError for missing uppercase', () => {
    expect(() => validatePassword('password123')).toThrow('Password must contain uppercase letter')
  })

  it('should throw ValidationError for missing number', () => {
    expect(() => validatePassword('Password')).toThrow('Password must contain a number')
  })

  it('should not throw for valid password', () => {
    expect(() => validatePassword('Password123')).not.toThrow()
  })
})
```

---

## Integration Test Patterns

### API Integration Tests

**Pattern**: Test API endpoints with real HTTP calls

```typescript
import request from 'supertest'
import { app } from '../src/app'
import { db } from '../src/database'

describe('POST /api/users', () => {
  beforeEach(async () => {
    await db.migrate.latest()
    await db.seed.run()
  })

  afterEach(async () => {
    await db.migrate.rollback()
  })

  it('should create a new user', async () => {
    const userData = {
      email: 'newuser@example.com',
      name: 'New User',
      password: 'Password123'
    }

    const response = await request(app)
      .post('/api/users')
      .send(userData)
      .expect(201)

    expect(response.body.data).toMatchObject({
      email: userData.email,
      name: userData.name
    })
    expect(response.body.data.id).toBeDefined()
    expect(response.body.data.password).toBeUndefined() // Should not return password

    // Verify in database
    const user = await db('users').where({ email: userData.email }).first()
    expect(user).toBeDefined()
  })

  it('should return 400 for invalid email', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({
        email: 'invalid-email',
        name: 'Test',
        password: 'Password123'
      })
      .expect(400)

    expect(response.body.error).toMatchObject({
      code: 'VALIDATION_ERROR',
      details: expect.arrayContaining([
        expect.objectContaining({
          field: 'email',
          message: expect.stringContaining('email')
        })
      ])
    })
  })

  it('should return 409 for duplicate email', async () => {
    const userData = {
      email: 'existing@example.com',
      name: 'Existing User',
      password: 'Password123'
    }

    // Create first user
    await request(app).post('/api/users').send(userData).expect(201)

    // Try to create duplicate
    const response = await request(app)
      .post('/api/users')
      .send(userData)
      .expect(409)

    expect(response.body.error.code).toBe('RESOURCE_ALREADY_EXISTS')
  })
})
```

### Database Integration Tests

**Pattern**: Test with real database using transactions

```typescript
import { setupTestDatabase, teardownTestDatabase } from './helpers/database'

describe('UserRepository', () => {
  let db: Database
  let userRepo: UserRepository

  beforeAll(async () => {
    db = await setupTestDatabase()
  })

  afterAll(async () => {
    await teardownTestDatabase(db)
  })

  beforeEach(async () => {
    userRepo = new UserRepository(db)
    await db.beginTransaction()
  })

  afterEach(async () => {
    await db.rollbackTransaction()
  })

  it('should create and retrieve user', async () => {
    const userData = {
      email: 'test@example.com',
      name: 'Test User'
    }

    const created = await userRepo.create(userData)
    expect(created.id).toBeDefined()

    const retrieved = await userRepo.findById(created.id)
    expect(retrieved).toMatchObject(userData)
  })

  it('should update user', async () => {
    const user = await userRepo.create({
      email: 'test@example.com',
      name: 'Test User'
    })

    const updated = await userRepo.update(user.id, { name: 'Updated Name' })
    expect(updated.name).toBe('Updated Name')
    expect(updated.email).toBe(user.email) // Unchanged
  })

  it('should delete user', async () => {
    const user = await userRepo.create({
      email: 'test@example.com',
      name: 'Test User'
    })

    await userRepo.delete(user.id)

    const retrieved = await userRepo.findById(user.id)
    expect(retrieved).toBeNull()
  })
})
```

### External Service Integration (with Mocking)

**Pattern**: Mock external services, test integration logic

```typescript
import nock from 'nock'

describe('PaymentService', () => {
  let paymentService: PaymentService

  beforeEach(() => {
    paymentService = new PaymentService()
  })

  afterEach(() => {
    nock.cleanAll()
  })

  it('should process payment successfully', async () => {
    // Mock external payment API
    nock('https://api.stripe.com')
      .post('/v1/charges')
      .reply(200, {
        id: 'ch_123',
        status: 'succeeded',
        amount: 1000
      })

    const result = await paymentService.processPayment({
      amount: 1000,
      currency: 'usd',
      source: 'tok_visa'
    })

    expect(result.success).toBe(true)
    expect(result.transactionId).toBe('ch_123')
  })

  it('should handle payment failure', async () => {
    nock('https://api.stripe.com')
      .post('/v1/charges')
      .reply(402, {
        error: {
          type: 'card_error',
          code: 'card_declined',
          message: 'Your card was declined'
        }
      })

    const result = await paymentService.processPayment({
      amount: 1000,
      currency: 'usd',
      source: 'tok_visa'
    })

    expect(result.success).toBe(false)
    expect(result.error).toBe('Your card was declined')
  })

  it('should retry on network error', async () => {
    nock('https://api.stripe.com')
      .post('/v1/charges')
      .replyWithError('Network error')
      .post('/v1/charges')
      .reply(200, { id: 'ch_123', status: 'succeeded' })

    const result = await paymentService.processPayment({
      amount: 1000,
      currency: 'usd',
      source: 'tok_visa'
    })

    expect(result.success).toBe(true)
  })
})
```

---

## E2E Test Patterns

### Page Object Pattern (Playwright)

**Pattern**: Encapsulate page interactions

```typescript
// pages/LoginPage.ts
export class LoginPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto('/login')
  }

  async login(email: string, password: string) {
    await this.page.fill('[data-testid="email-input"]', email)
    await this.page.fill('[data-testid="password-input"]', password)
    await this.page.click('[data-testid="login-button"]')
  }

  async getErrorMessage() {
    return this.page.textContent('[data-testid="error-message"]')
  }

  async isLoggedIn() {
    return this.page.isVisible('[data-testid="user-menu"]')
  }
}

// tests/login.spec.ts
import { test, expect } from '@playwright/test'
import { LoginPage } from '../pages/LoginPage'

test.describe('Login Flow', () => {
  let loginPage: LoginPage

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page)
    await loginPage.goto()
  })

  test('should login with valid credentials', async ({ page }) => {
    await loginPage.login('user@example.com', 'password123')

    await expect(page).toHaveURL('/dashboard')
    expect(await loginPage.isLoggedIn()).toBe(true)
  })

  test('should show error for invalid credentials', async () => {
    await loginPage.login('user@example.com', 'wrongpassword')

    const error = await loginPage.getErrorMessage()
    expect(error).toContain('Invalid credentials')
  })

  test('should show validation error for empty email', async () => {
    await loginPage.login('', 'password123')

    const error = await loginPage.getErrorMessage()
    expect(error).toContain('Email is required')
  })
})
```

### E2E Workflow Testing

**Pattern**: Test complete user journeys

```typescript
test.describe('E-commerce Checkout Flow', () => {
  test('should complete full checkout process', async ({ page }) => {
    // 1. Browse products
    await page.goto('/products')
    await page.click('[data-testid="product-1"]')

    // 2. Add to cart
    await page.click('[data-testid="add-to-cart"]')
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1')

    // 3. Go to cart
    await page.click('[data-testid="cart-icon"]')
    await expect(page).toHaveURL('/cart')

    // 4. Proceed to checkout
    await page.click('[data-testid="checkout-button"]')

    // 5. Fill shipping info
    await page.fill('[data-testid="address"]', '123 Main St')
    await page.fill('[data-testid="city"]', 'New York')
    await page.fill('[data-testid="zip"]', '10001')

    // 6. Fill payment info
    await page.fill('[data-testid="card-number"]', '4242424242424242')
    await page.fill('[data-testid="expiry"]', '12/25')
    await page.fill('[data-testid="cvc"]', '123')

    // 7. Complete order
    await page.click('[data-testid="place-order"]')

    // 8. Verify confirmation
    await expect(page).toHaveURL(/\/order\/\d+/)
    await expect(page.locator('[data-testid="order-status"]')).toHaveText('Confirmed')
  })
})
```

### Testing with Authentication

**Pattern**: Reuse authentication state

```typescript
// auth.setup.ts
import { test as setup } from '@playwright/test'

setup('authenticate', async ({ page }) => {
  await page.goto('/login')
  await page.fill('[data-testid="email"]', 'test@example.com')
  await page.fill('[data-testid="password"]', 'password123')
  await page.click('[data-testid="login-button"]')

  await page.waitForURL('/dashboard')

  // Save authentication state
  await page.context().storageState({ path: 'playwright/.auth/user.json' })
})

// playwright.config.ts
export default defineConfig({
  projects: [
    {
      name: 'setup',
      testMatch: /auth\.setup\.ts/,
    },
    {
      name: 'authenticated',
      dependencies: ['setup'],
      use: {
        storageState: 'playwright/.auth/user.json',
      },
    },
  ],
})

// tests/dashboard.spec.ts
test.describe('Dashboard', () => {
  test('should display user dashboard', async ({ page }) => {
    // Already authenticated from setup
    await page.goto('/dashboard')
    await expect(page.locator('[data-testid="welcome-message"]')).toBeVisible()
  })
})
```

---

## Mocking Patterns

### Dependency Injection for Testability

**Pattern**: Inject dependencies instead of hardcoding

```typescript
// ❌ Bad: Hard-coded dependencies
class UserService {
  async createUser(data: UserData) {
    const db = new Database() // Hard-coded!
    return db.users.create(data)
  }
}

// ✅ Good: Injected dependencies
class UserService {
  constructor(private db: Database) {}

  async createUser(data: UserData) {
    return this.db.users.create(data)
  }
}

// Test with mock
const mockDb = { users: { create: jest.fn() } }
const service = new UserService(mockDb as any)
```

### Module Mocking

**Pattern**: Mock entire modules

```typescript
// src/services/emailService.ts
export const emailService = {
  send: async (to: string, template: string) => {
    // Real implementation
  }
}

// tests/userService.test.ts
jest.mock('../services/emailService')

import { emailService } from '../services/emailService'

test('should send welcome email', async () => {
  const mockSend = jest.mocked(emailService.send)
  mockSend.mockResolvedValue(true)

  await userService.register({ email: 'test@example.com' })

  expect(mockSend).toHaveBeenCalledWith('test@example.com', 'welcome')
})
```

### Partial Mocking

**Pattern**: Mock only specific methods

```typescript
import * as utils from '../utils'

jest.spyOn(utils, 'generateId').mockReturnValue('fixed-id')

test('should use mocked generateId', () => {
  const result = createUser({ name: 'Test' })
  expect(result.id).toBe('fixed-id')
})
```

### Time Mocking

**Pattern**: Control time in tests

```typescript
describe('TokenService', () => {
  beforeEach(() => {
    jest.useFakeTimers()
    jest.setSystemTime(new Date('2024-01-01'))
  })

  afterEach(() => {
    jest.useRealTimers()
  })

  it('should generate token with correct expiration', () => {
    const token = tokenService.generateToken('user-123')

    const decoded = tokenService.decode(token)
    expect(decoded.exp).toBe(new Date('2024-01-02').getTime()) // 24h from now
  })

  it('should reject expired token', () => {
    const token = tokenService.generateToken('user-123')

    // Fast-forward time
    jest.advanceTimersByTime(25 * 60 * 60 * 1000) // 25 hours

    expect(() => tokenService.verify(token)).toThrow('Token expired')
  })
})
```

---

## Async Testing Patterns

### Promise Testing

**Pattern**: Proper async/await usage

```typescript
// ✅ Good: Using async/await
test('should fetch user data', async () => {
  const user = await userService.getUser('123')
  expect(user.id).toBe('123')
})

// ✅ Good: Using return
test('should fetch user data', () => {
  return userService.getUser('123').then(user => {
    expect(user.id).toBe('123')
  })
})

// ❌ Bad: Missing return or await
test('should fetch user data', () => {
  userService.getUser('123').then(user => {
    expect(user.id).toBe('123') // May not run!
  })
})
```

### Testing Async Errors

**Pattern**: Test rejected promises

```typescript
test('should handle async errors', async () => {
  await expect(userService.getUser('invalid')).rejects.toThrow(NotFoundError)

  await expect(
    userService.getUser('invalid')
  ).rejects.toThrow('User not found')
})
```

### Testing Callbacks

**Pattern**: Use done callback for callback-based code

```typescript
test('should handle callback', (done) => {
  fetchUser('123', (error, user) => {
    expect(error).toBeNull()
    expect(user.id).toBe('123')
    done()
  })
})

test('should handle callback error', (done) => {
  fetchUser('invalid', (error, user) => {
    expect(error).toBeInstanceOf(Error)
    expect(user).toBeUndefined()
    done()
  })
})
```

### Testing Concurrent Operations

**Pattern**: Test parallel async operations

```typescript
test('should handle concurrent requests', async () => {
  const promises = [
    userService.getUser('1'),
    userService.getUser('2'),
    userService.getUser('3'),
  ]

  const users = await Promise.all(promises)

  expect(users).toHaveLength(3)
  expect(users[0].id).toBe('1')
  expect(users[1].id).toBe('2')
  expect(users[2].id).toBe('3')
})
```

---

## Test Data Patterns

### Factory Functions

**Pattern**: Generate test data with factories

```typescript
import { faker } from '@faker-js/faker'

export function createUser(overrides?: Partial<User>): User {
  return {
    id: faker.string.uuid(),
    email: faker.internet.email(),
    name: faker.person.fullName(),
    createdAt: faker.date.past(),
    ...overrides,
  }
}

export function createPost(overrides?: Partial<Post>): Post {
  return {
    id: faker.string.uuid(),
    title: faker.lorem.sentence(),
    content: faker.lorem.paragraphs(3),
    authorId: faker.string.uuid(),
    publishedAt: faker.date.recent(),
    ...overrides,
  }
}

// Usage
test('should create user', () => {
  const user = createUser({ email: 'specific@example.com' })
  expect(user.email).toBe('specific@example.com')
  expect(user.id).toBeDefined()
})
```

### Fixture Files

**Pattern**: Static test data in files

```typescript
// fixtures/users.ts
export const testUsers = {
  admin: {
    id: 'admin-1',
    email: 'admin@example.com',
    name: 'Admin User',
    role: 'admin',
  },
  regular: {
    id: 'user-1',
    email: 'user@example.com',
    name: 'Regular User',
    role: 'user',
  },
  guest: {
    id: 'guest-1',
    email: 'guest@example.com',
    name: 'Guest User',
    role: 'guest',
  },
} as const

// Usage
import { testUsers } from '../fixtures/users'

test('admin can access admin panel', () => {
  const canAccess = checkAccess(testUsers.admin, 'admin_panel')
  expect(canAccess).toBe(true)
})
```

### Database Seeding

**Pattern**: Seed database with test data

```typescript
// seeds/test/users.ts
export async function seed(db: Database) {
  await db('users').del()

  await db('users').insert([
    {
      id: '1',
      email: 'user1@example.com',
      name: 'User 1',
    },
    {
      id: '2',
      email: 'user2@example.com',
      name: 'User 2',
    },
  ])
}

// Usage in tests
beforeEach(async () => {
  await db.seed.run()
})
```

---

## React Component Testing

### Testing with React Testing Library

**Pattern**: Test components from user perspective

```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { UserProfile } from './UserProfile'

describe('UserProfile', () => {
  it('should display user information', () => {
    const user = { name: 'John Doe', email: 'john@example.com' }

    render(<UserProfile user={user} />)

    expect(screen.getByText('John Doe')).toBeInTheDocument()
    expect(screen.getByText('john@example.com')).toBeInTheDocument()
  })

  it('should handle edit button click', async () => {
    const onEdit = jest.fn()
    const user = { name: 'John Doe', email: 'john@example.com' }

    render(<UserProfile user={user} onEdit={onEdit} />)

    const editButton = screen.getByRole('button', { name: /edit/i })
    await userEvent.click(editButton)

    expect(onEdit).toHaveBeenCalledWith(user)
  })

  it('should show loading state', () => {
    render(<UserProfile loading />)

    expect(screen.getByRole('status')).toBeInTheDocument()
    expect(screen.getByText(/loading/i)).toBeInTheDocument()
  })

  it('should show error state', () => {
    const error = new Error('Failed to load user')

    render(<UserProfile error={error} />)

    expect(screen.getByRole('alert')).toBeInTheDocument()
    expect(screen.getByText(/failed to load/i)).toBeInTheDocument()
  })
})
```

### Testing Async Components

**Pattern**: Test components with async data loading

```typescript
describe('UserList', () => {
  it('should load and display users', async () => {
    const users = [
      { id: '1', name: 'User 1' },
      { id: '2', name: 'User 2' },
    ]

    // Mock API call
    jest.spyOn(api, 'getUsers').mockResolvedValue(users)

    render(<UserList />)

    // Initially shows loading
    expect(screen.getByText(/loading/i)).toBeInTheDocument()

    // Wait for users to load
    await waitFor(() => {
      expect(screen.getByText('User 1')).toBeInTheDocument()
    })

    expect(screen.getByText('User 2')).toBeInTheDocument()
    expect(screen.queryByText(/loading/i)).not.toBeInTheDocument()
  })
})
```

### Testing Forms

**Pattern**: Test form interactions and validation

```typescript
describe('LoginForm', () => {
  it('should submit form with valid data', async () => {
    const onSubmit = jest.fn()

    render(<LoginForm onSubmit={onSubmit} />)

    await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com')
    await userEvent.type(screen.getByLabelText(/password/i), 'password123')
    await userEvent.click(screen.getByRole('button', { name: /log in/i }))

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123',
      })
    })
  })

  it('should show validation errors', async () => {
    render(<LoginForm onSubmit={jest.fn()} />)

    // Submit without filling form
    await userEvent.click(screen.getByRole('button', { name: /log in/i }))

    expect(await screen.findByText(/email is required/i)).toBeInTheDocument()
    expect(await screen.findByText(/password is required/i)).toBeInTheDocument()
  })

  it('should disable submit button while submitting', async () => {
    const onSubmit = jest.fn(() => new Promise(resolve => setTimeout(resolve, 1000)))

    render(<LoginForm onSubmit={onSubmit} />)

    await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com')
    await userEvent.type(screen.getByLabelText(/password/i), 'password123')

    const submitButton = screen.getByRole('button', { name: /log in/i })
    await userEvent.click(submitButton)

    expect(submitButton).toBeDisabled()
  })
})
```

---

## API Testing Patterns

See Integration Test Patterns section above for comprehensive API testing examples.

---

## Anti-Patterns to Avoid

### Testing Implementation Details

```typescript
// ❌ Bad: Testing internal state
test('should set isLoading to true', () => {
  const component = render(<UserProfile />)
  expect(component.state.isLoading).toBe(true)
})

// ✅ Good: Testing user-visible behavior
test('should show loading indicator', () => {
  render(<UserProfile />)
  expect(screen.getByRole('status')).toBeInTheDocument()
})
```

### Dependent Tests

```typescript
// ❌ Bad: Tests depend on each other
let userId

test('create user', () => {
  userId = createUser()
})

test('get user', () => {
  const user = getUser(userId) // Depends on previous test!
})

// ✅ Good: Independent tests
test('create user', () => {
  const userId = createUser()
  expect(userId).toBeDefined()
})

test('get user', () => {
  const userId = createUser() // Create fresh data
  const user = getUser(userId)
  expect(user).toBeDefined()
})
```

### Over-Mocking

```typescript
// ❌ Bad: Mocking everything in integration tests
test('should create order', () => {
  mockDb()
  mockPaymentService()
  mockEmailService()
  mockInventoryService()
  // Not really testing integration!
})

// ✅ Good: Use real dependencies for integration
test('should create order', () => {
  // Use real database, mock only external services
  const order = await orderService.createOrder(data)
  expect(order).toBeDefined()
})
```

### Brittle Selectors

```typescript
// ❌ Bad: Fragile CSS selectors
const button = page.locator('.btn.btn-primary.mt-4')

// ✅ Good: Stable test IDs
const button = page.locator('[data-testid="submit-button"]')

// ✅ Good: Role-based selectors
const button = page.getByRole('button', { name: /submit/i })
```

### Not Testing Edge Cases

```typescript
// ❌ Bad: Only testing happy path
test('calculate total', () => {
  expect(calculateTotal([10, 20])).toBe(30)
})

// ✅ Good: Test edge cases
describe('calculateTotal', () => {
  it('should sum numbers', () => {
    expect(calculateTotal([10, 20, 30])).toBe(60)
  })

  it('should handle empty array', () => {
    expect(calculateTotal([])).toBe(0)
  })

  it('should handle negative numbers', () => {
    expect(calculateTotal([10, -5])).toBe(5)
  })

  it('should handle decimals', () => {
    expect(calculateTotal([1.5, 2.5])).toBe(4.0)
  })
})
```
