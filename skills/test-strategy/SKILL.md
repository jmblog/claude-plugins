---
name: test-strategy
description: "Plan and implement comprehensive testing strategies for software projects. Use when users request: (1) Test planning and strategy, (2) Test case generation, (3) Testing best practices, (4) Test coverage improvement, (5) Mock/stub design, or (6) TDD/BDD guidance. Applies unit, integration, E2E testing principles, and test automation best practices."
---

# Test Strategy

## Overview

This skill provides comprehensive guidance for planning, designing, and implementing effective testing strategies. It covers unit tests, integration tests, E2E tests, test automation, and quality assurance practices for modern software development.

## Core Testing Workflow

When planning or reviewing a testing strategy:

1. **Understand the System**
   - Identify the application architecture (frontend, backend, full-stack)
   - Understand critical user flows and business logic
   - Determine existing test coverage and gaps
   - Review current testing tools and frameworks

2. **Design Test Strategy**
   - Apply testing pyramid principle (unit → integration → E2E)
   - Identify what to test at each level
   - Plan test data and fixtures
   - Design mock/stub strategies
   - Plan test environment setup

3. **Implement Tests**
   - Write unit tests for business logic
   - Create integration tests for component interactions
   - Develop E2E tests for critical user journeys
   - Set up test automation and CI/CD integration
   - Implement test utilities and helpers

4. **Review and Improve**
   - Analyze test coverage metrics
   - Identify flaky or slow tests
   - Refactor tests for maintainability
   - Update tests with code changes

## Quick Reference: Testing Principles

### Testing Pyramid

**Unit Tests (70%)**
- Fast, isolated, focused on single units
- Test individual functions, classes, components
- Mock external dependencies
- Run frequently during development

**Integration Tests (20%)**
- Test component interactions
- Verify database operations
- Test API integrations
- Validate service connections

**E2E Tests (10%)**
- Test complete user flows
- Verify UI interactions
- Test cross-system integration
- Validate critical business processes

### Test Characteristics (FIRST)

- **Fast**: Tests should run quickly
- **Independent**: Tests should not depend on each other
- **Repeatable**: Same results every time
- **Self-Validating**: Clear pass/fail without manual inspection
- **Timely**: Written before or with the code (TDD)

### AAA Pattern

- **Arrange**: Set up test data and preconditions
- **Act**: Execute the code under test
- **Assert**: Verify the expected outcome

### Test Coverage Goals

- **Critical paths**: 100% coverage
- **Business logic**: 90%+ coverage
- **Utility functions**: 80%+ coverage
- **UI components**: 70%+ coverage
- **Edge cases**: Comprehensive coverage

## Common Use Cases

### Unit Testing

For testing individual components:
- Pure functions and business logic
- Class methods and modules
- React/Vue components in isolation
- Utility functions and helpers
- Validation logic
- Data transformations

### Integration Testing

For testing component interactions:
- API endpoint testing
- Database operations
- Service layer integration
- External API mocking
- Authentication flows
- Message queue integration

### E2E Testing

For testing user workflows:
- Login and authentication flows
- Checkout and payment processes
- Form submissions
- Navigation and routing
- Search and filtering
- Multi-step wizards

### Test-Driven Development (TDD)

For TDD workflow:
- Write failing test first (Red)
- Write minimal code to pass (Green)
- Refactor code while keeping tests green (Refactor)
- Repeat cycle

### Behavior-Driven Development (BDD)

For BDD scenarios:
- Given-When-Then format
- Business-readable test descriptions
- Scenario-based testing
- Collaboration with stakeholders

## Testing Checklist

Use `references/checklist.md` for comprehensive testing review covering:
- Test strategy and planning
- Unit test coverage and quality
- Integration test design
- E2E test scenarios
- Test maintainability
- Test performance
- CI/CD integration
- Testing documentation

## Resources

### references/patterns.md

Comprehensive guide to testing patterns including:
- Unit test patterns for different scenarios
- Integration test patterns
- E2E test best practices
- Mocking and stubbing strategies
- Test data management
- Async testing patterns
- Error case testing
- Real-world examples and anti-patterns

### references/checklist.md

Structured evaluation framework:
- Test coverage assessment
- Test quality metrics
- Test organization
- Common testing issues
- Improvement recommendations

## Testing Frameworks

### Frontend Testing

**Unit/Component Testing:**
- Jest + React Testing Library (React)
- Vitest + Vue Test Utils (Vue)
- Jasmine + Karma (Angular)

**E2E Testing:**
- Playwright (recommended)
- Cypress
- Selenium WebDriver

### Backend Testing

**Unit Testing:**
- Jest (Node.js)
- Vitest (Node.js/Vite)
- pytest (Python)
- JUnit (Java)

**Integration Testing:**
- Supertest (Express/Node.js)
- TestContainers (Docker-based testing)

**API Testing:**
- REST Client
- Postman/Newman
- Insomnia

### Database Testing

- In-memory databases (SQLite)
- Docker test containers
- Transaction rollback pattern
- Database seeding and fixtures

## Test Organization

### Directory Structure

```
src/
├── components/
│   ├── Button.tsx
│   └── Button.test.tsx
├── services/
│   ├── userService.ts
│   └── userService.test.ts
└── utils/
    ├── validation.ts
    └── validation.test.ts

tests/
├── unit/
│   └── helpers/
├── integration/
│   ├── api/
│   └── database/
└── e2e/
    ├── auth.spec.ts
    └── checkout.spec.ts
```

### Naming Conventions

