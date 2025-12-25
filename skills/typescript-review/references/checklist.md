# TypeScript Code Review Checklist

This checklist provides a systematic framework for reviewing TypeScript code quality, type safety, and best practices.

## How to Use This Checklist

1. **Review each category** relevant to the code being analyzed
2. **Rate each item** (✅ Good, ⚠️ Needs improvement, ❌ Missing/Poor, N/A Not applicable)
3. **Note specific issues** with file paths and line numbers
4. **Prioritize improvements** based on impact (type safety > performance > style)
5. **Reference patterns.md** for detailed implementation guidance

---

## 1. Configuration & Setup

### tsconfig.json Review
- [ ] **Strict mode enabled**: Is `"strict": true` set?
- [ ] **Target version**: Is `target` appropriate for runtime environment?
- [ ] **Module system**: Is `module` correctly set (ESNext, CommonJS, etc.)?
- [ ] **Path aliases**: Are `paths` configured for cleaner imports?
- [ ] **Source maps**: Is `sourceMap: true` for debugging?
- [ ] **Declaration files**: Is `declaration: true` for libraries?
- [ ] **Lib configuration**: Are necessary libs included (ES2020, DOM, etc.)?
- [ ] **Module resolution**: Is `moduleResolution: "node"` or "bundler" set?

### Additional Strict Checks
- [ ] **noUnusedLocals**: Unused variables flagged?
- [ ] **noUnusedParameters**: Unused parameters flagged?
- [ ] **noImplicitReturns**: All code paths return values?
- [ ] **noFallthroughCasesInSwitch**: Switch cases have breaks?
- [ ] **noUncheckedIndexedAccess**: Array/object access safety?

**Impact**: High - Configuration sets foundation for type safety
**Priority**: Fix configuration issues before code review

---

## 2. Type Safety & Type Definitions

### any and unknown Usage
- [ ] **No implicit any**: Are all `any` types explicit and justified?
- [ ] **unknown over any**: Is `unknown` used when type is truly unknown?
- [ ] **Type assertions**: Are `as` assertions necessary and safe?
- [ ] **Non-null assertions**: Are `!` assertions guarded properly?
- [ ] **Any comments**: Are unavoidable `any` uses documented?

### Function Type Safety
- [ ] **Return types**: Do public functions have explicit return types?
- [ ] **Parameter types**: Are all parameters explicitly typed?
- [ ] **Arrow functions**: Are complex arrow functions properly typed?
- [ ] **Async functions**: Are Promise return types explicit?
- [ ] **Callback types**: Are callback parameters properly typed?

### Interface and Type Definitions
- [ ] **Interface vs Type**: Are interfaces used for objects, types for unions?
- [ ] **Readonly properties**: Are immutable fields marked `readonly`?
- [ ] **Optional properties**: Is `?` used appropriately?
- [ ] **Index signatures**: Are index signatures necessary and typed?
- [ ] **Extends vs intersections**: Is the right composition method used?

### Generic Types
- [ ] **Generic constraints**: Do generics have proper `extends` constraints?
- [ ] **Default type parameters**: Are defaults provided where sensible?
- [ ] **Type parameter naming**: Are names descriptive (not just T, U, V)?
- [ ] **Variance**: Are generic positions used correctly?

**Impact**: Critical - Type safety is TypeScript's primary value
**Priority**: Address type safety issues immediately

---

## 3. Modern TypeScript Features

### Type System Features
- [ ] **Template literals**: Are string types using template literal types?
- [ ] **Const assertions**: Is `as const` used for literal types?
- [ ] **Satisfies operator**: Is `satisfies` used for type validation?
- [ ] **Utility types**: Are built-in utilities (Pick, Omit, etc.) leveraged?
- [ ] **Discriminated unions**: Are tagged unions used for state management?

### Type Guards and Narrowing
- [ ] **Type predicates**: Are custom type guards using `is` keyword?
- [ ] **typeof guards**: Are primitive type checks properly used?
- [ ] **instanceof guards**: Are class instance checks used correctly?
- [ ] **in operator**: Are object property checks type-safe?
- [ ] **Truthiness checks**: Are nullish checks using `??` and `?.`?

### Advanced Patterns
- [ ] **Mapped types**: Are object transformations using mapped types?
- [ ] **Conditional types**: Are complex type computations justified?
- [ ] **Recursive types**: Are recursive types properly constrained?
- [ ] **Branded types**: Are nominal types used where needed?

**Impact**: Medium - Improves code quality and maintainability
**Priority**: Address after type safety issues

---

## 4. Error Handling

### Exception Handling
- [ ] **Try-catch blocks**: Are errors caught and typed appropriately?
- [ ] **Error types**: Are custom error classes properly typed?
- [ ] **Unknown errors**: Are catch clauses using `unknown` not `any`?
- [ ] **Error propagation**: Are errors re-thrown with context?
- [ ] **Async errors**: Are Promise rejections handled?

### Result Types
- [ ] **Result pattern**: Are Result<T, E> types used instead of throwing?
- [ ] **Optional types**: Is `| null` or `| undefined` used explicitly?
- [ ] **Error unions**: Are error cases part of return types?

