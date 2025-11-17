---
tags: 
 - js
 - module
 - fundamental
---

## 🔹 1. Types of Export

```ad-important
So, remember, `import` needs curly braces for named exports and doesn’t need them for the default one.
```

### a) Named Export

You can export multiple things by name.

```js
// math.js
export const PI = 3.1415;
export function add(a, b) { return a + b; }
export class Circle {}
```

Usage:

```js
// app.js
import { PI, add } from './math.js';
console.log(PI);          // 3.1415
console.log(add(2, 3));   // 5
```

- **Can rename on import**:
    

```js
import { PI as PiValue } from './math.js';
```

- **Can export inline or later**:
    

```js
function multiply(a, b) { return a * b; }
export { multiply };

// 📁 say.js
function sayHi(user) {
  alert(`Hello, ${user}!`);
}

function sayBye(user) {
  alert(`Bye, ${user}!`);
}

export {sayHi, sayBye}; // a list of exported variables
```

- **Can export using destructuring**:
    

```js
export const { createCustomer, updateName } = customerSlice.actions; // const is important here
```

```ad-note
No semicolons after export class/function

Please note that export before a class or a function does not make it a function expression. It’s still a function declaration, albeit exported.
```

---

### b) Default Export

A module can have **only one default export**.

```js
// logger.js
export default function log(msg) {
  console.log("LOG:", msg);
}
```

Usage:

```js
import log from './logger.js';
log("hello");
```

- The imported name is **your choice**, not tied to the export name (since the exported entity may have no name).
    
```js
import whatever from './logger.js';
```

- You can also rename with `as`:
    
```js
import {default as myLogger} from './logger.js';
```

---

### c) Mixed Export

A module can have both default and named exports.

```js
// user.js
export default class User {}
export const ROLE = "admin";
```

Usage:

```js
import User, { ROLE } from './user.js';
```

---

## 🔹 2. Import All

Import everything into a single namespace object.

```js
import * as math from './math.js';

console.log(math.PI);
console.log(math.add(2, 3));
```

---

## 🔹 3. Re-export (Aggregation)

Re-export from another module without importing into local scope.

```js
// shapes.js
export { Circle } from './math.js';
export * from './utils.js'; // re-export all named exports
```

- With default exports, must rename:
    

```js
export { default as Logger } from './logger.js';
```

```ad-note
The notable difference of `export ... from` compared to `import/export` is that re-exported modules aren’t available in the current file.
```

---

## 🔹 4. Dynamic Import

Import modules at runtime (returns a promise).

```js
if (condition) {
  const module = await import('./math.js');
  console.log(module.add(2, 3));
}
```

Useful for:

- Lazy loading
    
- Conditional loading
    
- Code splitting
    

---

## 🔹 5. Live Bindings

Exports are **live references**, not copies.

```js
// counter.js
export let count = 0;
export function increment() { count++; }
```

```js
// app.js
import { count, increment } from './counter.js';
console.log(count); // 0
increment();
console.log(count); // 1 (updated!)
```

⚠️ You cannot reassign imported bindings:

```js
count = 5; // ❌ Error
```

---

## 🔹 6. Importing for Side Effects Only

```js
import './polyfill.js'; // runs the file, nothing imported
```

---

## 🔹 7. The “default” Name in Default Exports

When you use a **default export**, the exported value is internally stored under the special name **`default`**.

Example:

```js
// math.js
export default function add(a, b) {
  return a + b;
}
```

This is equivalent to:

```js
// internally
export { function add(a, b) { return a + b; } as default };
```

#### 🔍 What this means

- On import, you don’t have to use the actual function/class/variable name.
    
- You choose the name yourself:
    

```js
import sum from './math.js';   // works
import anything from './math.js'; // also works
```

- Under the hood, the module really has a binding called `default`.
    

#### ⚡ Gotcha

If you mix **default export** with **named export**, you must explicitly reference `"default"` when re-exporting:

```js
// re-export default
export { default as MathAdder } from './math.js';
```

---

## 🔹 8. Best Practices & Tips ✅

1. **Prefer Named Exports**
    
    - Easier auto-complete in IDEs.
        
    - Safer refactoring (import names must match).
        
    - Clearer when reading code.
        
2. **Use Default Export when**
    
    - Module clearly exports **one main thing** (e.g., a React component).
        
3. **Organize Re-exports**
    
    - Use an `index.js` (barrel file) to re-export modules:
        
        ```js
        export * from './math.js';
        export * from './user.js';
        ```
        
        Then consumers can:
        
        ```js
        import { add, User } from './index.js';
        ```
        
4. **Don’t Overuse `import * as`**
    
    - Useful for namespaces, but can clutter (`math.add`, `math.PI`).
        
    - Prefer direct imports for tree-shaking efficiency.
        
5. **Tree-shaking friendly**
    
    - Named exports allow bundlers (like Webpack, Rollup) to remove unused code.
        
    - Default exports make tree-shaking less precise.
        
6. **Consistent Style**
    
    - Pick a style (named vs default) and apply it consistently across your project.
        
7. **Avoid Circular Dependencies**
    
    - A → B and B → A imports can cause partial initialization.
        
    - Refactor shared code into a third module.
        
8. **Dynamic Imports for Optimization**
    
    - Use `import()` to load heavy modules only when needed (e.g., charts).
        
9. **Module Pathing**
    
    - Always use relative paths (`./`, `../`) unless you configure aliases.
        
    - Don’t forget file extensions in browser ESM (`.js`).
        
10. **Remember Modules are Deferred**
    
    - `<script type="module">` always runs after HTML is parsed.
        
    - No need to place script tags at the end of `<body>`.
        

---

## 🔍 Quick Reference

```js
// Export
export const a = 1;
export default function main() {}
export { b, c as renamedC };

// Import
import x from './mod.js';
import { a, renamedC } from './mod.js';
import * as mod from './mod.js';
import './mod.js'; // side effect only
```