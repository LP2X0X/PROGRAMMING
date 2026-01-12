---
tags: 
 - typescript
 - term
---

In TypeScript, **"Ambient"** refers to declarations that describe code that exists **outside** of your current project's compilation context.

Think of "ambient" as meaning **"existing in the surrounding environment."**

When you write ambient declarations, you are telling TypeScript: _"I am not creating this code right now, but I promise it will be there when the application runs (usually from a browser, a library, or the server)."_

Here is the breakdown of what makes a declaration "ambient":

### 1. The Core Concept

- **Regular TypeScript:** You write code, and the compiler turns it into JavaScript.
    
- **Ambient TypeScript:** You write type definitions, and the compiler **erases** them. They generate _zero_ JavaScript. They exist purely to describe the shape of existing things so the compiler doesn't yell at you.
    

### 2. The Best Analogy: A Menu vs. The Meal

- **The Implementation (Regular Code):** This is the actual meal. It has substance and nutrition.
    
- **The Ambient Declaration:** This is the **menu**. It describes the meal (name, ingredients, price), but you cannot eat the menu.
    

If you have a menu (ambient declaration) but the kitchen is empty (no runtime code), you will go hungry (your app will crash at runtime).

### 3. Where you see Ambient Declarations

You usually encounter "ambient" concepts in two places:

#### A. Ambient Context (Global Variables)

Variables that exist globally in the browser or Node.js environment.

- _Example:_ `window`, `document`, `console`, `process`.
    
- TypeScript comes with `lib.d.ts` files that act as ambient declarations for standard browser features. This is why you can type `document.getElementById` without importing anything—it is "ambiently" available.
    

#### B. Ambient Modules (`.d.ts` files)

When you install a library that is written in plain JavaScript (e.g., an older version of lodash or jQuery), TypeScript doesn't understand it.

- You install `@types/library-name` (DefinitelyTyped).
    
- These files contain **Ambient Module Declarations**. They describe the exports of the library so you can import them, even though the library source code isn't in TypeScript.
    

### 4. Comparison: Normal vs. Ambient

| **Feature**  | **Regular Declaration**             | **Ambient Declaration**                                    |
| ------------ | ----------------------------------- | ---------------------------------------------------------- |
| **Keyword**  | `const`, `let`, `class`, `function` | `declare`, `declare global`                                |
| **Output**   | Compiles to JavaScript code         | **Removed completely** from output                         |
| **Purpose**  | To create logic and data            | To describe **existing** logic/data                        |
| **Location** | `.ts` or `.tsx` files               | Usually `.d.ts` files                                      |
| **Risk**     | Logic errors                        | Runtime crash if the described item doesn't actually exist |

### Example

**Regular Code (Implementation):**

```ts
// math.ts
// This creates a function and compiles to JS
export function add(a: number, b: number) {
  return a + b;
}
```

**Ambient Code (Description):**

```ts
// math.d.ts
// This creates NO code. It just tells TS that 'add' exists somewhere else.
declare function add(a: number, b: number): number;
```

### Summary

If a variable is **"ambient,"** it means:

1. It is defined elsewhere (browser, external lib).
    
2. It is available in the scope without explicit import (usually).
    
3. TypeScript needs a `declare` statement to know about its types.
    
