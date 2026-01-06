---
tags: 
 - typescript
 - note
 - const
 - enum
---

Below is a **clear, detailed note** explaining **why `const enum` is generally discouraged in TypeScript**, especially in modern toolchains.

---

## 🚫 What Is `const enum` (Quick Recap)

```ts
const enum Direction {
  Up,
  Down,
}

const d = Direction.Up;
```

TypeScript **inlines the enum values at compile time**:

```js
const d = 0;
```

No enum object exists at runtime.

---

## ⚠️ Why You Should Generally Avoid `const enum`

---

## 🧨 Breaks Isolated / Incremental Builds

`const enum` **requires full type information** at compile time.

That means it **does not work reliably** with:

- `isolatedModules`
    
- Babel
    
- SWC
    
- esbuild
    
- Vite
    
- ts-jest (without special config)
    

Most modern toolchains **do not support `const enum` safely**.

---

## 🔥 Incompatible with Babel & Non-tsc Transpilers

Babel **does not understand `const enum` semantics**.

```ts
const enum Status {
  Ready,
}
```

Babel cannot inline this value correctly → **build errors or wrong output**.

This is one of the **main reasons it is discouraged today**.

---

## 🧩 No Runtime Representation

Because `const enum` is erased:

```ts
Object.values(Direction); // ❌ impossible
```

You cannot:

- Reflect on values
    
- Iterate
    
- Serialize
    
- Debug easily
    

This removes useful runtime behavior.

---

## 🧠 Harder to Debug

In output JS:

```js
const state = 2;
```

Instead of:

```js
const state = Status.Loading;
```

This makes:

- Stack traces less readable
    
- Logs harder to understand
    
- Debugging more error-prone
    

---

## ⚠️ Fragile Across Package Boundaries

If you export a `const enum` from a library:

```ts
export const enum Role {
  Admin,
}
```

Consumers **must compile with the same TS settings**.

This is brittle and unsafe for:

- Shared libraries
    
- Monorepos
    
- Public packages
    

---

## 🧱 TypeScript Team Recommendation

The TypeScript team has effectively **moved away from encouraging `const enum`**.

They now recommend:

- Regular `enum`
    
- Or `as const` objects
    

---

## ✅ Preferred Alternatives

---

## 🟢 `as const` Object (Best Modern Option)

```ts
const Direction = {
  Up: 0,
  Down: 1,
} as const;

type Direction = typeof Direction[keyof typeof Direction];
```

✔ Tree-shakable  
✔ Runtime available  
✔ Tooling-friendly  
✔ Debuggable

---

## 🟢 Regular `enum` (If You Need Runtime)

```ts
enum Direction {
  Up,
  Down,
}
```

✔ Runtime object  
✔ Safer for libraries  
❌ Slightly more JS output

---

## 📊 Comparison Summary

| Feature          | `const enum` | `enum` | `as const` |
| ---------------- | ------------ | ------ | ---------- |
| Runtime object   | ❌           | ✅     | ✅         |
| Babel compatible | ❌           | ✅     | ✅         |
| Debuggable       | ❌           | ⚠️     | ✅         |
| Tree-shakable    | ✅           | ❌     | ✅         |
| Library-safe     | ❌           | ⚠️     | ✅         |

---

## 🧠 When `const enum` MAY Be Acceptable

- Single project
    
- `tsc` only
    
- No Babel / SWC / Vite
    
- No shared libraries
    
- Performance-critical edge cases
    

Even then, it must be **deliberate and documented**.

---

## 🧠 Mental Model

> `const enum` optimizes **by deleting itself**, and that deletion causes more problems than it solves in modern builds.

---

## 🧠 One-Sentence Summary

> You should generally avoid `const enum` because it breaks modern build tools, removes runtime safety, complicates debugging, and is fragile across projects—use `as const` or regular `enum` instead.
