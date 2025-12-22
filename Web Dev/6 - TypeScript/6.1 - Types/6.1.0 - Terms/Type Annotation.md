---
tags: 
 - typescript
 - term
---

**Type annotation** is the act of **explicitly telling TypeScript what the type of a variable, function parameter, return value, or expression should be**, instead of letting the compiler infer it.

It is how you _declare intent_ to the type system.

---

### Basic Example

```ts
let count: number = 10;
```

Here, `: number` is the type annotation.

---

### Function Annotations

```ts
function add(a: number, b: number): number {
  return a + b;
}
```

- `a: number`, `b: number` → parameter annotations
    
- `: number` → return type annotation
    

---

### Object Annotation

```ts
const user: { name: string; age: number } = {
  name: "Long",
  age: 20,
};
```

This prevents missing or incorrectly typed properties.

---

### When Type Annotations Are Needed

Type annotations are especially important when:

- The value starts empty (`let items: string[] = []`)
    
- The type cannot be inferred correctly
    
- You are defining public APIs or reusable functions
    
- You want to prevent type widening or accidental changes
    

---

### Annotation vs Inference

```ts
let x = 5;        // inferred as number
let y: number = 5; // explicitly annotated
```

Both are valid, but annotations provide **clarity and guarantees**, while inference reduces verbosity.

---

### Key Idea

> **Type inference figures out types; type annotation declares them.**

Used together, they make TypeScript both concise and safe.