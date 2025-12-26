# API Design Checklist

This checklist provides a systematic framework for designing and reviewing REST and GraphQL APIs.

## How to Use This Checklist

1. **Review each category** relevant to your API type (REST/GraphQL)
2. **Rate each item** (✅ Good, ⚠️ Needs improvement, ❌ Missing/Poor, N/A Not applicable)
3. **Note specific issues** with endpoint paths or schema locations
4. **Prioritize improvements** based on impact (security > correctness > performance > style)
5. **Reference patterns.md** for detailed implementation examples

---

## 1. Resource Design & Naming (REST)

### URL Structure
- [ ] **Nouns not verbs**: Are resources named as nouns (`/users` not `/getUsers`)?
- [ ] **Plural collections**: Are collection resources plural (`/products`)?
- [ ] **Kebab-case**: Are multi-word resources using kebab-case (`/order-items`)?
- [ ] **Hierarchy**: Do nested resources follow logical hierarchy (`/users/{id}/orders`)?
- [ ] **Depth limit**: Are URLs kept to 3 levels or less?
- [ ] **No trailing slashes**: Are endpoints consistent about trailing slashes?

### Resource Naming Consistency
- [ ] **Consistent naming**: Are similar resources named consistently?
- [ ] **No abbreviations**: Are abbreviations avoided unless universal?
- [ ] **Predictable**: Can developers guess endpoint names?
- [ ] **Domain language**: Do names match business domain?

**Impact**: High - Foundation of API usability
**Priority**: Must be correct before launch

---

## 2. HTTP Methods & Semantics (REST)

### Method Usage
- [ ] **GET for retrieval**: Are GET requests used only for reading?
- [ ] **POST for creation**: Are POST requests used for creating resources?
- [ ] **PUT for replacement**: Are PUT requests replacing entire resources?
- [ ] **PATCH for updates**: Are PATCH requests for partial updates?
- [ ] **DELETE for removal**: Are DELETE requests removing resources?
- [ ] **Idempotency**: Are PUT, PATCH, DELETE, and GET idempotent?

### Safety and Idempotency
- [ ] **Safe GETs**: Do GET requests have no side effects?
- [ ] **Idempotent operations**: Can safe operations be repeated without different results?
- [ ] **POST non-idempotent**: Does calling POST multiple times create multiple resources?

**Impact**: Critical - Affects correctness and caching
**Priority**: Must follow HTTP semantics

---

## 3. Status Codes (REST)

### Success Codes
- [ ] **200 OK**: Used for successful GET, PUT, PATCH, DELETE?
- [ ] **201 Created**: Used for successful POST with resource creation?
- [ ] **204 No Content**: Used for successful DELETE or update with no response body?
- [ ] **202 Accepted**: Used for async operations?

### Client Error Codes
- [ ] **400 Bad Request**: Used for validation errors?
- [ ] **401 Unauthorized**: Used for missing/invalid authentication?
- [ ] **403 Forbidden**: Used for authorization failures?
- [ ] **404 Not Found**: Used for missing resources?
- [ ] **409 Conflict**: Used for resource conflicts (duplicate, version mismatch)?
- [ ] **422 Unprocessable Entity**: Used for semantic validation errors?
- [ ] **429 Too Many Requests**: Used for rate limiting?

### Server Error Codes
- [ ] **500 Internal Server Error**: Used for unexpected server errors?
- [ ] **503 Service Unavailable**: Used for temporary outages?

### Code Consistency
- [ ] **Appropriate codes**: Are the most specific status codes used?
- [ ] **Consistent usage**: Are codes used consistently across endpoints?
- [ ] **Documentation**: Are status codes documented?

**Impact**: High - Affects client error handling
**Priority**: Critical for production APIs

---

## 4. Request Design

### Request Body
- [ ] **JSON format**: Is JSON used for request bodies?
- [ ] **Consistent structure**: Do requests follow consistent patterns?
- [ ] **Required fields**: Are required fields clearly defined?
- [ ] **Validation**: Are validation rules documented?
- [ ] **Size limits**: Are payload size limits enforced?

### Query Parameters
- [ ] **Filtering**: Are filter parameters consistent (`?status=active`)?
- [ ] **Sorting**: Is sorting parameter consistent (`?sort=created_at&order=desc`)?
- [ ] **Pagination**: Is pagination standardized (`?page=1&limit=20` or `?offset=0&limit=20`)?
- [ ] **Field selection**: Can clients request specific fields (`?fields=id,name,email`)?
- [ ] **Search**: Is search parameter consistent (`?q=query`)?

### Path Parameters
- [ ] **Resource IDs**: Are IDs using UUIDs or non-sequential IDs?
- [ ] **Type consistency**: Are similar IDs the same type across endpoints?
- [ ] **Validation**: Are path parameters validated?

