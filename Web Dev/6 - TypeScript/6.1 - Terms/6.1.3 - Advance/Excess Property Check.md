---
tags: 
 - typescript
 - term
 - advance
---

## Prerequisite
[[Structural Typing]]

---

## 🧠 What Excess Property Checking Is

Excess property checks are a **special, stricter check** that TypeScript applies **only** when:

- Assigning a **fresh object literal**
    
- To a **target type with known properties**
    

```ts
type User = { name: string };

const u: User = {
  name: "Alice",
  age: 30, // ❌ Excess property error
};
```

This is **not** normal structural typing — it’s an extra safety feature.

---

## 🚪 When Excess Property Checks Are Bypassed

---

## 🟢 1. Non-Literal (Non-Fresh) Objects

```ts
const temp = {
  name: "Alice",
  age: 30,
};

const user: User = temp; // ✅ No error
```

Why:

- Object is **not fresh**
    
- Structural typing applies
    
- Extra properties are allowed
    

---

## 🟢 2. Type Assertions (`as`)

```ts
const user = {
  name: "Alice",
  age: 30,
} as User; // ✅ No error (unsafe)
```

TypeScript trusts you.

---

## 🟢 3. Using `any`

```ts
const user: User = {
  name: "Alice",
  age: 30,
} as any; // ✅ No error
```

`any` disables checking.

---

## 🟢 4. Index Signatures

```ts
type User = {
  name: string;
  [key: string]: unknown;
};

const user: User = {
  name: "Alice",
  age: 30, // ✅ allowed
};
```

Index signatures explicitly allow extra properties.

---

## 🟢 5. Assigning to `object`, `{}`, or `unknown`

```ts
const u: object = { name: "Alice", age: 30 }; // ✅
const v: {} = { a: 1 };                       // ✅
const w: unknown = { x: 1 };                  // ✅
```

These types impose **no property constraints**.

---

## 🟢 6. Passing Through a Generic

```ts
function accept<T>(value: T) {
  return value;
}

accept<User>({
  name: "Alice",
  age: 30, // ✅ no excess check
});
```

Generics disable freshness checks.

---

## 🟢 7. Function Parameter Assignments (Indirect)

```ts
function takeUser(u: User) {}

const obj = { name: "Alice", age: 30 };

takeUser(obj); // ✅
```

Same rule: non-fresh object.

---

## 🔍 When Excess Property Checks DO Apply

```ts
takeUser({
  name: "Alice",
  age: 30, // ❌ excess property error
});
```

Fresh object literal → strict check.

---

## 🧠 Why TypeScript Does This

To catch bugs like:

```ts
takeUser({
  name: "Alice",
  agge: 30, // typo
});
```

But without breaking structural typing.

---

## 🧠 Mental Model

> Excess property checks only apply to **fresh object literals** and disappear as soon as the object is stored, asserted, or generalized.

---

## 📌 Summary Table

| Scenario              | Excess Check |
| --------------------- | ------------ |
| Fresh object literal  | ❌ Error     |
| Assigned via variable | ✅ Bypassed  |
| Type assertion        | ✅ Bypassed  |
| Index signature       | ✅ Bypassed  |
| `any`                 | ✅ Bypassed  |
| Generic               | ✅ Bypassed  |

---

## 🧠 One-Sentence Summary

> Excess property checks are bypassed whenever an object is no longer a fresh literal or when TypeScript is instructed to trust the developer.