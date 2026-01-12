---
tags: 
 - typescript
 - syntax
---

## `is` keyword in TypeScript ([[Type Predicate]]) 🧠

### What it is

- `is` is used to define a **user-defined type guard**
    
- It tells TypeScript how to **narrow a type** based on a boolean function
    
- Works only in **return types of functions**
    

---

### Basic syntax ✍️

```ts
function isX(value: A | B): value is A {
  return /* boolean condition */
}
```

- `value is A` is called a **type predicate**
    
- If the function returns `true`, TS treats `value` as type `A`
    

---

### Why it exists 🎯

- Built-in narrowing (`typeof`, `instanceof`, `in`) is limited
    
- `is` lets **you define your own narrowing logic**
    
- Essential for complex unions and domain-specific checks
    

---

### Simple example ✅

```ts
function isString(x: unknown): x is string {
  return typeof x === "string";
}
```

Usage:

```ts
if (isString(value)) {
  value.toUpperCase(); // value is string here
}
```

---

### How TypeScript uses it 🔍

- Outside the function: just a `boolean`
    
- Inside control flow: **type narrowing happens**
    

```ts
if (isUser(data)) {
  // data is User
} else {
  // data is not User
}
```

---

### `is` vs normal boolean return ⚠️

Without `is`:

```ts
function isUser(x: any): boolean { ... }
```

❌ No narrowing

With `is`:

```ts
function isUser(x: any): x is User { ... }
```

✅ Narrowing works

---

### Common use cases 🛠️

- Discriminating union types
    
- Validating API responses
    
- Runtime checks for complex objects
    
- Replacing unsafe type assertions (`as`)
    

---

### `is` vs `as` (important distinction) ⚖️

| `is`          | `as`             |
| ------------- | ---------------- |
| Runtime check | No runtime check |
| Type-safe     | Can lie          |
| Narrows types | Forces types     |

Prefer `is` when safety matters.

---

### Works well with arrays 📦

```ts
const strings = values.filter(isString);
// strings: string[]
```

---

### Limitations 🚫

- Only valid in **return position**
    
- Must return a boolean
    
- Logic must actually be correct (TS trusts you)
    

---

### Mental model 🧩

> "`is` teaches TypeScript how to understand my runtime check."

---

### One-line summary 📝

**The `is` keyword defines custom type guards that let TypeScript safely narrow types based on your own logic.**
