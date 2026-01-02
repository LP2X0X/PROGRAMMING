---
tags: 
 - typescript
 - operator
---

### Overview

`in` is a **context-dependent keyword** in TypeScript.  
It exists in **two separate layers**:

1. **Runtime (JavaScript operator)**
    
2. **Compile-time (TypeScript type keyword)**
    

These two uses share syntax but have **no shared semantics**.

---

## 1️⃣ `in` at Runtime (JavaScript Operator)

### Definition

At runtime, `in` is a **JavaScript operator** that checks whether a property exists on an object.

```ts
"length" in [];
// true
```

### Characteristics

- Checks **own + inherited** properties
    
- Works on objects only
    
- Returns a boolean
    
- Executed at runtime
    

### Common use

```ts
if ("id" in value) {
  // property existence check
}
```

---

## 2️⃣ `in` at Compile Time (TypeScript Keyword)

### Definition

At the type level, `in` is a **TypeScript keyword** used to **iterate over a union** of keys.

```ts
type Flags<T> = {
  [K in keyof T]: boolean;
};
```

Here, `in` means:

> “For each key `K` in the union `keyof T`…”

---

## 3️⃣ `in` in Mapped Types

### Core syntax

```ts
[K in U]: V
```

- `U` must be a **union of property keys**
    
- Produces one property per union member
    
- Keys are **required by default**
    

Example:

```ts
type Pages = "home" | "about";

type PageMap = {
  [P in Pages]: string;
};
```

Result:

```ts
{
  home: string;
  about: string;
}
```

---

## 4️⃣ `in` vs Index Signatures

```ts
// Index signature
type A = { [key: string]: number };

// Mapped type
type B = { [K in string]: number };
```

- `A` allows **any** string key
    
- `B` is illegal (mapped types require finite unions)
    

Mapped types **require enumerability**; index signatures do not.

---

## 5️⃣ `in` and Modifiers

Mapped types can modify properties:

```ts
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};
```

Modifiers supported:

- `readonly`
    
- `?`
    
- `-readonly`
    
- `-?`
    

---

## 6️⃣ `in` in Conditional Narrowing (Runtime)

TypeScript uses runtime `in` to **narrow types**:

```ts
function f(x: A | B) {
  if ("a" in x) {
    // x is A
  }
}
```

Here:

- `in` executes at runtime
    
- TypeScript _observes_ the result to narrow types
    

---

## 7️⃣ What `in` is NOT

- ❌ Not a helper type
    
- ❌ Not a utility type
    
- ❌ Not syntax sugar
    
- ❌ Not related to `for...in` at type level
    

---

## 8️⃣ Mental models

### Runtime

> **“Does this object have this property?”**

### Type-level

> **“Generate one property for each key in this union.”**

---

## 9️⃣ Category placement (recommended)

- **Language Keyword**
    
- **Type Operator (Type-level)**
    
- **Mapped Type Syntax**
    
- **Runtime Operator (JS)**
    

---

## 🔍 Summary table

| Layer      | Role                        | Executes     |
| ---------- | --------------------------- | ------------ |
| JavaScript | Property existence operator | Runtime      |
| TypeScript | Union enumerator            | Compile-time |

---

## One-line takeaway

> **`in` checks existence at runtime, and enumerates possibilities at the type level — same word, completely different jobs.**
