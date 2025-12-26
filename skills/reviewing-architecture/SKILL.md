---
name: reviewing-architecture
description: "Review and design software architecture for scalability, maintainability, and best practices. Use when users request: (1) Architecture review or design, (2) System design decisions, (3) Technology stack selection, (4) Scalability planning, (5) Microservices vs monolith decisions, or (6) Design pattern recommendations. Applies architectural patterns, SOLID principles, and industry best practices."
---

# Architecture Review

## Core Review Workflow

When reviewing or designing architecture:

1. **Understand Requirements**
   - Identify business requirements and constraints
   - Understand scale requirements (users, data volume, traffic)
   - Determine performance and reliability requirements
   - Review budget and resource constraints
   - Consider team size and expertise

2. **Analyze Current Architecture**
   - Review system components and their interactions
   - Identify architectural patterns in use
   - Assess data flow and communication patterns
   - Evaluate separation of concerns
   - Check for architectural anti-patterns

3. **Evaluate Architecture Quality**
   - Assess scalability and performance
   - Review security architecture
   - Evaluate maintainability and testability
   - Check deployment and operational concerns
   - Analyze technical debt and risks

4. **Generate Recommendations**
   - Prioritize improvements by impact and effort
   - Suggest specific architectural patterns
   - Recommend technology stack changes if needed
   - Provide migration strategies for changes
   - Document trade-offs and decisions

## Quick Reference: Architectural Principles

### SOLID Principles

- **Single Responsibility**: Each module should have one reason to change
- **Open/Closed**: Open for extension, closed for modification
- **Liskov Substitution**: Subtypes should be substitutable for base types
- **Interface Segregation**: Many specific interfaces better than one general
- **Dependency Inversion**: Depend on abstractions, not concretions

### Key Architectural Qualities

**Scalability:**
- Horizontal scaling capability
- Stateless design where possible
- Efficient resource utilization
- Load balancing and distribution

**Maintainability:**
- Clear separation of concerns
- Modular design
- Consistent patterns
- Good documentation

**Performance:**
- Efficient algorithms and data structures
- Appropriate caching strategies
- Optimized database queries
- Async processing where appropriate

**Security:**
- Defense in depth
- Least privilege principle
- Input validation
- Secure by default

**Reliability:**
- Fault tolerance
- Graceful degradation
- Monitoring and observability
- Disaster recovery plan

## Common Use Cases

### Monolith vs Microservices

For deciding architecture style:
- Team size and maturity
- System complexity
- Deployment requirements
- Scaling needs
- Organizational structure

### Layered Architecture

For organizing application layers:
- Presentation layer (UI)
- Application layer (business logic)
- Domain layer (core business rules)
- Data access layer (persistence)
- Infrastructure layer (external services)

### Event-Driven Architecture

For asynchronous systems:
- Decoupled components
- Event sourcing
- CQRS pattern
- Message queues
- Event streaming

### Cloud Architecture

For cloud-native applications:
- Containerization strategy
- Orchestration (Kubernetes)
- Serverless vs containers
- Cloud service selection
- Multi-cloud vs single cloud

## Architecture Patterns

### Common Patterns

**MVC/MVVM** (Frontend)
- Separation of concerns for UI
- Clear data flow
- Testable components

**Repository Pattern** (Data Access)
- Abstract data access
- Testable business logic
- Flexible data sources

**Service Layer** (Business Logic)
- Encapsulate business logic
- Transaction boundaries
- Reusable operations

**API Gateway** (Microservices)
- Single entry point
- Request routing
- Authentication/authorization
- Rate limiting

**CQRS** (Complex Domains)
- Separate read and write models
- Optimized for different operations
- Eventual consistency

**Event Sourcing** (Audit/History)
- Store events, not state
- Full audit trail
- Time travel debugging
- Replay events

## Technology Stack Considerations

### Frontend

**Framework Selection:**
- React: Component-based, large ecosystem
- Vue: Progressive, easier learning curve
- Angular: Full-featured, enterprise-ready
- Svelte: Compiled, minimal runtime

**State Management:**
- Context API (React built-in)
- Redux (predictable state)
- Zustand (lightweight)
- Jotai/Recoil (atomic state)

### Backend

**Runtime:**
- Node.js: JavaScript, event-driven, large ecosystem
- Python: AI/ML, data processing, rapid development
- Go: Performance, concurrency, cloud-native
- Java: Enterprise, mature ecosystem

**Framework:**
- Express (Node.js, minimal)
- NestJS (Node.js, structured, TypeScript-first)
- FastAPI (Python, async, auto-docs)
- Spring Boot (Java, enterprise)

### Database

**Relational (SQL):**
- PostgreSQL: Feature-rich, ACID, JSON support
- MySQL: Popular, simple, widely supported
- SQLite: Embedded, serverless

**Document (NoSQL):**
- MongoDB: Flexible schema, JSON documents
- DynamoDB: Managed, serverless, scalable

**In-Memory:**
- Redis: Cache, session store, pub/sub
- Memcached: Simple caching

### Infrastructure

