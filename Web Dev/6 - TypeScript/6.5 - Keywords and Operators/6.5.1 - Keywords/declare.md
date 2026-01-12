---
tags: 
 - typescript
 - keyword
---

In TypeScript, the `declare` keyword is used to create **ambient declarations**.

It tells the TypeScript compiler: _"Trust me, this variable, function, or module exists at runtime, and here is its type structure."_

This is essential when you are using JavaScript libraries that don't have their own type definitions, or when you are accessing global variables (like those injected by a browser script or a server environment) that TypeScript doesn't automatically know about.

**Crucially, code marked with `declare` does not compile into any JavaScript output.** It is used purely for type-checking during development.

---

### 1. Declaring Global Variables

If you are using a library loaded via a CDN (e.g., jQuery, Google Maps) in your HTML, TypeScript won't know about the global variables it creates. You use `declare` to prevent "Variable not found" errors.

```ts
// Tells TS: "There is a global variable named 'myLibrary' with this shape"
declare const myLibrary: {
  init: () => void;
  version: string;
};

// Now you can use it without errors, and with autocomplete
myLibrary.init();
```

### 2. Declaring Functions and Classes

You can declare the signatures of functions or classes that exist in the environment but are not imported.

```ts
// Declaring a global function
declare function greet(name: string): void;

// Declaring a global class
declare class Analytics {
  constructor(id: string);
  trackEvent(name: string): void;
}

// Usage
greet("Alice");
const tracker = new Analytics("UA-123");
```

### 3. Declaring Modules (Wildcards)

This is most commonly seen when you try to import non-code files (like images or CSS) in a TypeScript project (e.g., React or Vue). TypeScript doesn't know how to handle a `.png` file by default.

You can create a declaration file (usually `global.d.ts` or `declarations.d.ts`) and add:

```ts
// Allows you to import logo from './logo.png'
declare module '*.png' {
  const value: string;
  export default value;
}

// Allows you to import styles from './styles.css'
declare module '*.css';
```

### 4. Module Augmentation

You can use `declare` to add properties to existing types or libraries. A common use case is adding custom properties to the global `Window` object.

```ts
// Inside a module or .ts file
declare global {
  interface Window {
    __REDUX_DEVTOOLS_EXTENSION__: any;
    myCustomConfig: { apiKey: string };
  }
}

// Now valid
window.myCustomConfig.apiKey = "12345";
```

### Summary Table: When to use `declare`

| **Scenario**                                 | **Usage Example**                             |
| -------------------------------------------- | --------------------------------------------- |
| **Global variable via CDN**                  | `declare var $: any;`                         |
| **Non-JS import**                            | `declare module '*.svg';`                     | 
| **Extending built-in types**                 | `declare global { interface Window { ... } }` |
| **Fixing "Could not find declaration file"** | `declare module 'some-untyped-lib';`          |

---

### Important Caveats

- **No Runtime Code:** If you write `declare const x = 10;`, TypeScript will not generate `const x = 10;` in your JavaScript file. If `x` does not actually exist at runtime, your app will crash.
    
- **Use `.d.ts` files:** While you can use `declare` in any `.ts` file, it is best practice to place these declarations in a dedicated `.d.ts` file (e.g., `types/global.d.ts`) to keep your logic separate from your type definitions.
    