**Impact**: Medium - Affects API usability
**Priority**: Standardize before adding many endpoints

---

## 5. Response Design

### Response Structure
- [ ] **Consistent format**: Do all responses follow the same structure?
- [ ] **Data wrapper**: Is response data wrapped in a `data` field?
- [ ] **Metadata**: Are timestamps, pagination info included in `meta`?
- [ ] **Null handling**: Are null values handled consistently?
- [ ] **Empty arrays**: Are empty collections returned as `[]` not `null`?

### Response Fields
- [ ] **Camel case**: Are field names in camelCase?
- [ ] **ISO 8601 dates**: Are dates in ISO 8601 format (`2024-01-15T10:30:00Z`)?
- [ ] **Numeric IDs**: Are IDs strings, not numbers (to avoid overflow)?
- [ ] **Boolean values**: Are booleans actual boolean, not strings/numbers?

### Pagination
- [ ] **Consistent pagination**: Is pagination format consistent?
- [ ] **Total count**: Is total count included?
- [ ] **Page info**: Are next/previous links or cursors provided?
- [ ] **Default limit**: Is there a sensible default page size?
- [ ] **Max limit**: Is there a maximum page size enforced?

**Impact**: High - Affects all API consumers
**Priority**: Establish early and maintain consistency

---

## 6. Error Handling

### Error Response Format
- [ ] **Consistent structure**: Do all errors follow the same format?
- [ ] **Error code**: Is a machine-readable error code included?
- [ ] **Error message**: Is a human-readable message provided?
- [ ] **Field errors**: Are validation errors mapped to specific fields?
- [ ] **Request ID**: Is a request/trace ID included for debugging?

### Error Messages
- [ ] **Clear messages**: Are error messages clear and actionable?
- [ ] **No sensitive data**: Do errors avoid exposing sensitive information?
- [ ] **Localization**: Are errors localizable?
- [ ] **Consistency**: Are similar errors formatted consistently?

### Error Documentation
- [ ] **All errors documented**: Are all possible error codes documented?
- [ ] **Examples**: Are error response examples provided?
- [ ] **Recovery**: Is error recovery guidance provided?

**Impact**: High - Critical for developer experience
**Priority**: Must be comprehensive and consistent

---

## 7. Authentication & Authorization

### Authentication
- [ ] **HTTPS only**: Are all endpoints HTTPS only?
- [ ] **Standard auth**: Is OAuth 2.0, JWT, or API keys used?
- [ ] **Token format**: Are tokens in Authorization header (`Bearer {token}`)?
- [ ] **Token validation**: Are tokens properly validated?
- [ ] **Token expiration**: Do tokens have expiration times?
- [ ] **Refresh tokens**: Is token refresh implemented?

### Authorization
- [ ] **Permission checks**: Are permissions checked on every request?
- [ ] **Resource-level**: Is authorization checked at resource level?
- [ ] **Fail secure**: Does authorization fail closed?
- [ ] **Clear errors**: Do auth failures return 401/403 appropriately?

### Security Headers
- [ ] **CORS**: Are CORS headers properly configured?
- [ ] **Content-Type**: Is Content-Type validated?
- [ ] **Rate limiting**: Is rate limiting implemented?
- [ ] **API keys**: Are API keys not passed in URLs?

**Impact**: Critical - Security is paramount
**Priority**: Must be correct from the start

---

## 8. GraphQL Schema Design

### Type Design
- [ ] **Clear names**: Are type names descriptive and unambiguous?
- [ ] **Domain modeling**: Do types match business domain?
- [ ] **Interfaces**: Are interfaces used for shared fields?
- [ ] **Unions**: Are unions used for polymorphic types?
- [ ] **Nullability**: Is nullability explicit and appropriate?

### Field Design
- [ ] **Naming**: Are field names camelCase and descriptive?
- [ ] **Arguments**: Do fields have properly typed arguments?
- [ ] **Deprecation**: Are deprecated fields marked with `@deprecated`?
- [ ] **Descriptions**: Are types and fields documented?
- [ ] **Return types**: Are return types as specific as possible?

### Query Design
- [ ] **Filtering**: Can queries be filtered?
- [ ] **Sorting**: Can results be sorted?
- [ ] **Pagination**: Is cursor-based pagination implemented?
- [ ] **Connections**: Are lists using the Connection pattern?
- [ ] **Depth limiting**: Are query depth limits enforced?

### Mutation Design
- [ ] **Naming**: Are mutations named as verb-object (`createUser`)?
- [ ] **Input types**: Do mutations use input types?
- [ ] **Payload types**: Do mutations return payload types with errors?
- [ ] **Return data**: Do mutations return the modified object?
- [ ] **Idempotency**: Are mutations idempotent where possible?

**Impact**: High - Core of GraphQL API design
**Priority**: Get schema design right early

