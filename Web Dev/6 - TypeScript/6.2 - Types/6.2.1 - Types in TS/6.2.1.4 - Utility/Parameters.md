---
tags: 
 - typescript
 - type
 - utility
---

## `Parameters<T>` in TypeScript

### Definition

`Parameters<T>` is a **built-in utility type** that extracts a **tuple of parameter types** from a function type `T`.

```ts
type Parameters<T extends (...args: any) => any>
```

It works only on **function types**.

---

## Basic Example

```ts
function sum(a: number, b: string): boolean {
  return true;
}

type SumParams = Parameters<typeof sum>;
// [number, string]
```

Result:

```ts
type SumParams = [number, string];
```

---

## How It Works (Conceptually)

Internally, `Parameters<T>` is defined using a **conditional type** with `infer`:

```ts
type Parameters<T> =
  T extends (...args: infer P) => any ? P : never;
```

Meaning:

- If `T` is a function type
    
- Infer (`infer P`) the parameter list
    
- Return it as a tuple
    

---

## Why the Result Is a Tuple

Function parameters:

- Are ordered
    
- Can be optional
    
- Can include rest parameters
    

Tuples preserve all of that information.

Example:

```ts
type Fn = (x: number, y?: string, ...z: boolean[]) => void;

type FnParams = Parameters<Fn>;
// [number, (string | undefined)?, ...boolean[]]
```

---

## Common Use Cases

### 1. Forwarding arguments safely

```ts
function call<T extends (...args: any[]) => any>(
  fn: T,
  ...args: Parameters<T>
): ReturnType<T> {
  return fn(...args);
}
```

Ensures:

- Correct argument count
    
- Correct argument types
    

---

### 2. Reusing API signatures

```ts
type Handler = (event: Event, id: string) => void;

type HandlerArgs = Parameters<Handler>;
```

Prevents duplication and drift.

---

### 3. Wrapper / decorator functions

```ts
function log<T extends (...args: any[]) => any>(fn: T) {
  return (...args: Parameters<T>) => {
    console.log(args);
    return fn(...args);
  };
}
```

---

## Interaction with Overloaded Functions

For overloaded functions, `Parameters<T>` extracts parameters from the **last overload signature**.

```ts
function fn(x: number): number;
function fn(x: string): string;
function fn(x: any) {
  return x;
}

type FnParams = Parameters<typeof fn>;
// [string]
```

This is a known limitation.

---

## With Methods and Class Functions

```ts
class Service {
  run(a: number, b: boolean) {}
}

type RunParams = Parameters<Service["run"]>;
// [number, boolean]
```

---

## Constraints and Errors

If `T` is not a function:

```ts
type X = Parameters<number>; // Error
```

Because:

```ts
T extends (...args: any) => any
```

---

## Relationship to Other Utility Types

| Utility                    | Purpose                        |
| -------------------------- | ------------------------------ |
| `Parameters<T>`            | Extract parameter tuple        |
| `ReturnType<T>`            | Extract return type            |
| `ConstructorParameters<T>` | Extract constructor parameters |
| `InstanceType<T>`          | Extract instance type          |

---

## Runtime Impact

- `Parameters<T>` is **type-only**
    
- Fully **erasable**
    
- No runtime code emitted
    
- Compatible with `--experimental-strip-types`
    

---

## Key Pitfalls

- Loses overload info (uses last signature)
    
- Requires function type or `typeof fn`
    
- Does not work on callable objects without signatures
    

---

## One-Line Summary

> **`Parameters<T>` extracts a function’s parameter list as a tuple, enabling type-safe argument reuse, forwarding, and API composition.**