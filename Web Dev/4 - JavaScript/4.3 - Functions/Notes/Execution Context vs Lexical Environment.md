---
tags: 
 - js
 - note
 - distinguish
---

**[[Web Dev/4 - JavaScript/4.3 - Functions/4.3.5 - Scopes/Execution Context|Execution Context]]** and **[[Lexical Environment]]** are related but not the same. The difference is mainly about **scope vs lifecycle**.

---

## Execution Context

An **execution context** is the **runtime container** created when code starts running.

It includes:

- The **lexical environment**
    
- The **`this` binding**
    
- The **lifecycle** of execution (creation → execution → destruction)
    

Created:

- Once for global code
    
- Every time a function is called
    

Destroyed:

- When execution finishes (function return / program end)
    

> Think: _“When and how this code is executing.”_

---

## Lexical Environment

A **lexical environment** is a **scope structure** that stores:

- Variable and function bindings (`let`, `const`, `var`, functions)
    
- A reference to the **outer lexical environment**
    

Created:

- When code is **written and parsed** (lexically determined)
    
- Instantiated when the execution context is created
    

Preserved:

- As long as it is referenced (closures)
    

> Think: _“What variables are visible here.”_

---

## Key Differences

| Aspect   | Execution Context        | Lexical Environment              |
| -------- | ------------------------ | -------------------------------- |
| Purpose  | Manages execution        | Manages scope                    |
| Contains | Lexical env + `this`     | Bindings + outer link            |
| Created  | When code starts running | When a scope is entered          |
| Lifetime | Tied to execution        | Can outlive execution (closures) |
| Affects  | Call stack               | Scope chain                      |

---

## Why Closures Matter Here

```js
function outer() {
  let x = 10;
  return function inner() {
    console.log(x);
  };
}

const fn = outer();
fn(); // 10
```

- `outer`’s **execution context** is destroyed
    
- `outer`’s **lexical environment** is preserved
    
- `inner` closes over that environment
    

This is why they must be distinct concepts.

---

## One-Line Summary

> **Execution context controls _when_ code runs; lexical environment controls _what_ the code can see.**