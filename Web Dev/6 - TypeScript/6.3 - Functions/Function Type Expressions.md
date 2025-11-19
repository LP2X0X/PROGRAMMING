---
tags: 
 - typescript
 - function
 - type
 - expression
---

A **function type expression** in TypeScript is a way to _describe the type of a function_—its parameters and its return type—**without actually defining the function itself**.

Think of it as a _type annotation_ that tells TypeScript:

> “A function with this structure must accept these parameters and return this type.”

---

# ✅ Basic Form

A function type expression looks like this:

```ts
let fn: (a: number, b: number) => string;
```

Meaning:

- The function takes two parameters:
    
    - `a` is a number
        
    - `b` is a number
        
- It must return a string.
    

You can assign any function that matches this signature:

```ts
fn = (x, y) => `${x + y}`;
```

---

# 🧠 Why It’s Called _Function Type Expression_

Because it is an _expression_ that describes a _type_, not executable code.

This is a **type-level function signature**, not a function definition.

---

# Example with Parameters and Return Type

```ts
type Greeter = (name: string) => string;

const greet: Greeter = (n) => `Hello, ${n}`;
```

`Greeter` is a **function type expression**.

---

# Function Type Expression vs. Function Declaration

|Concept|Example|Meaning|
|---|---|---|
|**Function Type Expression**|`(x: number) => number`|_Describes_ a function’s shape|
|**Function Definition**|`function add(x: number) { return x + 1; }`|_Implements_ the function|

---

# Why It's Useful

✔ Reusable function types  
✔ Enforcing consistent signatures  
✔ Typing callbacks  
✔ Typing higher-order functions

Example: callback type

```ts
function repeat(n: number, action: (index: number) => void) {
  for (let i = 0; i < n; i++) action(i);
}
```

Here, `(index: number) => void` is a **function type expression**.

---

# Summary

A **function type expression** is:

- A way to describe the type of a function
    
- Written using the arrow syntax at the type level
    
- Used for variables, parameters, callbacks, and type aliases
    

It's essentially:

> A blueprint of what a function must look like.