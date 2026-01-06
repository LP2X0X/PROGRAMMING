---
tags: 
 - typescript
 - operator
 - note
---

The **`keyof typeof` chain** is a TypeScript pattern used to **extract a union of property names from a runtime value**.

---

## Why the Chain Exists

- `typeof` (type position) converts a **value → type**
    
- `keyof` extracts the **keys of that type**
    

You need both because `keyof` works on **types**, not values.

---

## Basic Example

```ts
const colors = {
  red: "#f00",
  green: "#0f0",
  blue: "#00f",
};

type ColorName = keyof typeof colors;
// "red" | "green" | "blue"
```

---

## Step-by-Step Meaning

```ts
typeof colors
```

➡️ Gets the type:

```ts
{
  red: string;
  green: string;
  blue: string;
}
```

```ts
keyof (typeof colors)
```

➡️ Produces:

```ts
"red" | "green" | "blue"
```

---

## Why Not Just `keyof colors`?

This is invalid:

```ts
keyof colors // ❌ error
```

Because:

- `colors` is a **value**
    
- `keyof` only operates on **types**
    

---

## Common Real-World Use Case

Enums replacement (preferred in modern TS):

```ts
const Roles = {
  Admin: "admin",
  User: "user",
  Guest: "guest",
} as const;

type Role = keyof typeof Roles;
// "Admin" | "User" | "Guest"
```

Or values:

```ts
type RoleValue = (typeof Roles)[keyof typeof Roles];
// "admin" | "user" | "guest"
```

---

## One-Line Summary

> **`keyof typeof X` means: “give me a union of all property names of the value `X`.”**