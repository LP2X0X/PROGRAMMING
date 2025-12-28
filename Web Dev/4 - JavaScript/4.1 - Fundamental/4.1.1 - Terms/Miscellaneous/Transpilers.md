---
tags:
  - js
  - term
  - miscellaneous
---

### 🎯 What a transpiler is

A **[transpiler](https://en.wikipedia.org/wiki/Source-to-source_compiler)** is a special piece of software that translates source code to another source code. It can parse (“read and understand”) modern code and rewrite it using older syntax constructs, so that it’ll also work in outdated engines.

> In practice: **modern syntax → older, compatible syntax**

```ad-note
This one rewrite new syntax with old supported syntax.
```

---

### 🧠 Why transpilers exist

Browsers and runtimes do not adopt new language features at the same pace.

Transpilers allow developers to:

- Write modern, expressive code
    
- Target older browsers or environments
    
- Standardize syntax across teams
    

---

### 🧩 What transpilers handle

Transpilers **rewrite syntax**, not runtime behavior.

Examples:

- Arrow functions
    
- Optional chaining (`?.`)
    
- Nullish coalescing (`??`)
    
- Class syntax
    
- JSX
    
- TypeScript types (removed)
    

---

### 🧪 Example

**Input (modern JS)**

```js
const getName = user => user?.profile?.name ?? "Anonymous";
```

**Output (older JS)**

```js
var getName = function (user) {
  return user && user.profile && user.profile.name != null
    ? user.profile.name
    : "Anonymous";
};
```

---

## 🆚 Transpiler vs Polyfill

|Concept|What it does|
|---|---|
|Transpiler|Rewrites syntax|
|Polyfill|Adds missing APIs|

Example:

- `class` → rewritten by a transpiler
    
- `Promise` → provided by a polyfill
    

They are complementary, not interchangeable.

---

## 🛠 Common transpilers

### JavaScript / Frontend

- **Babel**
    
- **SWC**
    
- **esbuild**
    
- **TypeScript compiler (tsc)**
    

### Other ecosystems

- TypeScript → JavaScript
    
- JSX → JavaScript
    
- SCSS → CSS (often called a preprocessor, but conceptually similar)
    

---

## ⚠️ Limitations

- Cannot transpile features that require new runtime behavior
    
- Often needs polyfills alongside it
    
- Adds build complexity
    

---

## 🧠 Mental model

> A transpiler changes **how code looks**, not **what the runtime can do**.

---

## ✅ Key takeaway

Use a transpiler to **write modern syntax safely**, and combine it with polyfills when targeting older environments.