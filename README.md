# Custom Linters Documentation

## Overview
This document explains the custom linting rules implemented in our React Native project. These linters use Abstract Syntax Tree (AST) analysis to enforce code quality, type safety, and best practices specific to our codebase.

## Table of Contents
1. [Introduction](#introduction)
2. [Available Rules](#available-rules)
3. [AST Implementation](#ast-implementation)
4. [Usage Examples](#usage-examples)
5. [Rule Details](#rule-details)

## Introduction

Our custom linters are built using `@typescript-eslint/utils` and leverage TypeScript's AST capabilities to enforce project-specific rules. These linters help maintain code quality by:

- Enforcing type safety
- Preventing common React Native anti-patterns
- Ensuring consistent code style
- Catching potential runtime errors at compile time

## Available Rules

### 1. `noAnyInModifiedFiles`
**Purpose**: Ensures type safety in modified files by preventing the use of `any` type.

```typescript
// ❌ Bad
const data: any = fetchData();

// ✅ Good
const data: ApiResponse = fetchData();
```

**Implementation**: Uses AST to detect `TSAnyKeyword` nodes and git operations to check only modified files.

### 2. `noHookDep`
**Purpose**: Prevents object-like values in React hook dependencies to avoid unnecessary re-renders.

```typescript
// ❌ Bad
useEffect(() => {
  // effect
}, [{ id: 1 }]); // Object in dependency array

// ✅ Good
const dependency = useMemo(() => ({ id: 1 }), []);
useEffect(() => {
  // effect
}, [dependency]);
```

**Implementation**: Analyzes `CallExpression` nodes to find hook usages and checks dependency array types.

### 3. `enforceOptionalParams`
**Purpose**: Enforces proper typing of optional parameters using `type | undefined` instead of optional parameters.

```typescript
// ❌ Bad
function greet(name?: string) {
  return `Hello ${name}`;
}

// ✅ Good
function greet(name: string | undefined) {
  return `Hello ${name}`;
}
```

**Implementation**: Uses AST to transform optional parameter syntax and enforce type safety.

### 4. `noAnyNavigations`
**Purpose**: Ensures type-safe navigation in React Native.

```typescript
// ❌ Bad
navigation.navigate('Screen', { data: any });

// ✅ Good
navigation.navigate('Screen', { data: ScreenParams });
```

**Implementation**: Analyzes navigation calls and their parameters using TypeScript's type checker.

### 5. `noDuplicateTestId`
**Purpose**: Prevents duplicate test IDs across the application.

```typescript
// ❌ Bad
<View testID="submit-button" />
<Button testID="submit-button" />

// ✅ Good
<View testID="form-submit-button" />
<Button testID="modal-submit-button" />
```

**Implementation**: Tracks test IDs across the codebase using AST traversal.

### 6. `noLazyPngImports`
**Purpose**: Enforces proper image import patterns in React Native.

```typescript
// ❌ Bad
const image = require('./image.png');

// ✅ Good
import { Image } from 'react-native';
<Image source={require('./image.png')} />
```

**Implementation**: Analyzes import statements and image usage patterns.

### 7. `noTailwindUsage`
**Purpose**: Enforces consistent styling approach by preventing Tailwind usage.

```typescript
// ❌ Bad
<View className="flex-1 bg-white" />

// ✅ Good
<View style={styles.container} />
```

**Implementation**: Detects Tailwind class usage in JSX attributes.

### 8. `enforceExplicitUndefined`
**Purpose**: Enforces explicit undefined passing for optional parameters to make code intent clearer.

```typescript
// ❌ Bad
function formatDate(date?: Date) {}
formatDate(); // Implicit undefined

// ✅ Good
function formatDate(date: Date | undefined) {}
formatDate(undefined); // Explicit undefined
```

**Implementation**: Uses AST to detect optional parameters and enforce explicit undefined passing.

**Why**:
- Makes undefined state explicit in type signatures
- Forces conscious undefined passing
- Prevents forgotten optional chaining
- Better aligns with TypeScript strict mode

### 9. `enforceExpressionCleanup`
**Purpose**: Enforces cleaner code by preventing unused expressions without side effects.

```typescript
// ✅ Allowed
isValid && submitForm();
isLoading ? <Spinner /> : <Content />;

// ❌ Blocked
someValue; // No side effects
unusedFunction(); // No side effects
```

**Implementation**: Uses `@typescript-eslint/no-unused-expressions` with smart exceptions.

**Why**:
- Enforces cleaner, more explicit code
- Prevents accidental no-op statements
- Allows common patterns like optional chaining

### 10. `enforceStrictNavigationTypes`
**Purpose**: Ensures type-safe navigation by enforcing proper typing for navigation calls.

```typescript
// ❌ Bad
const navigation = useNavigation<StackNavigationProp<any>>();
navigation.navigate('Screen', { data: any });

// ✅ Good
const navigation: StackNavigationProp<GlobalParamList> = useNavigation();
navigation.navigate('Screen', { data: ScreenParams });
```

**Implementation**: Analyzes navigation calls and their type parameters.

**Why**:
- Ensures correct screen names & params at compile time
- Prevents runtime navigation errors
- Makes refactoring safer

### 11. `enforceImmutability`
**Purpose**: Enforces functional programming practices by preventing unintended mutations.

```typescript
// ❌ Bad
let variable = 0;
function addItem(items: string[], newItem: string) {
    items.push(newItem); // Mutates original array
}

// ✅ Good
const variable = 0;
function addItem(items: readonly string[], newItem: string) {
    return [...items, newItem]; // Returns new array
}

// ✅ Exceptions (Allowed)
const ref = useRef(0);
ref.current = 5; // Allowed mutation
const selectUserProfile = (state: RootState) => selectUser(state).profile;
```

**Implementation**: Uses AST to detect mutable operations and enforce immutability.

**Why**:
- Ensures data integrity
- Makes state transitions predictable
- Reduces race conditions
- Avoids unnecessary re-renders

### 12. `enforceStaticImageImports`
**Purpose**: Enforces ES6 imports for static images instead of require().

```typescript
// ❌ Bad
const logo = require('@/assets/logo.png');

// ✅ Good
import logo from '@/assets/logo.png';
```

**Implementation**: Analyzes image import statements and usage patterns.

**Why**:
- Ensures type safety
- Matches React Native's static asset handling
- Makes dependencies explicit

## Migration and Adoption

### Automated Fixes
Our custom linters provide automatic fixes for most cases:
```typescript
// Before
function fetchData(options?: FetchOptions) {}
const navigation = useNavigation<any>();

// After (automatically fixed)
function fetchData(options: FetchOptions | undefined) {}
const navigation: StackNavigationProp<GlobalParamList> = useNavigation();
```

### Backward Compatibility
- Rules are designed to maintain backward compatibility
- Explicit undefined passing ensures new changes don't break existing code
- Migration scripts help transition existing code

### Progress Tracking
- Reduced `any` usage from 300 to ~40 instances in src and src-v2
- Ongoing migration from Tailwind to CSS
- Continuous improvement in type safety

## Best Practices

1. **Type Safety**:
   - Always use TypeScript's type checker when available
   - Prefer explicit types over `any`
   - Use union types for optional values

2. **Performance**:
   - Cache AST traversal results when possible
   - Use early returns to avoid unnecessary processing
   - Leverage TypeScript's type system for complex checks

3. **Maintainability**:
   - Keep rules focused and single-purpose
   - Document complex AST traversals
   - Use meaningful error messages

4. **Navigation Safety**:
   - Always use `GlobalParamList` for navigation types
   - Avoid `any` in navigation props
   - Define proper params for each route

5. **Immutability**:
   - Use spread operators for updates
   - Prefer immutable utility methods
   - Document necessary mutations

6. **Asset Management**:
   - Use ES6 imports for static assets
   - Define proper type definitions for assets
   - Avoid runtime require() calls

## Contributing

To add a new custom linter:

1. Create a new file in `rules/` directory
2. Implement the rule using `@typescript-eslint/utils`
3. Add tests for the rule
4. Update this documentation
5. Add the rule to `index.ts`

### Migration Guidelines
1. Use automated fixes where available
2. Handle exceptions manually when necessary
3. Update type definitions as needed
4. Document any necessary rule exceptions

## References

- [TypeScript AST Viewer](https://ts-ast-viewer.com/)
- [@typescript-eslint Documentation](https://typescript-eslint.io/)
- [ESLint Rule Documentation](https://eslint.org/docs/developer-guide/working-with-rules)
- [React Native TypeScript Guide](https://reactnative.dev/docs/typescript)
- [TypeScript Strict Mode](https://www.typescriptlang.org/tsconfig#strict)
- [React Native Performance](https://reactnative.dev/docs/performance)
