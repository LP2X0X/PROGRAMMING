---
tags: 
 - js
 - data
 - type
 - note
---

### Two Categories of Values

JavaScript has **primitive values** and **object values**. For historical and interoperability reasons, **most primitives have corresponding wrapper object types**.

---

### Primitive Types

Primitives represent **raw values**.

- `string`
    
- `number`
    
- `boolean`
    
- `bigint`
    
- `symbol`
    
- `null`
    
- `undefined`
    

**Characteristics**

- Immutable
    
- Compared by value
    
- Lightweight
    
- Preferred in almost all cases
    

---

### Wrapper Object Types

Wrapper objects provide **object-based interfaces** for primitives.

- `String`
    
- `Number`
    
- `Boolean`
    
- `BigInt`
    
- `Symbol`
    

Created via constructors:

```ts
new String("text");
new Number(42);
```

**Characteristics**

- Objects (allocated in memory)
    
- Compared by reference
    
- Rarely needed
    
- Can introduce bugs
    

---

### Why Wrapper Types Exist

They exist to allow **method access on primitives**.

```ts
"abc".toUpperCase();
```

Behind the scenes:

1. Primitive is temporarily wrapped
    
2. Method is called
    
3. Wrapper is discarded
    

This process is called **autoboxing**.

---

### TypeScript Recommendation

Always use **primitive types** in type annotations:

```ts
let name: string;
let count: number;
let isActive: boolean;
```

Avoid wrapper types unless interacting with legacy APIs or doing advanced meta-programming.

---

### Key Pitfall

```ts
new Boolean(false) === false; // false
```

Objects are truthy, even when wrapping falsy values.

---

### Rule of Thumb

> **Use primitives for data, wrappers only when you explicitly need object identity or prototype behavior.**

---

### Runtime vs Type System

- Primitives exist at runtime
    
- Wrapper types exist at runtime
    
- Type annotations disappear at runtime
    
- TypeScript allows both but **encourages primitives**
    

---

### Summary

- Primitives are values
    
- Wrappers are objects
    
- Autoboxing bridges the two
    
- Prefer primitives for correctness, performance, and clarity

---

### References

https://stackoverflow.com/questions/17256182/what-is-the-difference-between-string-primitives-and-string-objects-in-javascrip