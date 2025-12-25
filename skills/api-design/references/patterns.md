# API Design Patterns and Examples

This reference provides detailed patterns, real-world examples, and anti-patterns for REST and GraphQL API design.

## Table of Contents

1. [REST Resource Patterns](#rest-resource-patterns)
2. [GraphQL Schema Patterns](#graphql-schema-patterns)
3. [Pagination Patterns](#pagination-patterns)
4. [Filtering and Sorting](#filtering-and-sorting)
5. [Error Response Patterns](#error-response-patterns)
6. [Authentication Patterns](#authentication-patterns)
7. [Versioning Patterns](#versioning-patterns)
8. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)

---

## REST Resource Patterns

### Basic CRUD Resources

**Pattern**: Standard resource operations

```http
# List resources (with pagination)
GET /api/v1/users?page=1&limit=20
Response: 200 OK
{
  "data": [
    { "id": "123", "name": "John Doe", "email": "john@example.com" }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}

# Get single resource
GET /api/v1/users/123
Response: 200 OK
{
  "data": {
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com",
    "createdAt": "2024-01-15T10:30:00Z"
  }
}

# Create resource
POST /api/v1/users
Content-Type: application/json
{
  "name": "Jane Smith",
  "email": "jane@example.com"
}
Response: 201 Created
Location: /api/v1/users/124
{
  "data": {
    "id": "124",
    "name": "Jane Smith",
    "email": "jane@example.com",
    "createdAt": "2024-01-15T11:00:00Z"
  }
}

# Update resource (full replacement)
PUT /api/v1/users/123
Content-Type: application/json
{
  "name": "John Updated",
  "email": "john.new@example.com"
}
Response: 200 OK
{
  "data": {
    "id": "123",
    "name": "John Updated",
    "email": "john.new@example.com",
    "updatedAt": "2024-01-15T11:30:00Z"
  }
}

# Update resource (partial)
PATCH /api/v1/users/123
Content-Type: application/json
{
  "name": "John Patched"
}
Response: 200 OK
{
  "data": {
    "id": "123",
    "name": "John Patched",
    "email": "john@example.com",
    "updatedAt": "2024-01-15T11:45:00Z"
  }
}

# Delete resource
DELETE /api/v1/users/123
Response: 204 No Content
```

### Nested Resources

**Pattern**: Parent-child relationships

```http
# List orders for a user
GET /api/v1/users/123/orders
Response: 200 OK
{
  "data": [
    {
      "id": "order-456",
      "userId": "123",
      "total": 99.99,
      "status": "completed"
    }
  ]
}

# Create order for a user
POST /api/v1/users/123/orders
{
  "items": [
    { "productId": "prod-789", "quantity": 2 }
  ]
}
Response: 201 Created
Location: /api/v1/orders/order-456

# Get specific order (prefer flat endpoint)
GET /api/v1/orders/order-456
# Better than: GET /api/v1/users/123/orders/order-456
```

### Bulk Operations

**Pattern**: Batch create/update/delete

```http
# Bulk create
POST /api/v1/users/bulk
{
  "users": [
    { "name": "User 1", "email": "user1@example.com" },
    { "name": "User 2", "email": "user2@example.com" }
  ]
}
Response: 201 Created
{
  "data": {
    "created": 2,
    "users": [
      { "id": "125", "name": "User 1" },
      { "id": "126", "name": "User 2" }
    ]
  }
}

# Bulk update
PATCH /api/v1/users/bulk
{
  "updates": [
    { "id": "123", "status": "active" },
    { "id": "124", "status": "inactive" }
  ]
}
Response: 200 OK
{
  "data": {
    "updated": 2,
    "failed": 0
  }
}

# Bulk delete
DELETE /api/v1/users/bulk
{
  "ids": ["123", "124", "125"]
}
Response: 200 OK
{
  "data": {
    "deleted": 3
  }
}
```

### Custom Actions

**Pattern**: Non-CRUD operations as sub-resources

```http
# ✅ Good: Use POST for actions
POST /api/v1/users/123/suspend
{
  "reason": "Terms violation",
  "duration": "30d"
}
Response: 200 OK
{
  "data": {
    "id": "123",
    "status": "suspended",
    "suspendedUntil": "2024-02-15T10:30:00Z"
  }
}

# Password reset
POST /api/v1/users/123/password-reset
Response: 202 Accepted
{
  "data": {
    "message": "Password reset email sent"
  }
}

# Email verification
POST /api/v1/users/123/verify-email
{
  "code": "123456"
}
Response: 200 OK

# ❌ Bad: Don't use verbs in main resource path
POST /api/v1/suspendUser  # Wrong!
GET /api/v1/resetPassword # Wrong!
```

### File Upload

**Pattern**: Multipart file upload

```http
# Single file upload
POST /api/v1/users/123/avatar
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="avatar.jpg"
Content-Type: image/jpeg

[binary data]
------WebKitFormBoundary--

Response: 201 Created
{
  "data": {
    "url": "https://cdn.example.com/avatars/123.jpg",
    "size": 45678,
    "contentType": "image/jpeg"
  }
}

# Direct upload with presigned URL (recommended for large files)
POST /api/v1/uploads/presign
{
  "filename": "large-file.pdf",
  "contentType": "application/pdf",
  "size": 10485760
}
Response: 200 OK
{
  "data": {
    "uploadUrl": "https://storage.example.com/...",
    "fields": {
      "key": "uploads/uuid.pdf",
      "policy": "...",
      "signature": "..."
    },
    "expiresAt": "2024-01-15T12:00:00Z"
  }
}
```

---

## GraphQL Schema Patterns

### Basic Type Definitions

**Pattern**: Clean type design with nullability

```graphql
# Scalar types
scalar DateTime
scalar Email
scalar URL
scalar JSON

# Object types
type User {
  id: ID!
  email: Email!
  name: String!
  bio: String
  avatar: URL
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  publishedAt: DateTime
  author: User!
  comments(first: Int, after: String): CommentConnection!
  tags: [Tag!]!
}

type Tag {
  id: ID!
  name: String!
  posts(first: Int): PostConnection!
}
```

### Interfaces and Unions

**Pattern**: Polymorphic types

```graphql
# Interface for common fields
interface Node {
  id: ID!
  createdAt: DateTime!
  updatedAt: DateTime!
}

interface Timestamped {
  createdAt: DateTime!
  updatedAt: DateTime!
}

type User implements Node & Timestamped {
  id: ID!
  email: Email!
  name: String!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Post implements Node & Timestamped {
  id: ID!
  title: String!
  createdAt: DateTime!
  updatedAt: DateTime!
}

# Union for search results
union SearchResult = User | Post | Comment

type Query {
  search(query: String!): [SearchResult!]!
}

# Usage in resolver
query {
  search(query: "GraphQL") {
    ... on User {
      name
      email
    }
    ... on Post {
      title
      content
    }
    ... on Comment {
      text
      author { name }
    }
  }
}
```

### Connection Pattern (Pagination)

**Pattern**: Relay-style cursor pagination

```graphql
type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

type PostEdge {
  node: Post!
  cursor: String!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type Query {
  posts(
    first: Int
    after: String
    last: Int
    before: String
  ): PostConnection!
}

# Query usage
query {
  posts(first: 10, after: "cursor123") {
    edges {
      node {
        id
        title
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
    totalCount
  }
}
```

### Mutation Patterns

**Pattern**: Input types and payload types

```graphql
# Input types for mutations
input CreateUserInput {
  email: Email!
  name: String!
  password: String!
}

input UpdateUserInput {
  id: ID!
  email: Email
  name: String
  bio: String
}

# Error type
type UserError {
  field: String!
  message: String!
}

# Payload types
type CreateUserPayload {
  user: User
  errors: [UserError!]
}

type UpdateUserPayload {
  user: User
  errors: [UserError!]
}

type DeleteUserPayload {
  deletedId: ID
  success: Boolean!
  errors: [UserError!]
}

# Mutations
type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(input: UpdateUserInput!): UpdateUserPayload!
  deleteUser(id: ID!): DeleteUserPayload!
}

# Usage
mutation {
  createUser(input: {
    email: "john@example.com"
    name: "John Doe"
    password: "secure123"
  }) {
    user {
      id
      name
      email
    }
    errors {
      field
      message
    }
  }
}
```

### Subscription Pattern

**Pattern**: Real-time updates

```graphql
type Subscription {
  postCreated(authorId: ID): Post!
  commentAdded(postId: ID!): Comment!
  userStatusChanged(userId: ID!): UserStatus!
}

type UserStatus {
  userId: ID!
  status: OnlineStatus!
  lastSeen: DateTime
}

enum OnlineStatus {
  ONLINE
  AWAY
  OFFLINE
}

# Usage
subscription {
  commentAdded(postId: "post-123") {
    id
    text
    author {
      name
    }
    createdAt
  }
}
```

---

## Pagination Patterns

### Offset-Based Pagination (REST)

**Pattern**: Simple page-based pagination

```http
GET /api/v1/posts?page=2&limit=20

Response:
{
  "data": [...],
  "meta": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  },
  "links": {
    "first": "/api/v1/posts?page=1&limit=20",
    "prev": "/api/v1/posts?page=1&limit=20",
    "next": "/api/v1/posts?page=3&limit=20",
    "last": "/api/v1/posts?page=8&limit=20"
  }
}
```

### Cursor-Based Pagination (REST)

**Pattern**: Efficient pagination for large datasets

```http
GET /api/v1/posts?limit=20&cursor=eyJpZCI6MTIzfQ

Response:
{
  "data": [...],
  "pagination": {
    "nextCursor": "eyJpZCI6MTQzfQ",
    "prevCursor": "eyJpZCI6MTAzfQ",
    "hasMore": true
  }
}
```

### Keyset Pagination

**Pattern**: Pagination using last seen ID

```http
GET /api/v1/posts?limit=20&after_id=123

Response:
{
  "data": [
    { "id": "124", ... },
    { "id": "125", ... }
  ],
  "pagination": {
    "lastId": "143",
    "hasMore": true
  }
}
```

---

## Filtering and Sorting

### Query Parameter Filtering

**Pattern**: Multiple filter strategies

```http
# Simple equality
GET /api/v1/products?category=electronics&status=active

# Comparison operators
GET /api/v1/products?price_min=100&price_max=500

# Multiple values (OR)
GET /api/v1/products?category=electronics,computers

# Pattern matching
GET /api/v1/products?name_contains=laptop

# Date ranges
GET /api/v1/orders?created_after=2024-01-01&created_before=2024-01-31

# Complex filters (JSON)
GET /api/v1/products?filter={"price":{"gte":100,"lte":500},"category":"electronics"}
```

### Sorting

**Pattern**: Flexible sort options

```http
# Single field
GET /api/v1/products?sort=price

# Descending
GET /api/v1/products?sort=-price
# or
GET /api/v1/products?sort=price&order=desc

# Multiple fields
GET /api/v1/products?sort=category,price
# or
GET /api/v1/products?sort[]=category&sort[]=price

# With direction
GET /api/v1/products?sort=category:asc,price:desc
```

### GraphQL Filtering

**Pattern**: Typed filter arguments

```graphql
enum SortOrder {
  ASC
  DESC
}

input ProductFilter {
  category: String
  priceMin: Float
  priceMax: Float
  status: ProductStatus
  search: String
}

input ProductSort {
  field: ProductSortField!
  order: SortOrder!
}

enum ProductSortField {
  NAME
  PRICE
  CREATED_AT
}

type Query {
  products(
    filter: ProductFilter
    sort: [ProductSort!]
    first: Int
    after: String
  ): ProductConnection!
}

# Usage
query {
  products(
    filter: {
      category: "electronics"
      priceMin: 100
      priceMax: 500
    }
    sort: [
      { field: PRICE, order: ASC }
    ]
    first: 20
  ) {
    edges {
      node {
        id
        name
        price
      }
    }
  }
}
```

---

## Error Response Patterns

### Standard Error Response (REST)

**Pattern**: Consistent error structure

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "statusCode": 400,
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Email must be a valid email address"
      },
      {
        "field": "password",
        "code": "TOO_SHORT",
        "message": "Password must be at least 8 characters"
      }
    ],
    "requestId": "req_abc123xyz"
  }
}
```

### Error Codes (REST)

**Pattern**: Machine-readable error codes

```json
// Authentication errors
{
  "error": {
    "code": "AUTHENTICATION_REQUIRED",
    "message": "Authentication is required to access this resource",
    "statusCode": 401
  }
}

// Authorization errors
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "You do not have permission to perform this action",
    "statusCode": 403
  }
}