```typescript
// ✅ Good: Descriptive test names
describe('UserService', () => {
  describe('createUser', () => {
    it('should create a new user with valid data', () => {})
    it('should throw ValidationError for invalid email', () => {})
    it('should throw ConflictError when email already exists', () => {})
  })
})

// ❌ Bad: Vague test names
describe('UserService', () => {
  it('test1', () => {})
  it('works', () => {})
  it('error', () => {})
})
```

## Mocking Strategies

### Mock External Dependencies

```typescript
// ✅ Good: Mock external services
jest.mock('../services/emailService')

test('should send welcome email after registration', async () => {
  const mockSendEmail = jest.fn().mockResolvedValue(true)
  emailService.send = mockSendEmail

  await userService.register({ email: 'test@example.com' })

  expect(mockSendEmail).toHaveBeenCalledWith({
    to: 'test@example.com',
    template: 'welcome'
  })
})
```

### Dependency Injection

```typescript
// ✅ Good: Inject dependencies for testability
class UserService {
  constructor(
    private db: Database,
    private emailService: EmailService
  ) {}

  async register(data: UserData) {
    const user = await this.db.users.create(data)
    await this.emailService.send(user.email, 'welcome')
    return user
  }
}

// Test with mocks
test('register user', async () => {
  const mockDb = { users: { create: jest.fn() } }
  const mockEmail = { send: jest.fn() }
  const service = new UserService(mockDb, mockEmail)

  await service.register({ email: 'test@example.com' })

  expect(mockDb.users.create).toHaveBeenCalled()
  expect(mockEmail.send).toHaveBeenCalled()
})
```

## Test Data Management

### Fixtures

```typescript
// fixtures/users.ts
export const testUsers = {
  admin: {
    id: 'admin-1',
    email: 'admin@example.com',
    role: 'admin'
  },
  regular: {
    id: 'user-1',
    email: 'user@example.com',
    role: 'user'
  }
}

// Usage in tests
import { testUsers } from '../fixtures/users'

test('admin can access admin panel', () => {
  const result = checkPermission(testUsers.admin, 'admin_panel')
  expect(result).toBe(true)
})
```

### Factory Pattern

```typescript
// factories/userFactory.ts
export function createUser(overrides?: Partial<User>): User {
  return {
    id: faker.string.uuid(),
    email: faker.internet.email(),
    name: faker.person.fullName(),
    createdAt: new Date(),
    ...overrides
  }
}

// Usage
test('user creation', () => {
  const user = createUser({ email: 'test@example.com' })
  expect(user.email).toBe('test@example.com')
  expect(user.id).toBeDefined()
})
```

## Async Testing

### Promises

```typescript
// ✅ Good: Return promise or use async/await
test('should fetch user data', async () => {
  const user = await userService.getUser('123')
  expect(user.id).toBe('123')
})

// or
test('should fetch user data', () => {
  return userService.getUser('123').then(user => {
    expect(user.id).toBe('123')
  })
})
```

### Timeouts

```typescript
test('should timeout after 5 seconds', async () => {
  await expect(
    slowOperation()
  ).rejects.toThrow('Timeout')
}, 10000) // 10 second timeout for test
```

## Error Testing

```typescript
// Test expected errors
test('should throw error for invalid input', async () => {
  await expect(
    userService.createUser({ email: 'invalid' })
  ).rejects.toThrow(ValidationError)
})

// Test error messages
test('should have descriptive error message', async () => {
  await expect(
    userService.createUser({ email: 'invalid' })
  ).rejects.toThrow('Invalid email format')
})
```

## CI/CD Integration

### GitHub Actions

```yaml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm test
      - run: npm run test:coverage
      - uses: codecov/codecov-action@v3
```

### Test Scripts

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:unit": "jest --testPathPattern=unit",
    "test:integration": "jest --testPathPattern=integration",
    "test:e2e": "playwright test"
  }
}
```

## Anti-Patterns to Avoid

### Testing Anti-Patterns

- **Testing implementation details**: Test behavior, not internal structure
- **Fragile selectors**: Use data-testid instead of CSS classes
- **Test interdependence**: Each test should be independent
- **Not testing edge cases**: Test boundaries and error conditions
- **Excessive mocking**: Don't mock everything (integration tests)
- **Slow tests**: Optimize test performance
- **No test maintenance**: Keep tests updated with code changes
- **Testing private methods**: Test public API only
- **Snapshot testing overuse**: Use sparingly for UI components
- **No assertion messages**: Provide clear failure messages

### Common Mistakes

```typescript
// ❌ Bad: Testing implementation details
test('should call setState with user data', () => {
  const component = render(<UserProfile />)
  expect(component.setState).toHaveBeenCalled()
})

// ✅ Good: Testing behavior
test('should display user name after loading', async () => {
  render(<UserProfile userId="123" />)
  expect(await screen.findByText('John Doe')).toBeInTheDocument()
})

// ❌ Bad: Dependent tests
let userId
test('should create user', () => {
  userId = createUser()
})
test('should get user', () => {
  const user = getUser(userId) // Depends on previous test
})

// ✅ Good: Independent tests
test('should create user', () => {
  const userId = createUser()
  expect(userId).toBeDefined()
})
test('should get user', () => {
  const userId = createUser()
  const user = getUser(userId)
  expect(user).toBeDefined()
})
```

## Ethical Considerations

When testing:
- **Security Testing**: Don't perform unauthorized security testing
- **Data Privacy**: Use synthetic test data, never production data with PII
- **Performance**: Don't run tests that could impact production systems
- **Accessibility**: Include accessibility testing in your strategy
- **Honest Coverage**: Don't inflate metrics with meaningless tests
- **Documentation**: Maintain clear test documentation for team knowledge
