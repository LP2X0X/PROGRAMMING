---
tags: 
 - typescript
 - type
 - utility
---

## 🧠 What `Awaited<T>` Is

`Awaited<T>` is a **built-in TypeScript utility type** that **unwraps the resolved value of a Promise**.

```ts
type R = Awaited<Promise<number>>;
// number
```

It models **exactly what `await` does at runtime**, but **at the type level**.

---

## 🎯 The Problem It Solves

### Without `Awaited`

```ts
async function fetchUser() {
  return { id: 1 };
}

type R = ReturnType<typeof fetchUser>;
// Promise<{ id: number }>
```

But when you `await` it:

```ts
const user = await fetchUser();
// user is { id: number }
```

**Type mismatch:**  
`ReturnType` gives the Promise, but runtime gives the resolved value.

---

### With `Awaited`

```ts
type User = Awaited<ReturnType<typeof fetchUser>>;
// { id: number }
```

Now the type **matches runtime behavior**.

---

## ⚙️ How `Awaited<T>` Works (Conceptually)

Simplified definition:

```ts
type Awaited<T> =
  T extends PromiseLike<infer U>
    ? Awaited<U>
    : T;
```

Key ideas:

- Uses **conditional types**
    
- Uses **`infer`**
    
- **Recursively unwraps** nested Promises
    

---

## 🔁 Recursive Unwrapping

```ts
type A = Awaited<Promise<Promise<string>>>;
// string
```

This mirrors:

```ts
await await promise;
```

---

## 🧩 Works with `PromiseLike`, Not Just `Promise`

```ts
type CustomPromise = {
  then(onfulfilled: (value: number) => any): any;
};

type R = Awaited<CustomPromise>;
// number
```

This aligns with the **JavaScript spec** for `await`.

---

## 🧠 Common Real-World Use Cases

---

## ⚛️ With Async Functions

```ts
async function getData() {
  return { ok: true };
}

type Data = Awaited<ReturnType<typeof getData>>;
```

This is the **most common and recommended pattern**.

---

## ⚛️ With React Query / Data Fetching

```ts
const fetchCabins = async () => {
  return api.get("/cabins");
};

type Cabins = Awaited<ReturnType<typeof fetchCabins>>;
```

---

## 🧪 With Generic Helpers

```ts
async function resolve<T>(value: T): Promise<T> {
  return value;
}

type R = Awaited<ReturnType<typeof resolve<string>>>;
// string
```

---

## 🧠 Awaited vs Manual Unwrapping

### ❌ Manual

```ts
type R = Promise<{ id: number }> extends Promise<infer U> ? U : never;
```

### ✅ Built-in

```ts
type R = Awaited<Promise<{ id: number }>>;
```

Cleaner and future-proof.

---

## ⚠️ Important Constraints

---

## 🚫 Non-Promise Types

```ts
type R = Awaited<number>;
// number
```

No error. It returns the type as-is.

---

## ⚠️ `Awaited` Does NOT Make Code Async

```ts
type R = Awaited<Promise<string>>;
```

This affects **types only**.  
It does **not** add `await` at runtime.

---

## 🧠 Awaited vs `.then()`

At runtime:

```ts
promise.then(value => value);
```

At type level:

```ts
Awaited<typeof promise>
```

Same conceptual behavior.

---

## 🧠 Why `Awaited` Was Introduced

Before TS 4.5:

- Promise unwrapping was inconsistent
    
- Libraries implemented their own helpers
    
- Async typing was error-prone
    

`Awaited<T>` standardized async typing.

---

## 🧠 Relationship with Other Utility Types

| Utility           | Role                    |
| ----------------- | ----------------------- |
| `ReturnType<T>`   | Extract function return |
| `Awaited<T>`      | Unwrap Promise          |
| `Parameters<T>`   | Extract args            |
| `InstanceType<T>` | Extract class instance  |

Very commonly combined:

```ts
type Result = Awaited<ReturnType<typeof fn>>;
```

---

## 🧠 Mental Model

> `Awaited<T>` is the **type-level version of JavaScript’s `await` keyword**.

---

## 🧠 One-Sentence Summary

> `Awaited<T>` recursively unwraps Promises to produce the same type you get after using `await` at runtime.