// Resource not found
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "User with ID '123' not found",
    "resource": "User",
    "resourceId": "123",
    "statusCode": 404
  }
}

// Conflict
{
  "error": {
    "code": "RESOURCE_ALREADY_EXISTS",
    "message": "User with email 'john@example.com' already exists",
    "field": "email",
    "value": "john@example.com",
    "statusCode": 409
  }
}

// Rate limiting
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests, please try again later",
    "retryAfter": 60,
    "statusCode": 429
  }
}
```

### GraphQL Errors

**Pattern**: Errors in response structure

```json
{
  "data": {
    "createUser": {
      "user": null,
      "errors": [
        {
          "field": "email",
          "message": "Email already exists"
        }
      ]
    }
  },
  "errors": [
    {
      "message": "Database connection failed",
      "extensions": {
        "code": "DATABASE_ERROR",
        "serviceName": "user-service"
      },
      "path": ["createUser"]
    }
  ]
}
```

---

## Authentication Patterns

### JWT Bearer Token

**Pattern**: Stateless authentication

```http
POST /api/v1/auth/login
Content-Type: application/json
{
  "email": "john@example.com",
  "password": "secure123"
}

Response: 200 OK
{
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 3600,
    "tokenType": "Bearer"
  }
}

# Using the token
GET /api/v1/users/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Refresh token
POST /api/v1/auth/refresh
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

