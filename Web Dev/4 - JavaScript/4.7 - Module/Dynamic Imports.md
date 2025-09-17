---
tags: 
 - js
 - import
 - module
 - fundamental
---

## 🔹 What is it?

- `import()` is a **function-like expression** (not a keyword like `import` at the top).
    
- It allows you to **load modules dynamically at runtime**.
    
- It always returns a **Promise** that resolves to the module object.
    

---

## 🔹 Syntax

```js
import("./module.js")
  .then(module => {
    // access exports
    module.sayHi();
  })
  .catch(err => console.error("Failed to load module", err));
```

Or with `async/await`:

```js
(async () => {
  const module = await import("./module.js");
  module.sayHi();
})();
```

---

## 🔹 Key Features

1. **Dynamic paths**
    
    ```js
    let path = condition ? "./admin.js" : "./guest.js";
    const mod = await import(path);
    ```
    
2. **Lazy loading**
    
    - Load heavy modules only when needed.
        
    - Improves initial load time.
        
3. **Conditional loading**
    
    - Import only if a feature is required.
        
    
    ```js
    if (user.isAdmin) {
      const admin = await import("./admin-tools.js");
      admin.showPanel();
    }
    ```
    
4. **Code splitting**
    
    - Bundlers (Webpack, Vite, etc.) split code into separate chunks automatically.
        

---

## 🔹 Module Object Returned

The promise resolves to an object with all exports:

```js
// sayHi.js
export function sayHi(name) { console.log(`Hi, ${name}`); }
export const version = "1.0";
export default "Default export";
```

```js
const mod = await import("./sayHi.js");
mod.sayHi("Alice");     // Hi, Alice
console.log(mod.version);  // 1.0
console.log(mod.default);  // "Default export"
```

---

## 🔹 Best Practices & Tips ✅

- Use `import()` for **on-demand features** (dialogs, charts, admin panels).
    
- Always handle errors with `.catch` (network or path issues).
    
- Remember default exports are accessed as `mod.default`.
    
- Avoid overusing dynamic imports for everything → can fragment code unnecessarily.
    
- With bundlers, dynamic import often creates a **separate JS chunk** (lazy loaded).
    

---

👉 So in short:  
**Static `import`** = load at the start (eager, must be at top-level).  
**Dynamic `import()`** = load later (lazy, flexible, returns a promise).