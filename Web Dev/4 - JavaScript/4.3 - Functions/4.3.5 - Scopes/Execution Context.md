---
tags: 
  - js
  - term
  - advanced
---

In JavaScript, an **execution context** is the abstract environment in which code is **evaluated and executed**. It defines **what variables, functions, and objects are accessible**, and **how `this` is resolved**, at any given point during execution.

You can think of it as the _runtime container_ for code.

---

## Types of Execution Context

JavaScript has three main kinds:

### 1. Global Execution Context (GEC)

Created when the JavaScript file first runs.

Characteristics:

- Creates the **global object**
    
    - Browser: `window`
        
    - Node.js: `global`
        
- Sets `this` to the global object (in non-module scripts)
    
- Allocates memory for:
    
    - Global variables
        
    - Global function declarations
        

There is **only one** global execution context.

---

### 2. Function Execution Context (FEC)

Created **every time a function is invoked**.

Each function call gets its **own** execution context.

Contains:

- Function arguments
    
- Local variables
    
- Inner function declarations
    
- A local `this` value
    

Example:

```js
function foo(a) {
  let b = 10;
  bar();
}

function bar() {
  console.log('bar');
}

foo(5);
```

Execution contexts created:

1. Global Execution Context
    
2. Execution context for `foo`
    
3. Execution context for `bar`
    

---

### 3. Eval Execution Context (rare)

Created when code is executed inside `eval()`.  
Generally discouraged and rarely used in real-world code.

---

## Phases of an Execution Context

Each execution context is created in **two phases**.

### 1. Creation Phase (Memory Allocation)

Before any code runs:

- Variables are allocated memory and initialized
    
    - `var` → `undefined`
        
    - `let` / `const` → uninitialized (Temporal Dead Zone)
        
- Function declarations are fully hoisted
    
- Scope chain is established
    
- `this` is determined
    

Example:

```js
console.log(x); // undefined
var x = 5;
```

---

### 2. Execution Phase

- Code runs line by line
    
- Variables are assigned actual values
    
- Functions are executed
    

---

## Execution Context Stack (Call Stack)

Execution contexts are managed using a **stack** (LIFO).

Example:

```js
function a() {
  b();
}

function b() {
  c();
}

function c() {
  console.log('done');
}

a();
```

Call stack evolution:

```
Global
→ a()
→ b()
→ c()
```

When `c()` finishes, its context is popped off the stack, then `b()`, then `a()`.

---

## What an Execution Context Contains (Conceptually)

Each execution context has:

- **Variable Environment**
    
    - Variables (`var`)
        
    - Function declarations
        
- **Lexical Environment**
    
    - `let`, `const`
        
    - Reference to outer scope
        
- **`this` binding**
    

(Modern specs merge these concepts, but this mental model is still useful.)

---

## Relation to Closures (Important)

Closures work because **execution contexts retain access to their lexical environment**, even after the function has returned.

```js
function outer() {
  let count = 0;
  return function inner() {
    count++;
    return count;
  };
}

const fn = outer();
fn(); // 1
fn(); // 2
```

`inner` still has access to `count` because the lexical environment of `outer` is preserved.

---

## Short Summary

- **Execution context** = environment where JS code runs
    
- Created for:
    
    - Global code
        
    - Each function call
        
- Has two phases:
    
    - Creation (hoisting, scope setup)
        
    - Execution (actual code runs)
        
- Managed via the **call stack**
    
- Fundamental to understanding:
    
    - Hoisting
        
    - `this`
        
    - Scope
        
    - Closures
        
