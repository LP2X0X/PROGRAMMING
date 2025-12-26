---
tags: 
 - typescript
 - term
---

### What it is 🔍

- A **type predicate** is a special return type used in a function to tell TypeScript:
    

> “If this function returns `true`, then this value has this type.”

```ts
param is Type
```

---

### Where it’s used 📍

- **Only in function return types**
    
- Most commonly in **custom type guards**
    

---

### Basic example 🧪

```ts
function isNumber(value: unknown): value is number {
  return typeof value === "number";
}
```

- `value is number` → **type predicate**
    
- The function becomes a **type guard**
    

---

### Effect: type narrowing 📉

```ts
function print(value: unknown) {
  if (isNumber(value)) {
    // value is narrowed to number
    console.log(value.toFixed(2));
  }
}
```

🧠 TypeScript trusts the predicate inside the `if` block.

---

### Predicate syntax breakdown 🧠

```ts
value is number
│     │
│     └─ narrowed type
└────── parameter name (must match function param)
```

⚠️ The parameter name **must be the same** as the function argument.

---

### With object types 🧱

```ts
type User = { name: string };

function isUser(obj: unknown): obj is User {
  return (
    typeof obj === "object" &&
    obj !== null &&
    "name" in obj
  );
}
```

✔ Safe structural validation  
✔ Runtime + compile-time benefit

---

### Using predicates in array methods 📚

```ts
const mixed = [1, "a", 2, "b"];

const numbers = mixed.filter(
  (x): x is number => typeof x === "number"
);
// numbers: number[]
```

🔥 Very common and powerful use case.

---

### `asserts` vs `is` 🆚

```ts
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error("Not a string");
  }
}
```

|`is`|`asserts`|
|---|---|
|Narrows in conditionals|Narrows after function call|
|Returns boolean|Returns `void`|
|Needs `if`|Throws on failure|

---

### Common mistakes 🚧

❌ Predicate without real check

```ts
function isUser(x: any): x is User {
  return true; // unsafe
}
```

❌ Wrong parameter name

```ts
function isUser(x: any): value is User {} // invalid
```

---

### When to use a type predicate 🎯

- You receive `unknown` or wide unions
    
- Built-in narrowing is not enough
    
- You need reusable, explicit runtime checks
    

---

### Summary 📝

- Type predicate = `param is Type`
    
- Used in function return types
    
- Enables **custom type guards**
    
- Drives **type narrowing**
    
- Bridges runtime checks with static typing
    