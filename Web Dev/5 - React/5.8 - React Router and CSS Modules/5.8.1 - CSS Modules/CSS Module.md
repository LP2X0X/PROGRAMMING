---
tags: 
 - react
 - library
 - css
 - style
---

### ✅ What is a CSS Module?

- A **CSS Module** is just a `.module.css` (or `.module.scss`) file.
    
- When you import it into a component, the classes inside are **scoped locally** by default.
    
- This prevents **class name conflicts** across components.
    
- They’re a **built-in feature** of most modern React build tools, such as:
	- **Create React App (CRA)**
	- **Next.js**
	- **Vite**
	- **Parcel**

---

## 🔧 How CSS Modules Work

- Normally in CSS, class names are global. If two files use `.btn`, they will conflict.
    
- With CSS Modules, class names are **transformed** into unique identifiers (like `btn__3Ghjf`).
    
- This scoping happens during the build (via webpack or Vite).
    

Example:

**Button.module.css**

```css
.btn {
  background-color: blue;
  color: white;
  padding: 10px 15px;
  border-radius: 5px;
}
```

**Button.jsx**

```jsx
import React from "react";
import styles from "./Button.module.css"; // import CSS module

export default function Button() {
  return <button className={styles.btn}>Click Me</button>;
}
```

➡️ The `btn` class is converted into something like `Button_btn__3Ghjf`, making it unique.

---

## 📌 Features of CSS Modules

1. **Scoped by default**
    
    - No global conflicts (unlike regular CSS files).
        
2. **Class composition**
    
    - You can compose one class from another.
        
    
    ```css
    .primary {
      background: blue;
      color: white;
    }
    
    .rounded {
      border-radius: 10px;
    }
    
    .primaryRounded {
      composes: primary rounded;
      padding: 10px 20px;
    }
    ```
    
3. **Mix with global styles if needed**
    
    - You can still use global styles for resets, fonts, etc.
        
    - Just use a normal `.css` file (not `.module.css`).
        
4. **Dynamic styling**
    
    - Apply multiple classes using template literals:
        
    
    ```jsx
    <div className={`${styles.card} ${isActive ? styles.active : ""}`}>
      Card Content
    </div>
    ```
    

---

## 🛠️ Best Practices

- Use **CSS Modules for component-level styles**.
    
- Keep **global CSS** only for resets, typography, and theme variables.
    
- Combine with **utility frameworks** (like Tailwind) if needed.
    
- Follow **naming conventions** (`ComponentName.module.css`).
    

---

## ⚖️ Pros vs Cons

**✔ Pros**

- Scoped styles (no conflicts).
    
- Simple, still using pure CSS.
    
- Works with Sass too (`.module.scss`).
    
- No runtime overhead (compiles at build time).
    

**❌ Cons**

- Slightly more verbose (`styles.btn` instead of `"btn"`).
    
- Harder to share styles globally across many components.
    
- Not as dynamic as **CSS-in-JS**.
    

---

## 📚 When to Use CSS Modules

- Small/medium projects.
    
- Apps where you want **simple styling without conflicts**.
    
- If you prefer **CSS syntax over JS styling**.
    
- When you don’t want the extra overhead of CSS-in-JS.
    