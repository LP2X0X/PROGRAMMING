---
tags: 
 - typescript
 - term
 - fundamental
---

### 🔹 What is a label?

A **label** in TypeScript is a **descriptive name attached to a type position** to improve readability and tooling support.

Labels exist **only in the type system**.

```ad-warning
_labels_ exist for **type-level function signatures** (function types, arrow function types)
```

---

## 📌 Where labels appear

### 1️⃣ Tuple element labels

```ts
type Range = [start: number, end: number];
```

- `start` and `end` are **labels**
    
- The real types are still just `number`
    

Equivalent to:

```ts
type Range = [number, number];
```

---

### 2️⃣ Function parameter labels (type-level)

```ts
type Comparator = (a: number, b: number) => number;
```

- `a` and `b` are labels
    
- Used only for documentation and IntelliSense
    

---

### 3️⃣ Function signature types

```ts
type Handler = (event: MouseEvent) => void;
```

`event` is a label — not enforced at runtime.

---

## 🧠 What labels do

✔ Improve IntelliSense  
✔ Improve error messages  
✔ Act as documentation  
✔ Have zero runtime cost

---

## ❌ What labels do NOT do

✘ Do not create variables  
✘ Do not affect runtime behavior  
✘ Do not create object properties  
✘ Do not allow named access

---

## ⚠️ Common misconception

```ts
type T = [x: number, y: number];

const p: T = [10, 20];
p.x; // ❌ Error
```

Labels do **not** create named fields.

---

## 📋 Labels vs similar concepts

| Concept                | Runtime | Named access |
| ---------------------- | ------- | ------------ |
| Tuple labels           | ❌      | ❌           |
| Object properties      | ✅      | ✅           |
| Type aliases           | ❌      | ❌           |
| Destructured variables | ✅      | ✅           |

---

## 🧩 Why labels exist

Without labels:

```ts
type Rect = [number, number];
```

With labels:

```ts
type Rect = [width: number, height: number];
```

The second version is **self-explanatory**.

---

## 🧠 Mental model

> Labels are **type-level comments** enforced by the compiler.

---

## ✅ When to use labels

- Public APIs
    
- Library types
    
- Tuples with semantic meaning
    
- Function signatures used across files
    

---

## 🚫 When not needed

- Trivial tuples
    
- Internal throwaway types
    

---

### Final takeaway

Labels increase **clarity**, not **capability**.  
They exist to help humans and tools, not the runtime.