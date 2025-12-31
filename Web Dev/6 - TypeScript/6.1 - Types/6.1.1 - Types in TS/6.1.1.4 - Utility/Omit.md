---
tags: 
 - typescript
 - type
 - utility
---

### Definition

`Omit` is a **built-in utility type** that constructs a new object type by **removing one or more properties** from an existing object type.

---

## Syntax

```ts
Omit<T, K>
```

Where:

- `T` is an object type
    
- `K` is a union of keys to remove
    

Constraint (conceptual):

```ts
K extends PropertyKey
```

---

## What `Omit` does

```ts
type User = {
  id: number;
  name: string;
  password: string;
};

type PublicUser = Omit<User, "password">;
```

Result:

```ts
{
  id: number;
  name: string;
}
```

All properties except those listed in `K` remain.

---

## How `Omit` works internally

`Omit` is **defined using `Pick` and `Exclude`**:

```ts
type Omit<T, K extends PropertyKey> =
  Pick<T, Exclude<keyof T, K>>;
```

This means:

1. Remove `K` from `keyof T`
    
2. Pick the remaining keys
    

---

## Why `Omit` exists

- Avoids repetitive redefinition
    
- Keeps derived types in sync
    
- Improves refactor safety
    
- Makes intent explicit
    

---

## `Omit` vs `Pick`

| Utility      | Meaning       |
| ------------ | ------------- |
| `Pick<T, K>` | Keep only `K` |
| `Omit<T, K>` | Remove `K`    |

They are **logical opposites**.

---

## Important characteristics

- Preserves property types and modifiers
    
- Remaining properties are **required**
    
- Order of keys does not matter
    
- Works only with object types
    

---

## Omit does not distributed over Unions

When `T` is a **union type**, `Omit<T, K>` does **not** apply to each union member individually.

```ts
type A = { a: number; common: string; id: number };
type B = { b: boolean; common: string; id: number };

type U = A | B;

type R = Omit<U, "common">; 

// Below, R will have type {id: number, common: string}
// type R = Omit<U, "test">;
```

```ad-note
Omit now will try do its thing with the "mushed" type which is now {id: number, common: string} in the above example.
```

#### What you might expect

```ts
type R =
  | { a: number }
  | { b: boolean };
```

#### What actually happens

```ts
type R = {
  a?: number;
  b?: boolean;
};
```

#### Why this happens

`Omit` is defined as:

```ts
type Omit<T, K extends PropertyKey> =
  Pick<T, Exclude<keyof T, K>>;
```

Key points:

- `keyof (A | B)` becomes `"a" | "b" | "common"`
    
- The union is **collapsed** before `Pick` runs
    
- Properties not shared by all members become **optional**
    

> `Omit` operates on the **union as a whole**, not on each member.

---

## Common use cases

- Removing sensitive fields (`password`)
    
- Creating update DTOs
    
- API response shaping
    
- Deriving view models
    

---

## Limitations & pitfalls

- Does **not** affect runtime objects
    
- Removing keys from union object types can produce unexpected results
    
- Cannot remove properties conditionally without mapped types
    

---

## Mental model

> **`Omit` = “Take this type and subtract these keys.”**

---

## Category placement (for your notes)

- **Utility Types**
    
- **Mapped Types**
    
- **Type Transformation**
    

---

## One-line takeaway

> **`Omit` creates a new type by excluding specific properties from an existing object type.**