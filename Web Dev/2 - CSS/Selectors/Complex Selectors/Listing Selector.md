---
tags:
  - css
  - selector
  - fundamental
---

## 🔹 Listing Selector

A **listing selector** (also called a **grouping selector**) is used when you want to apply the _exact same CSS declarations_ to multiple, different selectors at once.  
Instead of repeating the same rules for each selector separately, you list them together, separated by commas **`,`**.

### 📌 Syntax

```css
selector1, selector2, selector3 {
  /* shared styles */
}
```

- **selector1, selector2, selector3** → any valid selectors (element, class, ID, attribute, etc.).
    
- **comma** → separates each selector in the list.
    
- The **same rule block** is applied to all listed selectors.
    

---

### 📍 Example: Applying same style to multiple tags

```css
h1, h2, h3 {
  font-family: Arial, sans-serif;
  color: #333;
}
```

✅ All `<h1>`, `<h2>`, and `<h3>` elements will use Arial and have dark gray text.

---

### 📍 Example: Mixing selector types

```css
p, .highlight, #main-title {
  margin: 0;
  padding: 0;
}
```

✅ All `<p>` elements, all elements with `.highlight`, and the element with `id="main-title"` will have no margin or padding.

---

### 📌 Best Practices

1. **Use grouping to avoid repetition** — makes CSS shorter and easier to maintain.
    
2. **Keep lists short & meaningful** — grouping 20 unrelated selectors together can make debugging harder.
    
3. **Format long lists on multiple lines** for readability:
    
    ```css
    h1,
    h2,
    h3,
    p {
      color: #444;
    }
    ```
    

---

💡 **Tip:** Grouping selectors does **not** combine their specificity — each selector keeps its own specificity. If one selector is more specific, it will still win in conflicts.