---
tags: 
 - typescript
 - type
 - custom
---

## 🧩 `DistributiveOmit` Explained

```ts
type DistributiveOmit<T, K extends PropertyKey> =
  T extends any ? Omit<T, K> : never;
```

---

## What this type is doing (high level)

This type **forces `Omit` to distribute over a union type**.

Normally:

- `Omit` works on **`T` as a whole**
    
- It does **not** apply to each union member separately
    

This construct changes that behavior.

---

## Why `T extends any ? ... : ...` matters

### 1️⃣ This is a **conditional type**

```ts
T extends any ? X : Y
```

In TypeScript:

- **Conditional types distribute over unions**
    
- If `T` is `A | B | C`, the conditional runs once per member
    

So this becomes:

```ts
(A extends any ? X : Y)
| (B extends any ? X : Y)
| (C extends any ? X : Y)
```

---

## Why `extends any` specifically?

Because **every type extends `any`**.

That means:

- The condition is **always true**
    
- The `: never` branch is never chosen
    
- The only purpose is to **trigger distribution**
    

> `extends any` is a **distribution trick**, not a logical test.

---

## Step-by-step expansion example

```ts
type A = { a: string; shared: number };
type B = { b: string; shared: number };

type U = A | B;
```

### Apply `DistributiveOmit`

```ts
type Result = DistributiveOmit<U, "shared">;
```

### Expand the conditional

```ts
(A extends any ? Omit<A, "shared"> : never)
| (B extends any ? Omit<B, "shared"> : never)
```

### Evaluate `Omit`

```ts
| { a: string }
| { b: string }
```

✅ Distribution achieved.

---

## What happens without this wrapper

```ts
type Bad = Omit<U, "shared">;
```

### Evaluation

```ts
keyof (A | B) // "shared"
```

Result:

```ts
{}
```

❌ Union structure is lost.

---

## Why `: never` is used

In distributive conditional types:

- `never` disappears from unions
    

```ts
A | never === A
```

So it is the correct “do nothing” fallback.

---

## Pattern classification

| Aspect      | Meaning                                     |
| ----------- | ------------------------------------------- |
| Pattern     | Distributive conditional wrapper            |
| Purpose     | Force per-member evaluation                 |
| Used with   | `Omit`, `Pick`, `Partial`, custom utilities | 
| Common name | “Distributive helper”                       |

---

## Equivalent alternative forms

```ts
type DistributiveOmit<T, K extends PropertyKey> =
  T extends unknown ? Omit<T, K> : never;
```

`any` and `unknown` behave the same **for distribution** here.

---

## One-line takeaway

> **`T extends any ? Omit<T, K> : never` is a deliberate conditional-type trick used to force `Omit` to run on each union member individually.**