Response: 200 OK
{
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 3600
  }
}
```

### API Key Authentication

**Pattern**: Simple API key

```http
GET /api/v1/data
X-API-Key: sk_live_abc123xyz456

# or
Authorization: ApiKey sk_live_abc123xyz456
```

### OAuth 2.0 Flow

**Pattern**: Third-party authorization

```http
# 1. Authorization request
GET /oauth/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=https://app.example.com/callback&
  scope=read write&
  state=random_state

# 2. Exchange code for token
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTH_CODE&
client_id=CLIENT_ID&
client_secret=CLIENT_SECRET&
redirect_uri=https://app.example.com/callback

Response: 200 OK
{
  "access_token": "...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "...",
  "scope": "read write"
}
```

---

## Versioning Patterns

### URL Versioning

**Pattern**: Version in path (most common)

```http
GET /api/v1/users
GET /api/v2/users
```

### Header Versioning

**Pattern**: Version in custom header

```http
GET /api/users
Accept: application/vnd.example.v1+json

GET /api/users
API-Version: 2
```

### Content Negotiation

**Pattern**: Version in Accept header

```http
GET /api/users
Accept: application/vnd.example+json; version=1
```

### GraphQL Field Deprecation

**Pattern**: Gradual deprecation

```graphql
type User {
  # Deprecated field
  fullName: String @deprecated(reason: "Use firstName and lastName instead")

  # New fields
  firstName: String!
  lastName: String!
}
```

---

## Anti-Patterns to Avoid

### REST Anti-Patterns

```http
# ❌ Verbs in URLs
GET /api/getUsers
POST /api/createUser
DELETE /api/deleteUser

