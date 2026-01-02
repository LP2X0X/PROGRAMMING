---
tags: 
 - typescript
 - keyword
---

## 1️⃣ Runtime significance (JavaScript)

```ts
const x = 10;
```

### What `const` guarantees

- The **binding cannot be reassigned**
    
- The variable must be **initialized**
    
- Scope is **block-level**
    

```ts
x = 20; // ❌ Error
```

### What `const` does **not** guarantee

```ts
const obj = { a: 1 };
obj.a = 2; // ✅ allowed
```

`const` does **not** make values immutable.

---

## 2️⃣ [[Type Inference|Type inference]] significance (TypeScript)

This is where `const` really matters in TS.

### `let` vs `const`

```ts
let a = "home";   // type: string
const b = "home"; // type: "home"
```

Why:

- `let` → value can change → widened type
    
- `const` → value cannot change → **[[Literal|literal type]]**
    

---

## 3️⃣ Literal type preservation

```ts
const status = "loading";
// type: "loading"
```

This enables:

- Discriminated unions
    
- Exact comparisons
    
- Safer APIs
    

---

## 4️⃣ `as const` (const assertion)

```ts
const config = {
  mode: "dark",
  retry: 3,
} as const;
```

Effect:

- Properties become **readonly**
    
- Values become **literal types**
    

Result:

```ts
{
  readonly mode: "dark";
  readonly retry: 3;
}
```

---

## 5️⃣ `const` with arrays and tuples

```ts
const arr = [1, 2, 3];
// number[]

const tuple = [1, 2, 3] as const;
// readonly [1, 2, 3]
```

`as const` prevents widening **and** tuple decay.

---

## 6️⃣ `const` and control-flow narrowing

```ts
const kind = "a";

if (kind === "a") {
  // always true
}
```

TS can reason more strongly about constants.

---

## 7️⃣ `const` vs `readonly`

| Feature      | `const`  | `readonly` |
| ------------ | -------- | ---------- |
| Scope        | Variable | Property   |
| Runtime      | Yes      | No         |
| Reassignment | ❌       | ❌         |
| Mutation     | ✅       | ❌         |

They solve **different problems**.

---

## Mental model

> **`const` freezes the binding; TypeScript uses that guarantee to infer narrower, safer types.**

---

## Summary

| Aspect         | Effect                   |
| -------------- | ------------------------ |
| Runtime        | No reassignment          |
| Type inference | Prevents widening        |
| Literal types  | Preserved                |
| `as const`     | Deep readonly + literals |

---

## One-line takeaway

> **In TypeScript, `const` is not just about safety at runtime—it directly enables more precise type inference.**