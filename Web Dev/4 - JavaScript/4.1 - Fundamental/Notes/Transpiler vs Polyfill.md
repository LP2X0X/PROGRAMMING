---
tags: 
 - js
 - note
 - distinguish
---

Both are tools used to make modern JavaScript run on older browsers/environments, but they solve different parts of the problem.

### The Quick Summary

- **Transpiler:** Transforms **Syntax**. It rewrites your code (like `=>` functions) into older code (like `function`) before the browser ever sees it.
    
- **Polyfill:** Adds **Functionality**. It is a piece of code that "fills in" missing browser features (like `Array.prototype.includes`) at runtime.
    

---

### 1. Transpiler (Source-to-Source Compiler)5

A transpiler runs **during development/build time**. It takes modern syntax that an old browser would consider "syntax errors" and translates it into an equivalent older syntax.
- **What it handles:** Syntax changes (Arrow functions, Classes, Const/Let, Spread operator).
    
- **Famous Example:** Babel, TypeScript.
    
- **Analogy:** It is like a **Translator**. If you speak Modern English but your grandparents only understand Old English, the translator rewrites your letter into Old English before mailing it.
    

Example:

You write (Modern Syntax):
```js
[1, 2, 3].map(n => n + 1);
```

_Transpiler outputs (Old Syntax):_
```js
[1, 2, 3].map(function(n) {
  return n + 1;
});
```

### 2. Polyfill

A polyfill runs **at runtime** (in the browser). It detects if a specific feature is missing and, if so, adds a custom implementation of that feature using standard JavaScript.
- **What it handles:** New Objects and Methods (`Promise`, `fetch`, `Array.includes`, `Object.entries`).12
    
- **Famous Example:** `core-js`, `polyfill.io`.
    
- **Analogy:** It is like **adding a ramp**. If a building (browser) lacks a wheelchair ramp (feature), you build a temporary one so people can still get in.
    

Example:

You write (Modern Feature):
```js
// Old browsers don't know what .includes() is
if (!Array.prototype.includes) {
  // The Polyfill adds it manually
  Array.prototype.includes = function(searchElement) {
    // ... logic to find element ...
  };
}
```

---

### Comparison Table

| **Feature**                | **Transpiler**                                          | **Polyfill**                                              |
| -------------------------- | ------------------------------------------------------- | --------------------------------------------------------- |
| **Primary Job**            | Rewrites **Syntax**                                     | Implements **Methods/APIs**                               |
| **When it runs**           | **Compile Time** (Build step)                           | **Runtime** (In the user's browser)                       |
| **Output**                 | New file with different code                            | Same file, but patches global objects                     |
| **Can it fix everything?** | No. It cannot teach an old browser what a `Promise` is. | No. It cannot make an old browser understand `=>` syntax. |
| **Examples**               | Arrow functions, Classes, Async/Await syntax            | `fetch`, `Promise`, `Map`, `Set`                          |

### Which one do you need?

In modern development, you almost always need **both**.

1. You use a **Transpiler** (like Babel) so you can write clean, modern syntax (`const`, `=>`, `class`).13
    
2. You import **Polyfills** (like `core-js`) so your code can use modern functions (`Promise.all`, `[].includes`) without crashing on older devices.14