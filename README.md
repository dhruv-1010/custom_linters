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

## AST Implementation

Our linters use TypeScript's AST capabilities through `@typescript-eslint/utils`. Here's a simplified example of how AST is used:

```typescript
// Example AST node structure for a function
FunctionDeclaration {
    id: Identifier { name: "greet" },
    params: [
        Parameter {
            name: "name",
            typeAnnotation: TSTypeAnnotation {
                typeAnnotation: TSUnionType {
                    types: [
                        TSStringKeyword {},
                        TSUndefinedKeyword {}
                    ]
                }
            }
        }
    ],
    body: BlockStatement {
        // Function body
    }
}
```

## Usage Examples

### 1. Creating a New Linter

```typescript
import { ESLintUtils } from '@typescript-eslint/utils';
import type { TSESTree } from '@typescript-eslint/types';

export const myCustomRule = ESLintUtils.RuleCreator.withoutDocs({
    meta: {
        type: 'problem',
        messages: {
            errorMessage: 'Custom error message'
        }
    },
    create(context) {
        return {
            // AST visitor methods
            CallExpression(node: TSESTree.CallExpression) {
                // Rule implementation
            }
        };
    }
});
```

### 2. Using TypeScript's Type Checker

```typescript
const parserServices = context.sourceCode.parserServices;
const checker = parserServices?.program?.getTypeChecker();

if (checker) {
    const type = checker.getTypeAtLocation(node);
    // Type analysis
}
```

## Rule Details

### AST Node Types Used

1. **Type Checking Nodes**:
   - `TSAnyKeyword`
   - `TSUnionType`
   - `TSIntersectionType`
   - `TSTypeReference`

2. **JSX Nodes**:
   - `JSXElement`
   - `JSXAttribute`
   - `JSXExpressionContainer`

3. **Function Nodes**:
   - `FunctionDeclaration`
   - `ArrowFunctionExpression`
   - `CallExpression`

4. **TypeScript Specific Nodes**:
   - `TSInterfaceDeclaration`
   - `TSPropertySignature`
   - `TSTypeAnnotation`

### Common Patterns

1. **Type Checking**:
```typescript
function isObjectOrArrayType(type: ts.Type, checker: ts.TypeChecker): boolean {
    return type.isUnion()
        ? type.types.some(t => isObjectOrArrayType(t, checker))
        : type.getFlags() & ts.TypeFlags.Object;
}
```

2. **JSX Analysis**:
```typescript
JSXElement(node: TSESTree.JSXElement) {
    if (node.openingElement.name.type === 'JSXIdentifier') {
        // Analyze JSX element
    }
}
```

3. **Function Analysis**:
```typescript
CallExpression(node: TSESTree.CallExpression) {
    if (node.callee.type === 'Identifier') {
        // Analyze function calls
    }
}
```

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

## Contributing

To add a new custom linter:

1. Create a new file in `rules/` directory
2. Implement the rule using `@typescript-eslint/utils`
3. Add tests for the rule
4. Update this documentation
5. Add the rule to `index.ts`

## References

- [TypeScript AST Viewer](https://ts-ast-viewer.com/)
- [@typescript-eslint Documentation](https://typescript-eslint.io/)
- [ESLint Rule Documentation](https://eslint.org/docs/developer-guide/working-with-rules)
