---
tags: 
 - typescript
 - type
 - utility
---

## 🧠 What `ReturnType<T>` Is

`ReturnType<T>` is a **built-in TypeScript utility type** that **extracts the return type of a function type**.

```ts
type R = ReturnType<() => number>;
// R = number
```

It works **purely at the type level** and has **no runtime effect**.

---

## ⚙️ Formal Definition (Conceptual)

Under the hood, it is defined approximately as:

```ts
type ReturnType<T> =
  T extends (...args: any[]) => infer R ? R : any;
```

Key ideas:

- Uses **conditional types**
    
- Uses **`infer`** to capture the return type
    
- Works only on **function types**
    

---

## 🧩 Basic Usage

### 🎯 Extract Return Type from a Function Type

```ts
type Fn = () => string;

type Result = ReturnType<Fn>;
// string
```

---

### 🎯 Extract from a Named Function

```ts
function getUser() {
  return { id: 1, name: "Alice" };
}

type User = ReturnType<typeof getUser>;
```

Why `typeof`?

- Functions exist at runtime
    
- Types operate at compile time
    
- `typeof` bridges runtime → type space
    

---

## 🧠 Why `ReturnType` Is Useful

### 📌 Prevents Type Duplication

Without `ReturnType`:

```ts
type User = {
  id: number;
  name: string;
};
```

With `ReturnType`:

```ts
type User = ReturnType<typeof getUser>;
```

Now types **stay in sync** automatically.

---

### 📌 Ideal for Factory Functions

```ts
function createConfig() {
  return {
    env: "prod",
    debug: false,
  };
}

type Config = ReturnType<typeof createConfig>;
```

---

### 📌 Works Well with Hooks

```ts
const useAuth = () => ({
  user: null,
  login() {},
});

type Auth = ReturnType<typeof useAuth>;
```

Common in React codebases.

---

## ⚠️ Important Constraints

---

## 🚫 Must Be a Function Type

```ts
type T = ReturnType<string>; // ❌ error
```

Correct:

```ts
type T = ReturnType<() => string>;
```

---

## 🧱 Overloaded Functions (Caveat)

```ts
function fn(x: string): string;
function fn(x: number): number;
function fn(x: any) {
  return x;
}

type R = ReturnType<typeof fn>; 
// R = string | number
```

ReturnType produces a **union** of all overload returns.

---

## 🔁 Async Functions

```ts
async function fetchUser() {
  return { id: 1 };
}

type R = ReturnType<typeof fetchUser>;
// Promise<{ id: number }>
```

To unwrap:

```ts
type User = Awaited<ReturnType<typeof fetchUser>>;
```

---

## 🧠 ReturnType vs Explicit Return Types

### Explicit Return Type

```ts
function sum(a: number, b: number): number {
  return a + b;
}
```

### Derived Return Type

```ts
type SumResult = ReturnType<typeof sum>;
```

**Best practice:**

- Use explicit return types for **public APIs**
    
- Use `ReturnType` for **internal reuse**
    

---

## 🧠 ReturnType in Generic Functions

```ts
function wrap<T>(value: T) {
  return { value };
}

type WrappedString = ReturnType<typeof wrap<string>>;
```

But often better:

```ts
type Wrapped<T> = { value: T };
```

---

## ⚠️ Common Mistakes

### ❌ Forgetting `typeof`

```ts
type R = ReturnType<getUser>; // ❌
```

Correct:

```ts
type R = ReturnType<typeof getUser>;
```

---

### ❌ Using ReturnType Where Simpler Types Work

```ts
type Flag = ReturnType<() => boolean>;
```

Better:

```ts
type Flag = boolean;
```

Use it where it adds **real value**.

---

## 🧠 Interaction with Conditional Types

```ts
type Fn<T> = () => T;

type R = ReturnType<Fn<number>>;
// number
```

ReturnType itself **is a conditional type**.

---

## 🧠 Mental Model

> `ReturnType<T>` is a **type-level mirror** of what a function returns at runtime.

---

## 🧠 Comparison with Related Utility Types

| Utility           | Purpose                 |
| ----------------- | ----------------------- |
| `ReturnType<T>`   | Extract return type     |
| `Parameters<T>`   | Extract parameter types |
| `Awaited<T>`      | Unwrap Promise          |
| `InstanceType<T>` | Extract class instance  |

---

## 🧠 One-Sentence Summary

> `ReturnType<T>` extracts the return type of a function type using conditional types and inference, helping keep types consistent without duplication.