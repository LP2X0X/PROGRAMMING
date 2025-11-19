---
tags: 
 - css
 - module
---

**CSS Modules** are designed to **scope styles locally** to a specific component.  
But sometimes you _do_ need **global styles** — for example:

- Resetting default browser styles
    
- Setting base typography, colors, or layout rules
    
- Importing 3rd-party CSS libraries (like normalize.css or Tailwind)
    

You can absolutely do this — here’s how 👇

---

## 🧩 1️⃣ Global CSS File (Recommended)

Create a normal `.css` file (not `.module.css`) for your global styles.  
Then import it once — usually in `src/index.js` or `src/main.jsx`.

### Example:

```bash
src/
├── index.js
├── App.jsx
├── App.module.css
└── global.css
```

**global.css**

```css
/* global.css */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', sans-serif;
  background-color: #f8f9fa;
}
```

**index.js**

```js
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import "./global.css"; // ← global styles imported here

ReactDOM.createRoot(document.getElementById("root")).render(<App />);
```

✅ These styles apply **globally** across the entire app.

---

## 💅 2️⃣ Global Selectors Inside a CSS Module (using `:global`)

Sometimes you want to define a _few_ global rules _inside_ a CSS Module file.

### Example:

**App.module.css**

```css
.container {
  padding: 20px;
  background: white;
}

/* this affects global <h1> elements */
:global(h1) {
  color: #333;
  font-size: 2rem;
}

/* or a specific global class */
:global(.btn) {
  padding: 10px 20px;
  background: #007bff;
  color: white;
}
```

✅ `:global(...)` makes those selectors global.  
Everything else stays locally scoped.

---

## 🌐 3️⃣ Mixing Global + Module Styles

You can combine them like this:

```jsx
import styles from "./App.module.css";
import "./global.css";

export default function App() {
  return (
    <div className={styles.container}>
      <h1>Welcome</h1>
      <button className="btn">Click</button>
    </div>
  );
}
```

- `.container` → scoped (from CSS Module)
    
- `.btn` → global (from global.css or `:global(.btn)`)
    

---

## ⚙️ 4️⃣ In Next.js or Vite (important note)

Most bundlers (like **Vite**, **CRA**, **Next.js**) follow this rule:

- `.css` → **global styles**
    
- `.module.css` → **scoped styles**
    

So if you name the file `something.module.css`, its styles are local by default.  
If you name it just `something.css`, it’s global automatically.

---

## ✅ Summary

|Need|Use|
|---|---|
|App-wide reset, layout, typography|Regular `global.css` imported once|
|A few global overrides inside module|`:global(selector)` syntax|
|Scoped component styles|`.module.css`|
|Global + module together|Import both in component|