# Custom Linters Documentation

This document outlines the custom linting rules implemented in our React Native project to maintain code quality and consistency.

## Table of Contents
1. [Explicit Optional Parameters](#explicit-optional-parameters)
2. [Safe Hook Dependencies](#safe-hook-dependencies)
3. [Explicit Undefined Passing](#explicit-undefined-passing)
4. [Expression Cleanup](#expression-cleanup)
5. [Tailwind Usage Restriction](#tailwind-usage-restriction)
6. [Strict Stack Navigation Types](#strict-stack-navigation-types)
7. [Strict Types in Modified Files](#strict-types-in-modified-files)
8. [Enforced Immutability](#enforced-immutability)
9. [ES6 Imports for Static Images](#es6-imports-for-static-images)

## Explicit Optional Parameters

**Rule**: Replace optional parameters (`param?: Type`) with union types (`param: Type | undefined`).

```typescript
// ❌ Avoid
function fetchData(options?: FetchOptions) {}

// ✅ Use
function fetchData(options: FetchOptions | undefined) {}
```

**Benefits**:
- Makes undefined state explicit in type signatures
- Forces conscious undefined passing
- Prevents forgotten optional chaining
- Reduces runtime surprises

## Safe Hook Dependencies

**Rule**: Objects/arrays in useEffect dependencies will trigger warnings.

```typescript
// ❌ Avoid - unstable dependency
useEffect(() => {
  // effect
}, [userData]);

// ✅ Use - stable dependencies
useEffect(() => {
  // effect
}, [userData.id]);
```

**Benefits**:
- Prevents infinite loops
- Forces stable dependency references
- Makes effect dependencies more predictable
- Easier to track effect execution

## Explicit Undefined Passing

**Rule**: Functions with `| undefined` parameters require explicit undefined when called without arguments.

```typescript
// ❌ Avoid
function formatDate(date?: Date) {}
formatDate();

// ✅ Use
function formatDate(date: Date | undefined) {}
formatDate(undefined);
```

**Benefits**:
- Clearer intent in function calls
- Better TypeScript strict mode compliance
- Safer parameter additions
- Prevents accidental undefined passing

## Expression Cleanup

**Rule**: Enforces `@typescript-eslint/no-unused-expressions` with smart exceptions.

```typescript
// ✅ Allowed
isValid && submitForm();
isLoading ? <Spinner /> : <Content />;

// ❌ Blocked
someValue; // No side effects
```

**Benefits**:
- Prevents unused expressions
- Catches accidental no-op statements
- Allows common patterns (optional chaining, short-circuiting)

## Tailwind Usage Restriction

**Rule**: Tailwind-based styling is disallowed in favor of React Native's StyleSheet.

```typescript
// ❌ Avoid
const styles = tailwind('bg-red-500 text-white');

// ✅ Use
import { StyleSheet } from 'react-native';
const styles = StyleSheet.create({
  container: {
    backgroundColor: 'red',
    color: 'white',
  },
});
```

**Benefits**:
- Prevents unnecessary re-renders
- Better performance
- Native styling approach

## Strict Stack Navigation Types

**Rule**: Enforces proper typing for navigation calls.

```typescript
// ❌ Avoid
const navigation = useNavigation<StackNavigationProp<any>>();

// ✅ Use
const navigation: StackNavigationProp<GlobalParamList> = useNavigation();
```

**Benefits**:
- Type-safe navigation
- Compile-time screen name validation
- Safer refactoring
- Prevents runtime navigation errors

## Strict Types in Modified Files

**Rule**: Eliminates `any` usage in modified files.

```typescript
// ❌ Avoid
const handleIntentReceived = async (event: any) => {};

// ✅ Use
const handleIntentReceived = async (event: IntentEvent) => {};
```

**Benefits**:
- Ensures type safety
- Improves maintainability
- Better IDE support
- Prevents runtime type errors

## Enforced Immutability

**Rule**: Enforces functional programming practices and prevents unintended mutations.

```typescript
// ❌ Avoid
let variable = 0;
function addItem(items: string[], newItem: string) {
    items.push(newItem);
}

// ✅ Use
const variable = 0;
function addItem(items: readonly string[], newItem: string) {
    return [...items, newItem];
}

// ✅ Exceptions (allowed)
const ref = useRef(0);
ref.current = 5; // useRef mutation allowed
```

**Benefits**:
- Predictable state transitions
- Easier testing
- Reduced race conditions
- Better performance through stable references

## ES6 Imports for Static Images

**Rule**: Use ES6 imports instead of require() for static images.

```typescript
// ❌ Avoid
const logo = require('@/assets/logo.png');

// ✅ Use
import logo from '@/assets/logo.png';
```

**Benefits**:
- Type-safe image imports
- Consistent asset handling
- Better TypeScript integration
- Explicit dependencies

## Migration and Maintenance

- Automated fixes are available for most rules
- Backward compatibility is maintained
- Rules can be disabled when necessary with proper justification
- For issues or exceptions, please raise them in the team channel

## Type Definitions

For proper TypeScript support, ensure you have the following in your global type definitions:

```typescript
declare module '*.png' {
  const value: number;
  export default value;
}
```
