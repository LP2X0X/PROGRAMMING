---
tags: 
 - typescript
 - type
 - utility
---

`Record<K, T>` is a **utility (helper) type** that constructs an **object type** whose:

- **keys** come from `K`
    
- **values** are all of type `T`
    

```ts
Record<K, T>
```

It answers the question:

> “I want an object where I know the set of keys, and all values share the same type.”

---

## 🔑 Basic Syntax

```ts
type UserRoles = Record<string, number>;
```

Equivalent to:

```ts
type UserRoles = {
  [key: string]: number;
};
```

But `Record` is **more explicit and constrained**.

---

## 🎯 Common Example

```ts
type Page = "home" | "about" | "contact";

type PageTitles = Record<Page, string>;
```

Resulting type:

```ts
{
  home: string;
  about: string;
  contact: string;
}
```

All keys are **required**.

---

## ⚠️ Required keys (important behavior)

`Record` **does not allow missing keys**:

```ts
const titles: PageTitles = {
  home: "Home",
  about: "About",
  // ❌ contact missing → error
};
```

This makes `Record` useful when **completeness matters**.

---

## 🧠 Mental Model

Think of `Record` as:

> “For every key in `K`, map it to a value of type `T`.”

Formally, it is a **mapped type**:

```ts
type Record<K extends keyof any, T> = {
  [P in K]: T;
};
```

---

## 🧩 `Record` vs Index Signature

### Index signature

```ts
type Map = {
  [key: string]: number;
};
```

- Any string key allowed
    
- No guarantee which keys exist
    

### Record

```ts
type Map = Record<"a" | "b", number>;
```

- Only `"a"` and `"b"` allowed
    
- Both must exist
    

✅ `Record` is **stricter and safer**

---

## 🧪 Using `Record` with other utility types

```ts
type Config = Record<string, boolean>;

type OptionalConfig = Partial<Config>;
```

Or:

```ts
type SafeConfig = Readonly<Record<string, boolean>>;
```

---

## 🚫 What `Record` is NOT

- ❌ Not a runtime object
    
- ❌ Not a map data structure
    
- ❌ Not a class
    
- ❌ Not flexible like `Map<K, V>`
    

It exists **only at compile time**.

---

## 🧩 Typical Use Cases

- Lookup tables
    
- Configuration maps
    
- Enum-like objects
    
- State dictionaries
    
- Permission matrices
    

```ts
type Permission = "read" | "write";

const perms: Record<Permission, boolean> = {
  read: true,
  write: false,
};
```

---

## 📌 Where `Record` fits in TypeScript concepts

- Category: **Utility / Helper Type**
    
- Built from: **Mapped Types**
    
- Used in: **Type composition**
    
- Not a type annotation itself
    

---

## 📝 One-line summary

> **`Record<K, T>` creates a type-safe object where every key in `K` maps to a value of type `T`.**