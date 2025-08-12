---
tags: css, selector, fundamental
---

### 📖 Definition

A **pseudo selector** (or **pseudo-class** / **pseudo-element**) in CSS is used to style elements based on their state, position, or to style parts of an element that aren't directly accessible in the HTML.

There are **two main types**:

1. **Pseudo-classes** – target elements based on a state or condition (e.g., `:hover`, `:nth-child()`).
    
2. **Pseudo-elements** – style specific parts of an element (e.g., `::before`, `::after`).
    

---

### 🛠️ Syntax

```css
/* Pseudo-class */
selector:pseudo-class {
  /* styles */
}

/* Pseudo-element */
selector::pseudo-element {
  /* styles */
}
```

---

### 💡 Examples

**Pseudo-class example:**

```css
button:hover {
  background-color: lightblue;
}
```

**Pseudo-element example:**

```css
p::first-line {
  font-weight: bold;
}
```

---

### 📝 Tips & Best Practices

- Use pseudo-classes for **interaction** and **state changes**.
    
- Use pseudo-elements for **decorative content** or **selective styling**.
    
- **Best practice**: When using pseudo-elements like `::before` and `::after`, always define the `content` property, even if it's empty. Example:
    

```css
*, *::before, *::after {
  box-sizing: border-box;
}
```

- This ensures **consistent styling** across the element and its generated content.