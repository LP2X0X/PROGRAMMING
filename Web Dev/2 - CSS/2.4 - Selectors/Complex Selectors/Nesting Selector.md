---
tags:
  - css
  - selector
  - fundamental
---

### 📌 Description

Nesting selectors allow you to **write CSS rules inside other rules**, keeping related styles together and avoiding repetitive selectors.  
This is common in **preprocessors** like **Sass/SCSS** and is now partially supported in **native CSS** (with the `&` syntax).

---

### 📍 Syntax (Native CSS Nesting)

```css
article {
  color: black;

  & h2 {
    color: teal;
  }

  & p {
    font-size: 1.2rem;
  }
}
```

✅ `&` refers to the **parent selector** in the nesting block.

---

### 📍 Example with Pseudo-Classes & Media Queries

```css
button {
  background: lightblue;

  &:hover {
    background: steelblue;
  }

  @media (max-width: 600px) {
    font-size: 14px;
  }
}
```

✅ Keeps hover states and responsive rules **inside the button block** for readability.

---

### 📍 Example with Preprocessors (Sass/SCSS)

```scss
nav {
  ul {
    list-style: none;
  }
  li {
    display: inline-block;
  }
  a {
    text-decoration: none;
  }
}
```

✅ This compiles to standard CSS without nesting.

---

### 📌 Why Use Nesting?

- **Cleaner & more maintainable** code.
    
- Avoids **selector repetition**.
    
- Groups related rules together for better readability.
    

---

### ⚠️ Best Practices

- Don’t nest too deeply (max 3 levels recommended).
    
- Keep selectors **short and meaningful**.
    
- Be careful with specificity — nested selectors can make rules harder to override.
    