# ✅ Use HTTP methods properly
GET /api/users
POST /api/users
DELETE /api/users/{id}

# ❌ Exposing implementation details
GET /api/database/users_table

# ✅ Abstract implementation
GET /api/users

# ❌ Inconsistent naming
GET /api/users
GET /api/product
GET /api/order-items

# ✅ Consistent plural resources
GET /api/users
GET /api/products
GET /api/order-items

# ❌ Deeply nested URLs
GET /api/users/123/orders/456/items/789/reviews/012

# ✅ Flat structure with filtering
GET /api/reviews/012
GET /api/reviews?item_id=789

# ❌ Returning different structures
GET /api/users/{id}  → { "user": {...} }
GET /api/products/{id}  → { "data": {...} }

# ✅ Consistent wrapper
GET /api/users/{id}  → { "data": {...} }
GET /api/products/{id}  → { "data": {...} }
```

### GraphQL Anti-Patterns

```graphql
# ❌ Exposing database structure
type User {
  user_id: Int
  user_name: String
  created_ts: Int
}

# ✅ Use domain language
type User {
  id: ID!
  name: String!
  createdAt: DateTime!
}

# ❌ Too many arguments
type Query {
  users(
    name: String
    email: String
    status: String
    createdAfter: String
    createdBefore: String
    city: String
    country: String
  ): [User!]!
}

# ✅ Use input types
type Query {
  users(filter: UserFilter, sort: UserSort): [User!]!
}

input UserFilter {
  name: String
  email: String
  status: UserStatus
  createdBetween: DateRange
  location: LocationFilter
}

# ❌ No pagination
type Query {
  posts: [Post!]!  # Could return millions
}

# ✅ Always paginate lists
type Query {
  posts(first: Int, after: String): PostConnection!
}
```

### Security Anti-Patterns

```http
# ❌ Sequential IDs in URLs
GET /api/users/1
GET /api/users/2  # Easy to enumerate

# ✅ Use UUIDs or random IDs
GET /api/users/550e8400-e29b-41d4-a716-446655440000

# ❌ Sensitive data in URLs
GET /api/auth?api_key=secret123&password=pass123

# ✅ Use headers and request body
POST /api/auth
Authorization: Bearer secret123
{ "password": "pass123" }

# ❌ Verbose error messages
{
  "error": "SQL error: duplicate key in table users_pkey"
}

# ✅ Generic error messages
{
  "error": {
    "code": "RESOURCE_CONFLICT",
    "message": "User already exists"
  }
}
```
