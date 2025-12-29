---
tags:
  - js
  - term
  - miscellaneous
---

### Context
New language features may include not only syntax constructs and operators, but also built-in functions.

For example, `Math.trunc(n)` is a function that “cuts off” the decimal part of a number, e.g `Math.trunc(1.23)` returns `1`.

In some (very outdated) JavaScript engines, there’s no `Math.trunc`, so such code will fail.

As we’re talking about new functions, not syntax changes, there’s no need to transpile anything here. We just need to declare the missing function.

A script that updates/adds new functions is called “polyfill”. It “fills in” the gap and adds missing implementations.

---

### 🎯 What a polyfill is

A **polyfill** is code that **adds support for a feature that a browser does not natively support**, so newer web features can work in older or unsupported environments.

> In short: **it “fills the gap” between modern code and older browsers.**

```ad-note
This one add the missing code which support new feature.
```

---

### 🧠 Why polyfills exist

Web standards evolve faster than browsers.

Polyfills allow developers to:

- Use modern APIs
    
- Maintain backward compatibility
    
- Avoid writing multiple code paths
    

---

### 📦 What polyfills typically cover

- JavaScript APIs
    
- DOM APIs
    
- Some CSS features (via preprocessing)
    

**Examples**

- `Promise`
    
- `fetch`
    
- `Array.prototype.includes`
    
- `Element.closest`
    

---

### 🧪 Example (JavaScript polyfill)

```js
if (!Array.prototype.includes) {
  Array.prototype.includes = function (value) {
    return this.indexOf(value) !== -1;
  };
}
```

If the browser already supports `includes`, this code does nothing.

---

## 🧩 Polyfills vs Transpilation

| Concept    | Purpose           |
| ---------- | ----------------- |
| Polyfill   | Adds missing APIs |
| Transpiler | Rewrites syntax   |

Example:

- **Babel** transpiles `?.` into older syntax
    
- **core-js** polyfills `Promise`, `Map`, etc.
    

They often work together.

---

## 🎨 CSS polyfills (important nuance)

CSS cannot truly be polyfilled at runtime.

Instead, tools:

- **Transform CSS** at build time
    
- **Inject fallbacks**
    

Examples:

- Autoprefixer
    
- postcss-preset-env
    

These are _polyfill-like_, but not true runtime polyfills.

---

## ⚠️ Limitations

- Cannot polyfill syntax the browser cannot parse
    
- Some APIs cannot be fully polyfilled (e.g. layout engines)
    
- Adds bundle size
    

---

## 🧠 Mental model

> A polyfill is a **compatibility layer**, not a rewrite.

---

## ✅ Key takeaway

Use polyfills when you want to **write modern code without breaking older browsers**, but be intentional—polyfills have a performance and size cost.