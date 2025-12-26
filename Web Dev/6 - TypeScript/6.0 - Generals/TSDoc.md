---
tags: 
 - typescript
 - term
---

**TSDoc** is a **documentation standard** for TypeScript code, similar in spirit to JSDoc, but **designed specifically for TypeScript’s type system**.

It defines **comment syntax + semantics** so tools can generate consistent, structured API documentation.

---

## What TSDoc is (and is not)

**Is**

- A standardized doc comment format
    
- Type-aware (works with generics, unions, overloads)
    
- Used by TypeScript libraries and tooling
    
- Parsed by documentation generators (e.g., API Extractor)
    

**Is not**

- A compiler feature
    
- A runtime feature
    
- Required for TypeScript to work
    

---

## Basic TSDoc syntax

TSDoc uses **`/** ... */`** comments placed immediately before declarations.

```ts
/**
 * Adds two numbers.
 *
 * @param a - The first number
 * @param b - The second number
 * @returns The sum of `a` and `b`
 */
export function add(a: number, b: number): number {
  return a + b;
}
```

---

## Common TSDoc tags

| Tag                     | Purpose                       |
| ----------------------- | ----------------------------- |
| `@param`                | Describe a parameter          |
| `@returns`              | Describe return value         |
| `@remarks`              | Additional details            |
| `@example`              | Usage example                 |
| `@throws`               | Errors that may be thrown     |
| `@deprecated`           | Mark API as deprecated        | 
| `@see`                  | Reference related APIs        |
| `@public` / `@internal` | API visibility (with tooling) |

Example:

```ts
/**
 * Fetches user data from the API.
 *
 * @remarks
 * This function caches results for 5 minutes.
 *
 * @example
 * ```ts
 * const user = await getUser(1);
 * ```
 *
 * @throws Error if the request fails
 */
export async function getUser(id: number): Promise<User> {
  ...
}
```

---

## How TSDoc relates to editors

- Editors (VS Code) show TSDoc in:
    
    - Hover tooltips
        
    - IntelliSense completion
        
- `tsserver` reads these comments and surfaces them in the UI
    

So TSDoc directly improves **developer experience**, not runtime behavior.

---

## TSDoc vs JSDoc

| Aspect          | JSDoc           | TSDoc              |
| --------------- | --------------- | ------------------ |
| Origin          | JavaScript      | TypeScript         |
| Type syntax     | JSDoc-specific  | Native TypeScript  |
| Standardization | Loosely defined | Strict spec        |
| Tooling         | Broad           | TypeScript-focused |

**Key difference**:  
With TypeScript, you **do not** repeat types in comments.

❌ JSDoc-style:

```ts
/** @param {number} x */
```

✅ TSDoc-style:

```ts
/** @param x - A number */
```

---

## Advanced features

### Linking symbols

```ts
/**
 * See {@link User} for details.
 */
```

### Generics

```ts
/**
 * Wraps a value in an array.
 *
 * @typeParam T - The item type
 */
function wrap<T>(value: T): T[] {
  return [value];
}
```

---

## When you should use TSDoc

Use TSDoc when:

- Writing libraries or shared packages
    
- Maintaining public APIs
    
- Working in teams where API clarity matters
    
- Generating docs automatically
    

You may skip it for:

- Small scripts
    
- Throwaway prototypes
    

---

### One-sentence summary

> **TSDoc is a standardized, type-aware documentation format that makes TypeScript APIs self-explanatory to both humans and tools.**