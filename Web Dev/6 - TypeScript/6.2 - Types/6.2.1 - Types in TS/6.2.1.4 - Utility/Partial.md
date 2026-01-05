---
tags: 
 - typescript
 - type
 - utility
---

## 🧩 `Partial<T>` (TypeScript Utility Type)

### Category

- **Utility type**
    
- **Mapped type**
    
- **Non-distributive**
    

---

## What `Partial<T>` does

```ts
Partial<T>
```

Makes **all properties of `T` optional**.

---

## Basic example

```ts
type User = {
  id: number;
  name: string;
  active: boolean;
};

type PartialUser = Partial<User>;
```

Equivalent to:

```ts
type UserDraft = {
  id?: number;
  name?: string;
  active?: boolean;
};
```

---

## How it works internally

```ts
type Partial<T> = {
  [K in keyof T]?: T[K];
};
```

Key points:

- `keyof T` → union of keys
    
- `in` → iterates over keys
    
- `?` → optional modifier
    
- `T[K]` → indexed access
    

---

## `Partial` is **not distributive**

```ts
type A = { a: number };
type B = { b: string };

type X = Partial<A | B>;
```

Result:

```ts
{
  a?: number;
  b?: string;
}
```

It **does not** become:

```ts
Partial<A> | Partial<B>
```

Why:

- Mapped types operate on the **entire union**
    
- No `extends` → no distribution
    

---

## When to use `Partial`

- Update / patch objects
    
- Form drafts
    
- Configuration overrides
    
- Builder patterns
    

---

## When **not** to use `Partial`

❌ When missing properties must be rejected  
❌ When you want union preservation

Use **DistributedPartial** instead.

---

## Distributed version (advanced)

```ts
type DistributedPartial<T> =
  T extends any ? Partial<T> : never;
```

This preserves union members.

---

## Common pitfall

```ts
function update(user: User, patch: Partial<User>) {
  user.id = patch.id; // ❌ possibly undefined
}
```

`Partial` allows `undefined`, not omission only.

---

## Mental model

> **`Partial<T>` loosens structure by making presence optional, not values nullable.**

---

## Summary

| Feature      | Behavior         |
| ------------ | ---------------- |
| Type         | Utility / mapped |
| Distribution | ❌ No            |
| Optionality  | All keys         |
| Runtime      | ❌ None          |

---

## One-line takeaway

> **`Partial<T>` converts every property into an optional one using a mapped type.**