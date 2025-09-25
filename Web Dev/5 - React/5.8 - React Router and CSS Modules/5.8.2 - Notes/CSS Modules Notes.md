---
tags: 
 - css
 - module 
 - note
---

### `:global()`

- Use `:global()` to write **global styles** inside a CSS Module.
    
- Example:
    
    ```css
    :global(body) {
      margin: 0;
      font-family: sans-serif;
    }
    ```
    
- Styles inside `:global()` are **not scoped** and apply everywhere.
    
- ✅ Best for resetting defaults or styling 3rd-party components.

---

### Element Selectors

- You **can** use element selectors (`ul`, `p`, etc.) in CSS Modules.
    
- If written **inside a class**, they stay scoped:
    
    ```css
    .list ul { list-style: none; }
    ```
    
- If written **bare (`ul {…}`)**, they become **global** (not scoped).
    
- ✅ Best practice: always combine element selectors with a class for proper scoping.

---

### Destructuring Styles

- You can **destructure classes** from the imported styles object.
    
- Example:
    
    ```jsx
    import { primary, secondary } from "./Button.module.css";
    // const { primary, secondary } = styles; // You can do it like this also
    <button className={primary}>Click</button>
    ```
    
- Works the same as object destructuring in JS.
    
- ✅ Useful for avoiding long `styles.className` references when using many classes.