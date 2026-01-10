---
name: reviewing-typescript
description: "Review and improve TypeScript code quality, type safety, and best practices. Use when users request: (1) TypeScript code review, (2) Type safety improvements, (3) Best practice validation, (4) Performance optimization for TypeScript code, (5) Refactoring suggestions, or (6) Migration from JavaScript to TypeScript. Applies TypeScript-specific patterns, strict mode principles, and modern language features."
---

# TypeScript Review

## Core Review Workflow

When reviewing TypeScript code:

1. **Understand the Context**
   - Identify the type of code (frontend/backend, library/application)
   - Check tsconfig.json settings and compiler options
   - Understand the project's TypeScript version and target environment
   - Review existing type definitions and interfaces

2. **Type Safety Analysis**
   - Identify `any`, `unknown`, and type assertions usage
   - Check for implicit any and missing return types
   - Validate generic constraints and type parameters
   - Review union/intersection type usage
   - Assess discriminated unions and type guards

3. **Code Quality Review**
   - Evaluate code organization and module structure
   - Check for code duplication and reusability
   - Review error handling patterns
   - Assess async/await usage and Promise handling
   - Validate naming conventions and consistency

4. **Generate Prioritized Recommendations**
   - List specific, actionable improvements
   - Prioritize by impact (type safety > performance > style)
   - Provide code examples for complex changes
   - Reference TypeScript best practices and patterns

## Quick Reference: Key Review Areas

### Type Safety (Highest Priority)

**Critical Issues:**
- `any` type usage without justification
- Type assertions (`as`) that bypass type checking
- Missing return type annotations on public functions
- Implicit any in function parameters
- Non-null assertions (`!`) without proper guards

**Best Practices:**
- Enable strict mode in tsconfig.json
- Use `unknown` instead of `any` when type is truly unknown
- Prefer type inference for local variables
- Use discriminated unions for complex state
- Implement proper type guards

### Modern TypeScript Features

**Recommended Patterns:**
- Template literal types for string manipulation
- `const` assertions for literal types
- Utility types (Partial, Pick, Omit, Record, etc.)
- Conditional types for advanced scenarios
- `satisfies` operator for type validation

**Advanced Features:**
- Mapped types for transformations
- Recursive types for nested structures
- Branded types for nominal typing
- Type predicates for narrowing

### Performance Considerations

**Common Issues:**
- Excessive type computations slowing compilation
- Circular type dependencies
- Over-use of complex conditional types
- Large union types causing slow IntelliSense

**Optimizations:**
- Use type aliases to simplify complex types
- Avoid deep type recursion
- Cache computed types
- Use interface merging strategically

### Code Organization

**Module Structure:**
- Proper use of import/export
- Barrel exports (index.ts) for clean APIs
- Separation of types and implementation
- Avoiding circular dependencies

**File Organization:**
- Co-locate types with implementation when appropriate
- Use separate .d.ts files for large type definitions
- Organize by feature, not by file type

## Common Use Cases

### Frontend Code Review

For React, Vue, or Angular TypeScript code:
- Component prop type definitions
- State management type safety (Redux, Zustand, Pinia)
- Event handler typing
- API response type definitions
- Form validation and type safety

### Backend Code Review

For Node.js, Express, NestJS code:
- Request/response typing
- Database model type definitions
- Service layer type safety
- Middleware typing
- Error handling patterns

### Library/Package Development

For publishable TypeScript packages:
- Public API type definitions
- Generic type parameters and constraints
- Declaration file (.d.ts) quality
- Breaking change detection
- Type compatibility versioning

### Migration Review

For JavaScript to TypeScript migrations:
- Incremental migration strategy
- Type coverage progression
- Third-party library types (@types packages)
- Type assertion cleanup
- Configuration optimization

## Review Checklist

Use `references/checklist.md` for systematic code review covering:
- tsconfig.json configuration
- Type safety and strict mode compliance
- Modern TypeScript features usage
- Performance and compilation speed
- Code organization and maintainability
- Testing and type testing
- Documentation and JSDoc comments

## Resources

### references/patterns.md

Comprehensive guide to TypeScript patterns including:
- Type-safe builder patterns
- Factory patterns with generics
- Repository patterns with type constraints
- Dependency injection typing
- Event emitter typing
- Real-world examples and anti-patterns

### references/checklist.md

Structured evaluation framework:
- Configuration review criteria
- Type safety scoring
- Performance assessment
- Best practice validation
- Common issues and solutions

## Strict Mode Guidelines

When `strict: true` is enabled in tsconfig.json, these checks are enforced:

- `noImplicitAny`: All parameters and variables must have explicit or inferred types
- `strictNullChecks`: null and undefined must be handled explicitly
- `strictFunctionTypes`: Function parameter types are checked contravariantly
- `strictBindCallApply`: bind/call/apply methods are type-checked
- `strictPropertyInitialization`: Class properties must be initialized
- `noImplicitThis`: this must have explicit type in functions
- `alwaysStrict`: Files parsed in strict mode with "use strict"

## Anti-Patterns to Avoid

### Type Safety Anti-Patterns
- Using `any` to bypass type errors
- Excessive use of type assertions
- Ignoring TypeScript errors with `@ts-ignore`
- Not enabling strict mode
- Using `Object` or `{}` as general types

### Performance Anti-Patterns
- Deep recursive types without limits
- Excessive use of complex conditional types
- Large discriminated unions (>50 members)
- Unnecessary type re-computation

### Organization Anti-Patterns
- Mixing types and implementation in large files
- Global type pollution
- Circular dependencies between modules
- Not using path aliases for deep imports

## Ethical Considerations

When reviewing code:
- **Be Constructive**: Focus on improvement, not criticism
- **Explain Why**: Always provide reasoning for suggestions
- **Respect Context**: Consider project constraints and deadlines
- **Share Knowledge**: Use reviews as teaching opportunities
- **Prioritize**: Don't overwhelm with minor issues when major problems exist
