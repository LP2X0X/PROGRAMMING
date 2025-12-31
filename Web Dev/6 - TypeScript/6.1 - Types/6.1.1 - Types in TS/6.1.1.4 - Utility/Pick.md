---
tags: 
 - typescript
 - type
 - utility
---

### Definition

`Pick` is a **built-in utility type** that constructs a new object type by **selecting a subset of properties** from another type.

---

## Syntax

```ts
Pick<T, K>
```

Where:

- `T` is an object type
    
- `K` is a union of property keys from `T`
    

Constraint:

```ts
K extends keyof T
```

---

## What `Pick` does

```ts
type User = {
  id: number;
  name: string;
  email: string;
};

type PublicUser = Pick<User, "id" | "name">;
```

Result:

```ts
{
  id: number;
  name: string;
}
```

Only the selected keys remain; their original types are preserved.

---

## How `Pick` works (internally)

```ts
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

This is a **[[Mapped|mapped type]]** that:

- Iterates over `K`
    
- Copies each property from `T`
    

---

## Why `Pick` exists

Without `Pick`, you would need to:

- Manually redefine object subsets
    
- Maintain consistency when base types change
    

`Pick` enables:

- Reuse
    
- Refactoring safety
    
- Precise API typing
    

---

## `Pick` vs related utilities

### `Omit`

```ts
Omit<T, K> // remove keys
```

### `Partial`

```ts
Partial<T> // make all properties optional
```

### `Record`

```ts
Record<K, V> // create new object type
```

`Pick` **preserves structure**, others transform it differently.

---

## Common use cases

- API response shaping
    
- Public vs private models
    
- Component props narrowing
    
- DTO and view-model definitions
    

---

## Important characteristics

- Selected properties are **required**
    
- Property modifiers (`readonly`, `?`) are preserved
    
- Order does not matter
    
- Produces a **new type**, not a subtype at runtime
    

---

## What `Pick` is NOT

- ❌ Runtime filtering
    
- ❌ Object destructuring
    
- ❌ A conditional type
    

It exists **only at the type level**.

---

## Mental model

> **`Pick` = “Copy only these keys from this type.”**

---

## Category placement (for your notes)

- **Utility Types**
    
- **Mapped Types**
    
- **Type Transformation**
    

---

## One-line takeaway

> **`Pick` creates a new type by selecting specific properties from an existing object type.**