---

## 9. Performance

### Query Optimization
- [ ] **N+1 prevention**: Is DataLoader or similar used?
- [ ] **Batch loading**: Are related entities batched?
- [ ] **Query complexity**: Is query complexity limited?
- [ ] **Timeouts**: Are query timeouts configured?

### Caching
- [ ] **Cache headers**: Are appropriate Cache-Control headers set?
- [ ] **ETags**: Are ETags used for conditional requests?
- [ ] **Vary header**: Is Vary header set appropriately?
- [ ] **Cache invalidation**: Is cache invalidation strategy defined?

### Response Size
- [ ] **Field selection**: Can clients select specific fields?
- [ ] **Compression**: Is response compression enabled (gzip, brotli)?
- [ ] **Pagination**: Are large lists paginated?
- [ ] **Lazy loading**: Can expensive fields be loaded on demand?

### Rate Limiting
- [ ] **Rate limits**: Are rate limits implemented per user/IP?
- [ ] **Limit headers**: Are rate limit headers returned?
- [ ] **Gradual limits**: Are limits enforced gradually (not hard cutoff)?
- [ ] **Documented limits**: Are rate limits documented?

**Impact**: High - Affects scalability and cost
**Priority**: Implement before public launch

---

## 10. Versioning

### Versioning Strategy
- [ ] **Version scheme**: Is versioning strategy chosen (URL, header, content negotiation)?
- [ ] **Clear versions**: Are version numbers semantic (v1, v2)?
- [ ] **Version documentation**: Is each version documented?
- [ ] **Default version**: Is there a default version?

### Breaking Changes
- [ ] **Change policy**: Is breaking change policy defined?
- [ ] **Deprecation**: Are deprecated features clearly marked?
- [ ] **Sunset dates**: Are deprecation timelines communicated?
- [ ] **Migration guide**: Are migration guides provided?

### Backward Compatibility
- [ ] **Additive changes**: Are new features additive?
- [ ] **Optional fields**: Are new fields optional?
- [ ] **Default values**: Do new fields have sensible defaults?
- [ ] **No removal**: Are fields never removed, only deprecated?

**Impact**: Critical - Affects all API consumers
**Priority**: Define strategy before first release

---

## 11. Documentation

### API Documentation
- [ ] **OpenAPI/GraphQL**: Is there machine-readable spec (OpenAPI, GraphQL schema)?
- [ ] **Interactive docs**: Is Swagger UI/GraphQL Playground available?
- [ ] **Getting started**: Is there a quick start guide?
- [ ] **Examples**: Are request/response examples provided?
- [ ] **Code samples**: Are code samples in multiple languages provided?

### Endpoint Documentation
- [ ] **All endpoints**: Is every endpoint documented?
- [ ] **Parameters**: Are all parameters described?
- [ ] **Responses**: Are all response codes documented?
- [ ] **Errors**: Are all error scenarios documented?
- [ ] **Authentication**: Is auth process clearly explained?

### Supporting Documentation
- [ ] **Changelog**: Is there a maintained changelog?
- [ ] **Migration guides**: Are version migration guides available?
- [ ] **Use cases**: Are common use cases documented?
- [ ] **Best practices**: Are API best practices shared?

**Impact**: High - Critical for adoption
**Priority**: Must be comprehensive at launch

---

## 12. Testing

### Test Coverage
- [ ] **Unit tests**: Are API handlers unit tested?
- [ ] **Integration tests**: Are endpoints integration tested?
- [ ] **Contract tests**: Are API contracts tested?
- [ ] **Error cases**: Are error scenarios tested?

### Test Documentation
- [ ] **Test examples**: Are example tests provided?
- [ ] **Postman/Insomnia**: Are collections available?
- [ ] **Mock server**: Is a mock server available for testing?

**Impact**: Medium - Ensures reliability
**Priority**: Implement before production release

---

## Scoring and Prioritization

### Issue Severity

**Critical (Block release):**
- Security vulnerabilities
- Missing authentication
- Incorrect HTTP semantics
- Data exposure risks
- No rate limiting

**High (Fix before public launch):**
- Inconsistent naming
- Missing error handling
- Poor status code usage
- Missing documentation
- No versioning strategy

**Medium (Fix soon):**
- Missing pagination
- No caching strategy
- Suboptimal performance
- Incomplete documentation
- Missing filtering/sorting

**Low (Nice to have):**
- Additional convenience features
- Better examples
- More code samples
- Minor inconsistencies

### Review Process

1. **Security review** (30 min): Authentication, authorization, input validation
2. **Design review** (45 min): Resource design, consistency, HTTP semantics
3. **Documentation review** (30 min): Completeness, accuracy, examples
4. **Performance review** (20 min): Caching, rate limiting, pagination

**Total time**: ~2 hours for thorough API review
