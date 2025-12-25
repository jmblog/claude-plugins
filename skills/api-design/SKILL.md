---
name: api-design
description: "Design and review REST/GraphQL APIs with best practices, consistency, and scalability. Use when users request: (1) API design or review, (2) RESTful API planning, (3) GraphQL schema design, (4) API documentation review, (5) Endpoint naming and structure, or (6) API versioning strategies. Applies industry standards, REST principles, GraphQL best practices, and security considerations."
---

# API Design

## Overview

This skill provides comprehensive guidance for designing, reviewing, and improving REST and GraphQL APIs. It focuses on consistency, scalability, security, developer experience, and adherence to industry standards.

## Core Design Workflow

When designing or reviewing an API:

1. **Understand Requirements**
   - Identify the API purpose and target consumers
   - Understand data models and relationships
   - Determine scalability and performance requirements
   - Consider authentication and authorization needs

2. **Design API Structure**
   - Choose API style (REST, GraphQL, or hybrid)
   - Define resource hierarchy and relationships
   - Plan endpoint structure and naming conventions
   - Design request/response schemas
   - Plan error handling and status codes

3. **Review Security and Performance**
   - Validate authentication mechanisms
   - Check authorization and permissions
   - Review rate limiting and throttling
   - Assess caching strategies
   - Evaluate pagination and filtering

4. **Document and Validate**
   - Create OpenAPI/GraphQL schema documentation
   - Provide usage examples
   - Define error codes and messages
   - Plan versioning strategy

## Quick Reference: Key Principles

### REST API Design Principles

**Resource Naming:**
- Use nouns, not verbs (`/users`, not `/getUsers`)
- Use plural names for collections (`/products`, not `/product`)
- Use kebab-case for multi-word resources (`/order-items`)
- Keep URLs hierarchical (`/users/{id}/orders`)

**HTTP Methods:**
- `GET` - Retrieve resource(s), idempotent, no side effects
- `POST` - Create new resource, non-idempotent
- `PUT` - Replace entire resource, idempotent
- `PATCH` - Partial update, idempotent
- `DELETE` - Remove resource, idempotent

**Status Codes:**
- `2xx` - Success (200 OK, 201 Created, 204 No Content)
- `3xx` - Redirection (301 Moved, 304 Not Modified)
- `4xx` - Client errors (400 Bad Request, 401 Unauthorized, 404 Not Found)
- `5xx` - Server errors (500 Internal Server Error, 503 Service Unavailable)

**Response Design:**
- Consistent structure across endpoints
- Include metadata (pagination, timestamps)
- Use standard date formats (ISO 8601)
- Return appropriate status codes
- Provide clear error messages

### GraphQL Design Principles

**Schema Design:**
- Use clear, descriptive type names
- Design types around business domain
- Use interfaces for shared fields
- Implement proper nullability
- Avoid deeply nested queries

**Query Design:**
- Provide filtering and sorting arguments
- Implement cursor-based pagination
- Use connections pattern for lists
- Design for N+1 prevention (DataLoader)

**Mutation Design:**
- Use verb-object naming (`createUser`, `updateProduct`)
- Return modified object in response
- Include success/error information
- Design input types for complex data

### Security Principles

**Authentication & Authorization:**
- Use industry-standard auth (OAuth 2.0, JWT)
- Implement proper token validation
- Use HTTPS for all endpoints
- Validate permissions at every level
- Avoid exposing sensitive data

**Input Validation:**
- Validate all input parameters
- Use strong typing and schemas
- Sanitize user input
- Set appropriate size limits
- Implement rate limiting

**Data Protection:**
- Never expose internal IDs in URLs (use UUIDs or slugs)
- Filter sensitive fields from responses
- Implement field-level permissions
- Use proper CORS configuration
- Log security events

## Common Use Cases

### REST API Design

For building RESTful APIs:
- Resource modeling and hierarchy
- CRUD operation endpoints
- Filtering, sorting, and pagination
- Batch operations
- File uploads/downloads
- Webhook implementations

### GraphQL API Design

For building GraphQL APIs:
- Schema design and type system
- Query and mutation design
- Subscription for real-time data
- Federation for microservices
- Batching and caching strategies

### API Migration

For evolving existing APIs:
- Versioning strategies (URL, header, content negotiation)
- Breaking vs non-breaking changes
- Deprecation planning
- Backward compatibility
- Migration documentation

### API Documentation

For documenting APIs:
- OpenAPI (Swagger) specifications
- GraphQL schema documentation
- Interactive documentation (Swagger UI, GraphQL Playground)
- Example requests and responses
- Authentication guides

