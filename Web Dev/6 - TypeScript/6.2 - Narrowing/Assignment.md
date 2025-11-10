---
tags: 
 - typescript
 - narrowing
---

In TypeScript, when we assign a value to a variable, TypeScript **looks at the right-hand side of the assignment** and **narrows the type of the left-hand side** appropriately.

---

## Example 1: Assigning number or string

```ts
let x = Math.random() < 0.5 ? 10 : "hello world!";

let x: string | number;

x = 1; // ✅ valid
console.log(x);

x = "goodbye!"; // ✅ valid
console.log(x);
```

**Explanation:**

- Even though `x` was narrowed to `number` after the first assignment, we can still assign a `string` to it.
    
- This is because the **declared type** of `x` (`string | number`) allows both `string` and `number`.
    
- Assignability is always checked against the **declared type**, not the narrowed type.
    

---

## Example 2: Invalid assignment

```ts
let x = Math.random() < 0.5 ? 10 : "hello world!";

let x: string | number;

x = true; // ❌ Error: Type 'boolean' is not assignable to type 'string | number'.
console.log(x);
```

**Explanation:**

- `boolean` is not part of the declared type (`string | number`), so TypeScript raises an error.
    

---

### Key Points

- **Declared type**: The type the variable starts with (e.g., `string | number`).
    
- **Narrowed type**: TypeScript temporarily narrows the type based on the value assigned.
    
- Assignments must always be **compatible with the declared type**, not just the current narrowed type.
    