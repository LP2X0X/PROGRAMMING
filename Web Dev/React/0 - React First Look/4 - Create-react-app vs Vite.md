---
tags:
  - react
  - introduction
---

![[Pasted image 20250628153700.png|center]]

---

The reason **Create React App (CRA)** installs **so many node_modules** is due to how modern JavaScript development works, especially with tooling and dependency trees.

---

### **📦 Why So Many node_modules ?**

#### **1. Tooling + Transpiling**

CRA sets up tools like:

- **Webpack** – for bundling
- **Babel** – for transpiling modern JavaScript/JSX
- **ESLint** – for linting
- **PostCSS / Autoprefixer** – for CSS processing
    These tools **each have many dependencies**, and those dependencies have _their own_ dependencies.

---

#### **2. Deep Dependency Chains**

In JavaScript, even a small feature (like color formatting or logging) might live in its own package. So a single tool can pull in **dozens to hundreds** of small modules.


Example:

```
webpack → loader-utils → big.js → ...
```

---

#### **3. Cross-platform Support**

Packages often include polyfills or workarounds for:

- Old browsers
- Different operating systems
    That increases package size and count.

---

#### **4. Loose Dependency Trees**

Unlike other ecosystems (e.g. Go or Rust), npm allows many versions of the same package to coexist. So you might have:

```
lodash@4.17.15
lodash@4.17.21
```

…used by different tools, increasing total packages.

---

#### **5. CRA Tries to Be Zero-Config**

CRA gives you:

- Babel
- Webpack
- Fast Refresh
- JSX support
- CSS Modules
- Test runner (Jest)
- Environment variable support
- TypeScript (optional)

…without needing to configure any of it — but the tradeoff is lots of **under-the-hood dependencies**.

---

### **💡 Want a lighter setup?**

- Use [**Vite**](https://vitejs.dev/) instead — it’s much leaner and faster.
- Or roll your own minimal React setup with just Babel + Webpack/Vite.