## Design Checklist

Use `references/checklist.md` for comprehensive API review covering:
- Resource design and naming
- HTTP methods and status codes
- Request/response structure
- Error handling
- Authentication and authorization
- Performance and caching
- Versioning and compatibility
- Documentation quality

## Resources

### references/patterns.md

Comprehensive guide to API patterns including:
- REST resource patterns
- GraphQL schema patterns
- Pagination strategies
- Filtering and sorting
- Error response formats
- Authentication flows
- Real-world examples and anti-patterns

### references/checklist.md

Structured evaluation framework:
- REST API design criteria
- GraphQL schema review
- Security assessment
- Performance considerations
- Documentation standards

## REST API Conventions

### URL Structure

```
GET    /api/v1/users              # List users
GET    /api/v1/users/{id}         # Get single user
POST   /api/v1/users              # Create user
PUT    /api/v1/users/{id}         # Update user (full)
PATCH  /api/v1/users/{id}         # Update user (partial)
DELETE /api/v1/users/{id}         # Delete user

GET    /api/v1/users/{id}/orders  # List user's orders
POST   /api/v1/users/{id}/orders  # Create order for user
```

### Query Parameters

```
# Filtering
GET /api/v1/products?category=electronics&price_min=100

# Sorting
GET /api/v1/users?sort=created_at&order=desc

# Pagination
GET /api/v1/posts?page=2&limit=20
GET /api/v1/posts?offset=40&limit=20

# Field selection
GET /api/v1/users?fields=id,name,email

# Search
GET /api/v1/products?q=laptop
```

### Response Format

```json
{
  "data": {
    "id": "123",
    "type": "user",
    "attributes": {
      "name": "John Doe",
      "email": "john@example.com"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

### Error Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}
```

## GraphQL Conventions

### Schema Design

```graphql
# Types
type User {
  id: ID!
  name: String!
  email: String!
  posts(first: Int, after: String): PostConnection!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  publishedAt: DateTime
}

# Connections (pagination)
type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type PostEdge {
  node: Post!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

# Queries
type Query {
  user(id: ID!): User
  users(first: Int, after: String, filter: UserFilter): UserConnection!
  searchUsers(query: String!): [User!]!
}

# Mutations
type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(input: UpdateUserInput!): UpdateUserPayload!
  deleteUser(id: ID!): DeleteUserPayload!
}

# Input types
input CreateUserInput {
  name: String!
  email: String!
}

# Payload types
type CreateUserPayload {
  user: User
  errors: [UserError!]
}

type UserError {
  field: String!
  message: String!
}
```

## Versioning Strategies

### URL Versioning (Recommended for REST)
```
GET /api/v1/users
GET /api/v2/users
```

### Header Versioning
```
GET /api/users
Accept: application/vnd.myapi.v1+json
```

### Query Parameter Versioning
```
GET /api/users?version=1
```

### GraphQL Versioning (Field Deprecation)
```graphql
type User {
  name: String! @deprecated(reason: "Use firstName and lastName")
  firstName: String!
  lastName: String!
}
```

## Performance Patterns

### Caching Headers
```
Cache-Control: public, max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
Last-Modified: Wed, 15 Jan 2024 10:30:00 GMT
```

### Rate Limiting Headers
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1609459200
```

### Pagination
```
Link: <https://api.example.com/users?page=3>; rel="next",
      <https://api.example.com/users?page=1>; rel="prev"
```

## Anti-Patterns to Avoid

### REST Anti-Patterns
- Using verbs in URLs (`/getUser`, `/createPost`)
- Not using proper HTTP methods
- Inconsistent naming conventions
- Deeply nested URLs (`/users/123/posts/456/comments/789/likes`)
- Exposing database structure directly
- Missing or incorrect status codes
- Inconsistent error responses

### GraphQL Anti-Patterns
- Overly nested queries (depth > 5)
- N+1 query problems
- Exposing database schema directly
- Not using DataLoader for batching
- Missing pagination
- Unclear error handling
- Mutation design inconsistency

### Security Anti-Patterns
- Exposing sensitive data in responses
- Missing authentication on endpoints
- No rate limiting
- SQL injection vulnerabilities
- Missing input validation
- Verbose error messages in production

## Ethical Considerations

When designing APIs:
- **Privacy**: Don't expose user data without consent
- **Security**: Implement proper authentication and encryption
- **Accessibility**: Provide clear documentation and examples
- **Performance**: Design for efficiency and scalability
- **Transparency**: Document limitations and rate limits
- **Backward Compatibility**: Don't break existing clients without notice
