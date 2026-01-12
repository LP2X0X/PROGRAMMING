---
tags: 
 - typescript
 - type
 - modifier
---

## What `readonly` does

```ts
readonly
```

Prevents **assignment to a property** after initialization.

```ts
type User = {
  readonly id: number;
  name: string;
};

const u: User = { id: 1, name: "Alice" };
u.id = 2; // ❌ Error
```

---

## `readonly` is **not** `const`

| Aspect         | `const`      | `readonly`      |
| -------------- | ------------ | --------------- |
| Applies to     | Variables    | Properties      |
| Runtime effect | Yes          | No              |
| Reassignment   | ❌           | ❌              |
| Mutation       | ✅ (objects) | ❌ (properties) |

---

## `readonly` does not freeze values

```ts
type Box = {
  readonly items: number[];
};

box.items.push(1); // ✅ allowed
box.items = [];    // ❌ not allowed
```

It protects the **reference**, not the inner structure.

---

## `readonly` in interfaces

```ts
interface Config {
  readonly url: string;
  readonly timeout?: number;
}
```

Common in public APIs.

---

## `readonly` in mapped types

```ts
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};
```

### Removing readonly

```ts
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};
```

---

## ❗ `readonly` with arrays and tuples

### Readonly array

```ts
readonly number[]
ReadonlyArray<number>
```

Prevents mutation methods.

---

### Readonly tuple

```ts
readonly [number, string]
```

Preserves tuple shape and immutability.

---

## `as const` and `readonly`

```ts
const config = {
  mode: "dark",
  retries: 3,
} as const;
```

Effects:

- All properties become `readonly`
    
- Values become literal types
    
- Deep (but shallow for objects inside)
    

---

## `readonly` and function parameters

```ts
function log(user: Readonly<User>) {
  // user.id = 3; ❌
}
```

Prevents accidental mutation.

---

## Mental model

> **`readonly` protects object shape, not object contents.**

---

## Common pitfalls

- `readonly` ≠ deep freeze
    
- `readonly` is erased at runtime
    
- Mutation through aliases is still possible
    

---

## Summary

| Feature    | Behavior               |
| ---------- | ---------------------- |
| Scope      | Property, array, tuple |
| Runtime    | ❌                     |
| Assignment | ❌                     |
| Mutation   | Depends on value type  |

---

## One-line takeaway

> **`readonly` enforces immutability at the type level by disallowing property reassignment.**