---
tags: 
 - typescript
 - function
 - note
---

# 🔑 Typing Quirks of `Object.keys` and `Object.entries` in TypeScript

```ad-note
Once you understand that TypeScript does not strictly enforce precise object shapes and generally allows excess properties, the intentionally loose typing of `Object.keys` and `Object.entries` becomes much easier to understand.
```

---

## 🧠 The Core Quirk

In TypeScript:

```ts
Object.keys(obj);    // string[]
Object.entries(obj); // [string, any][]
```

Even when `obj` is strongly typed, **TypeScript intentionally widens the result**.

```ts
const user = {
  id: 1,
  name: "Alice",
};

Object.keys(user); 
// string[], NOT ("id" | "name")[]
```

This surprises many developers.

---

## ❓ Why They Are Loosely Typed

### Reason 1: JavaScript Runtime Reality

At runtime:

- Objects can gain or lose properties dynamically
    
- Keys can be added via prototypes
    
- `Object.keys` returns **only strings**
    

TypeScript must match **runtime behavior**, not idealized types.

---

### Reason 2: Structural Typing + Index Signatures

```ts
type User = {
  id: number;
  name: string;
};

declare const user: User;
```

Even though `User` has fixed keys, TypeScript **cannot guarantee**:

- No extra properties exist
    
- The object isn’t widened
    
- The object wasn’t mutated
    

---

### Reason 3: `string | number | symbol` → `string`

JS converts keys to strings:

```ts
const obj = { 1: "one" };
Object.keys(obj); // ["1"]
```

So TS cannot safely return `keyof T`.

---

## ⚠️ The Practical Problem

```ts
Object.keys(user).forEach(key => {
  user[key]; // ❌ Error: string cannot index User
});
```

Because:

- `key` is `string`
    
- `User` does not have a string index signature
    

---

## 🧩 Same Issue with `Object.entries`

```ts
Object.entries(user).forEach(([key, value]) => {
  key;   // string
  value; // any
});
```

TypeScript **gives up on precision**.

---

## 🛠 Common Workarounds (With Tradeoffs)

---

## ✅ 1. Type Assertion (Most Common)

```ts
const keys = Object.keys(user) as (keyof typeof user)[];
```

Now safe:

```ts
keys.forEach(key => {
  user[key]; // ✅
});
```

⚠️ You are asserting correctness manually.

---

## ✅ 2. Generic Helper Function (Best Practice)

```ts
function typedKeys<T extends object>(obj: T): (keyof T)[] {
  return Object.keys(obj) as (keyof T)[];
}
```

Usage:

```ts
typedKeys(user).forEach(key => {
  user[key]; // ✅
});
```

---

## ✅ 3. `as const` for Literal Objects

```ts
const config = {
  mode: "dev",
  retry: 3,
} as const;
```

Still:

```ts
Object.keys(config); // string[]
```

`as const` does **not** fix this by itself.

---

## 🚫 What Does NOT Work

```ts
Object.keys<T>(obj); // ❌ no generic overload
```

There is **no safe generic signature** in the standard library.

---

## 🧠 Why TypeScript Does Not “Fix” This

If TS returned `(keyof T)[]`, this would be unsound:

```ts
const obj: { a: number } = Object.create({ b: 2 });

Object.keys(obj); // ["b"] ❌ but keyof obj is "a"
```

TypeScript chooses **soundness over convenience**.

---

## 🧠 Mental Model

> `Object.keys` and `Object.entries` describe **runtime behavior**, not static structure.

---

## 📌 Best Practice Summary

- Expect `string[]` and `[string, any][]`
    
- Use typed helper functions
    
- Avoid indexing directly with raw `Object.keys`
    
- Prefer `for...in` with `hasOwn` checks when appropriate
    
- Use assertions deliberately and sparingly
    

---

## 🧠 One-Sentence Summary

> `Object.keys` and `Object.entries` are loosely typed in TypeScript by design to reflect JavaScript’s dynamic runtime behavior, even when the input object is strongly typed.