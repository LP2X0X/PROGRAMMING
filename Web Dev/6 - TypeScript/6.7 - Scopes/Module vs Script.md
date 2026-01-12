---
tags: 
 - typescript
 - scope
 - note
---

## 1. Conceptual Difference

### Script

A **script** is the traditional execution unit. It assumes a shared global scope and is evaluated in order.

### Module

A **module** is a self-contained unit with its own scope. It explicitly declares dependencies via `import` / `export`.

---

## 2. JavaScript: Script vs Module

### JavaScript Script

**How it is defined**

```html
<script src="app.js"></script>
```

**Characteristics**

- Top-level declarations (`var`, `function`) go to the global scope
    
- `this` at top level refers to `window` (in browsers)
    
- No `import` / `export`
    
- Loaded synchronously by default
    
- Prone to name collisions
    

```js
var count = 0; // global
function increment() {}
```

---

### JavaScript Module

**How it is defined**

```html
<script type="module" src="app.js"></script>
```

**Characteristics**

- File has its own module scope
    
- Must use `import` / `export`
    
- Top-level `this` is `undefined`
    
- Automatically runs in strict mode
    
- Loaded asynchronously (deferred)
    
- Supports tree-shaking
    

```js
export const count = 0;

export function increment() {}
```

---

## 3. TypeScript: Script vs Module

TypeScript follows the **same conceptual model**, but classification is **compile-time–based**.

### TypeScript Script

A `.ts` file is a **script** if it has **no `import` or `export`**.

**Characteristics**

- Declarations are global across all script files
    
- Merges with other scripts in the same compilation
    
- Often used for legacy code or small projects
    

```ts
// script.ts
let count = 0;
```

```ts
// another-script.ts
count++; // OK (same global scope)
```

---

### TypeScript Module

A `.ts` file is a **module** if it contains **any** `import` or `export`.

**Characteristics**

- File has its own scope
    
- No global pollution
    
- Requires explicit imports
    
- Preferred and recommended in modern codebases
    

```ts
// counter.ts
export let count = 0;
```

```ts
// main.ts
import { count } from "./counter";
```

---

## 4. Key Differences at a Glance

| Aspect            | Script        | Module       |
| ----------------- | ------------- | ------------ |
| Scope             | Global        | File-local   |
| `import / export` | ❌            | ✅           |
| Top-level `this`  | `window` (JS) | `undefined`  |
| Strict mode       | ❌            | ✅           |
| Loading           | Synchronous   | Asynchronous |
| Name collisions   | Possible      | Prevented    |
| Tree-shaking      | ❌            | ✅           |

---

## 5. Important TypeScript-Specific Notes

### Global Augmentation (Edge Case)

Modules **can still affect global scope** via explicit declarations:

```ts
export {};

declare global {
  interface Window {
    myAppVersion: string;
  }
}
```

This is intentional and controlled.

---

## 6. Practical Guidance

**Use scripts only if**

- You are working with legacy global-based code
    
- You intentionally want shared globals
    

**Use modules when**

- Writing modern JS or TS (default choice)
    
- Building apps with bundlers (Vite, Webpack, Rollup)
    
- You care about maintainability, scalability, and correctness
    

> In modern TypeScript and JavaScript, **everything should be a module unless you have a strong reason otherwise**.