**Containerization:**
- Docker: Standard containerization
- Docker Compose: Local multi-container
- Kubernetes: Production orchestration

**Cloud Providers:**
- AWS: Comprehensive, mature
- Google Cloud: AI/ML, Kubernetes
- Azure: Enterprise, Microsoft integration
- Vercel/Netlify: Frontend-focused

## Review Checklist

Use `references/checklist.md` for comprehensive architecture review covering:
- Overall architecture design
- Component organization
- Data architecture
- Security architecture
- Performance and scalability
- Deployment and operations
- Technology stack appropriateness

## Resources

### references/patterns.md

Comprehensive guide to architectural patterns:
- Layered architecture patterns
- Microservices patterns
- Event-driven patterns
- Data architecture patterns
- Caching patterns
- Security patterns
- Real-world examples and trade-offs

### references/checklist.md

Structured evaluation framework:
- Architecture quality assessment
- Pattern identification
- Scalability analysis
- Security review
- Technology stack evaluation

## Decision Framework

### Choosing Monolith vs Microservices

**Choose Monolith when:**
- Small team (< 10 developers)
- Early stage product
- Simple domain
- Tight coupling makes sense
- Fast iteration needed

**Choose Microservices when:**
- Large team (multiple teams)
- Complex domain with clear boundaries
- Independent scaling needed
- Different technology stacks required
- Organizational autonomy needed

### Database Selection

**Relational (SQL) when:**
- Complex relationships
- ACID transactions required
- Strong consistency needed
- Complex queries
- Mature ecosystem needed

**Document (NoSQL) when:**
- Flexible schema needed
- Horizontal scaling required
- Simple key-value lookups
- Eventual consistency acceptable
- High write throughput

### Caching Strategy

**Client-side caching:**
- Static assets (CDN)
- API responses (HTTP cache)
- Application state (LocalStorage)

**Server-side caching:**
- Database query results (Redis)
- Computed values (in-memory)
- Session data (Redis/Memcached)

**CDN caching:**
- Static files
- Edge-computed content
- API responses (with care)

## Scalability Patterns

### Horizontal Scaling

- Stateless application design
- Load balancing (round-robin, least connections)
- Session management (centralized)
- Shared nothing architecture

### Vertical Scaling

- Increase server resources
- Optimize resource usage
- Limit: hardware constraints

### Database Scaling

**Read Replicas:**
- Separate read and write traffic
- Eventual consistency
- Load distribution

**Sharding:**
- Partition data by key
- Horizontal partitioning
- Complex queries harder

**Caching:**
- Reduce database load
- Faster response times
- Cache invalidation strategy

## Security Architecture

### Defense in Depth

**Application Layer:**
- Input validation
- Output encoding
- Authentication/authorization
- Rate limiting

**Network Layer:**
- Firewall rules
- VPN for internal services
- DDoS protection
- SSL/TLS encryption

**Data Layer:**
- Encryption at rest
- Encryption in transit
- Access controls
- Backup encryption

### Authentication & Authorization

**Authentication:**
- JWT tokens (stateless)
- Session-based (stateful)
- OAuth 2.0 (third-party)
- Multi-factor authentication

**Authorization:**
- Role-based access control (RBAC)
- Attribute-based access control (ABAC)
- Permission-based
- Resource-level permissions

## Deployment Architecture

### Deployment Strategies

**Blue-Green Deployment:**
- Two identical environments
- Instant rollback
- Zero downtime

**Canary Deployment:**
- Gradual rollout
- Test with subset of users
- Monitor and expand

**Rolling Deployment:**
- Update instances incrementally
- Always some instances running
- Slower than blue-green

**Feature Flags:**
- Deploy code, enable features separately
- A/B testing
- Gradual feature rollout

### CI/CD Pipeline

**Stages:**
1. Source control (Git)
2. Build and test
3. Security scanning
4. Staging deployment
5. Integration tests
6. Production deployment
7. Monitoring

## Anti-Patterns to Avoid

**Big Ball of Mud:**
- Lack of clear architecture
- Tight coupling everywhere
- No separation of concerns

**Golden Hammer:**
- Using same solution for every problem
- "I know MongoDB, so use it everywhere"
- Not evaluating alternatives

**Premature Optimization:**
- Optimizing before measuring
- Complex solutions for simple problems
- Over-engineering

**Tight Coupling:**
- Changes in one module affect many others
- Hard to test independently
- Difficult to replace components

**Distributed Monolith:**
- Microservices with tight coupling
- Shared database
- Synchronized deployments needed

**Not Invented Here:**
- Rebuilding existing solutions
- Ignoring proven libraries/frameworks
- Wasting time on solved problems

## Ethical Considerations

When designing architecture:
- **Privacy by Design**: Build privacy into the architecture
- **Data Minimization**: Collect only necessary data
- **Security First**: Don't compromise on security for speed
- **Accessibility**: Design for inclusive access
- **Environmental Impact**: Consider energy efficiency
- **Vendor Lock-in**: Evaluate long-term flexibility
- **Open Standards**: Prefer open protocols and formats
