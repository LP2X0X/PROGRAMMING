---
tags: 
 - typescript
 - operator
---

## 🔑 `keyof` Operator (TypeScript)

### Category

- **Type operator**
    
- **Indexing / reflection operator**
    
- **Structural typing**
    

---

## What `keyof` does

```ts
keyof T
```

Produces a **union of property keys** of type `T`.

---

## Basic example

```ts
type User = {
  id: number;
  name: string;
  active: boolean;
};

type UserKeys = keyof User;
// "id" | "name" | "active"
```

---

## Resulting type

`keyof T` is always a subtype of:

```ts
PropertyKey // string | number | symbol
```

---

## `keyof` with index signatures

### String index signature

```ts
type Dict = {
  [key: string]: number;
};

type Keys = keyof Dict;
// string | number
```

Reason:

- JS coerces numeric keys to strings
    
- TS reflects that behavior
    

---

### Number index signature

```ts
type Arr = {
  [index: number]: string;
};

type Keys = keyof Arr;
// number
```

---

## `keyof` and unions (important)

```ts
type A = { a: number };
type B = { b: string };

type Keys = keyof (A | B);
// never
```

Why:

- Only keys **common to all union members** are allowed
    
- There are none
    

---

## `keyof` and intersections

```ts
type Keys = keyof (A & B);
// "a" | "b"
```

Intersections **merge** properties.

---

## `keyof` with generics

```ts
function get<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

Why this works:

- `K` is constrained to valid keys of `T`
    
- `T[K]` is safe
    

---

## Relationship to `in` (mapped types)

```ts
type Optional<T> = {
  [K in keyof T]?: T[K];
};
```

Flow:

1. `keyof T` → union of keys
    
2. `in` → iterate over that union
    
3. Produce a new object type
    

---

## `keyof any`

```ts
keyof any
// string | number | symbol
```

This is effectively `PropertyKey`.

---

## Common pitfalls

### Assuming union distribution

```ts
keyof (A | B) // ❌ not "a" | "b"
```

### Confusing runtime vs type-level

`keyof`:

- ❌ Does not exist at runtime
    
- ✅ Compile-time only
    

---

## Mental model

> **`keyof` asks the type system: “What keys are guaranteed to exist?”**

---

## Summary

| Expression       | Result                  |
| ---------------- | ----------------------- |
| `keyof { a: 1 }` | `"a"`                   |
| `keyof T`        | Union of property names | 
| `keyof any`      | `PropertyKey`           |
| `keyof (A & B)`  | Union of keys           |
| `keyof (A \| B)` | Intersection of keys    |

---

## One-line takeaway

> **`keyof` turns an object type into a union of its property names, constrained by structural safety.**