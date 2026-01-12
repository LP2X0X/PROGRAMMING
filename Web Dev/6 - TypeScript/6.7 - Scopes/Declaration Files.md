---
tags: 
 - typescript
 - file
---

In TypeScript, a **declaration file** (`.d.ts`) describes the **shape of code**—types, interfaces, functions, classes—**without providing implementations**. It exists purely for **type checking and tooling**, not runtime execution.

```ad-summary
**Declaration files (`.d.ts`) are used to describe the public type shape of existing JavaScript or TypeScript code so the compiler can type-check usage without emitting runtime code.**
```

---

## 1. What a Declaration File Is

- File extension: **`.d.ts`**
    
- Contains **only type-level constructs**
    
- Emits **no JavaScript**
    
- Used to provide types for:
    
    - JavaScript libraries
        
    - Native APIs
        
    - Global variables
        
    - Generated type output for consumers
        

```ts
// math.d.ts
export function add(a: number, b: number): number;
```

---

## 2. Two Kinds of Declaration Files

### 2.1 Global Declaration Files (Script-style)

These **add names to the global scope**.

```ts
// globals.d.ts
declare const VERSION: string;

declare function log(message: string): void;
```

Usage (no import needed):

```ts
log("hello");
```

**Key properties**

- No `import` or `export`
    
- Acts like a script
    
- Common in older codebases
    

---

### 2.2 Module Declaration Files (Module-style)

These describe a **module’s public API**.

```ts
// math.d.ts
export function add(a: number, b: number): number;
export const PI: number;
```

Usage:

```ts
import { add } from "./math";
```

**Key properties**

- Contains `export`
    
- File is a module
    
- Preferred for modern libraries
    

---

## 3. `declare` Keyword

`declare` tells TypeScript:

> “This exists at runtime, but I am not defining it here.”

```ts
declare const process: {
  env: Record<string, string>;
};
```

No JS is emitted.

---

## 4. Declaring Existing JavaScript Code

### For a JS file

```js
// legacy.js
function greet(name) {
  return "Hello " + name;
}
```

```ts
// legacy.d.ts
export function greet(name: string): string;
```

---

## 5. Module Declarations for Third-Party Libraries

### Declaring an untyped npm package

```ts
// untyped-lib.d.ts
declare module "untyped-lib" {
  export function run(): void;
}
```

This enables:

```ts
import { run } from "untyped-lib";
```

---

## 6. Ambient vs Non-Ambient Declarations

| Term               | Meaning                           |
| ------------------ | --------------------------------- |
| **Ambient**        | Uses `declare`, no implementation | 
| **Non-ambient**    | Has implementation (normal `.ts`) |
| **Global ambient** | Adds to global scope              |
| **Module ambient** | Declares a module                 |

---

## 7. `.d.ts` and Modules vs Scripts

| File type                | Scope               |
| ------------------------ | ------------------- |
| `.d.ts` with `export`    | Module              |
| `.d.ts` without `export` | Global script       |
| `.d.ts` with `export {}` | Module (no globals) |

```ts
// force-module.d.ts
export {};
```

Useful to **prevent accidental global declarations**.

---

## 8. Declaration Generation

TypeScript can **generate `.d.ts` files** from `.ts`:

```json
{
  "compilerOptions": {
    "declaration": true
  }
}
```

Emits:

```
dist/
  index.js
  index.d.ts
```

This is how libraries ship types.

---

## 9. Where Declaration Files Are Used

- `@types/*` packages (DefinitelyTyped)
    
- Library distribution
    
- Framework global typings (`dom`, `node`)
    
- Project-wide globals (`env.d.ts`, `global.d.ts`)
    

---

## 10. Common Pitfalls

### Accidental Globals

```ts
// bad.d.ts
interface User {
  id: string;
}
```

This becomes **global**. To avoid:

```ts
export {};

interface User {
  id: string;
}
```

---

## 11. One-Sentence Summary

> A **`.d.ts` file defines the type contract of existing code—global or modular—without producing any runtime JavaScript.**