**Impact**: High - Affects runtime reliability
**Priority**: Critical for production code

---

## 5. Async/Await and Promises

### Promise Handling
- [ ] **Await usage**: Are all Promises properly awaited?
- [ ] **Promise types**: Are Promise<T> types explicit?
- [ ] **Error handling**: Are async errors caught?
- [ ] **Promise.all**: Are parallel operations properly typed?
- [ ] **Void promises**: Are fire-and-forget promises intentional?

### Async Patterns
- [ ] **Race conditions**: Are concurrent operations safe?
- [ ] **Async iteration**: Is `for await...of` used correctly?
- [ ] **Cancellation**: Are long-running operations cancellable?

**Impact**: High - Affects correctness and performance
**Priority**: Review carefully for backend code

---

## 6. Object-Oriented Programming

### Class Design
- [ ] **Access modifiers**: Are `private`, `protected`, `public` used appropriately?
- [ ] **Readonly fields**: Are immutable fields marked `readonly`?
- [ ] **Constructor parameters**: Are shorthand properties used?
- [ ] **Method return types**: Do all methods have explicit return types?
- [ ] **Abstract classes**: Are base classes properly abstracted?

### Inheritance and Composition
- [ ] **Extends usage**: Is inheritance appropriate or should composition be used?
- [ ] **Interface implementation**: Are `implements` clauses complete?
- [ ] **Method overrides**: Are overridden methods properly typed?
- [ ] **Super calls**: Are parent constructors called correctly?

**Impact**: Medium - Affects maintainability
**Priority**: Address for OOP-heavy codebases

---

## 7. Module System and Imports

### Import/Export Patterns
- [ ] **Named exports**: Are named exports preferred over default?
- [ ] **Type imports**: Is `import type` used for type-only imports?
- [ ] **Barrel files**: Are index.ts files used appropriately?
- [ ] **Circular dependencies**: Are circular imports avoided?
- [ ] **Path aliases**: Are `@/` style imports used consistently?

### Module Organization
- [ ] **Feature modules**: Is code organized by feature?
- [ ] **Type files**: Are large type definitions in separate .d.ts files?
- [ ] **Shared types**: Are common types in a shared location?

**Impact**: Medium - Affects code organization
**Priority**: Address during refactoring

---

## 8. Performance and Compilation

### Type Computation Performance
- [ ] **Complex types**: Are deeply nested types necessary?
- [ ] **Type caching**: Are computed types aliased?
- [ ] **Union size**: Are large unions (>50 members) necessary?
- [ ] **Recursive depth**: Are recursive types limited?

### Build Performance
- [ ] **Incremental builds**: Is `incremental: true` enabled?
- [ ] **Project references**: Are large projects split?
- [ ] **Skip lib check**: Is `skipLibCheck: true` used appropriately?
- [ ] **Watch mode**: Does watch mode perform well?

**Impact**: Medium - Affects developer experience
**Priority**: Address if compilation is slow (>10s)

---

## 9. Testing

### Test Type Safety
- [ ] **Test typing**: Are test functions properly typed?
- [ ] **Mock typing**: Are mocks type-safe?
- [ ] **Test data**: Are test fixtures properly typed?
- [ ] **Type assertions**: Are runtime assertions matched with types?

### Type Testing
- [ ] **Type tests**: Are complex types unit tested?
- [ ] **dtslint/tsd**: Are type tests automated?
- [ ] **Breaking changes**: Are type changes validated?

**Impact**: Medium - Prevents type regressions
**Priority**: Implement for libraries and shared code

---

## 10. Documentation

### Code Documentation
- [ ] **JSDoc comments**: Are public APIs documented?
- [ ] **Type annotations**: Are complex types explained?
- [ ] **Examples**: Are usage examples provided?
- [ ] **Deprecations**: Are `@deprecated` tags used?

### Type Documentation
- [ ] **Generic parameters**: Are type parameters documented?
- [ ] **Constraints**: Are type constraints explained?
- [ ] **Edge cases**: Are type edge cases documented?

**Impact**: Low - Affects maintainability over time
**Priority**: Address for public APIs and complex code

---

## Scoring and Prioritization

### Issue Severity

**Critical (Fix immediately):**
- `any` usage without justification
- Missing error handling in production code
- Non-null assertions without guards
- Disabled strict mode

**High (Fix before merge):**
- Missing return types on public functions
- Unsafe type assertions
- Implicit any in parameters
- Unhandled Promise rejections

**Medium (Fix in near term):**
- Missing utility type usage
- Poor module organization
- Inefficient type computations
- Missing type guards

**Low (Nice to have):**
- Missing JSDoc comments
- Inconsistent naming
- Minor style issues
- Could use newer TS features

### Review Process

1. **Quick scan** (5 min): Check configuration and critical issues
2. **Deep review** (20-30 min): Full checklist coverage
3. **Recommendations** (10 min): Prioritized list with examples
4. **Discussion** (15 min): Review findings with team

**Total time**: ~1 hour for thorough review of medium-sized PR
