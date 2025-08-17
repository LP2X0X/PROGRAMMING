---
tags: css, selector, fundamental
---

### 📌 Description

The **descendant combinator** (a **space** between selectors) selects elements that are **inside another element** at **any nesting depth**.

- It matches **children, grandchildren, great-grandchildren**, etc.
    
- The relationship is **ancestor → descendant**, not necessarily **direct child**.
    

---

### 📍 Syntax

```css
A B {
  /* styles */
}
```

- `A` = **ancestor** selector.
    
- `B` = **descendant** selector.
    
- Space between `A` and `B` = **descendant combinator**.
    

---

### 📍 Example

HTML:

```html
<div>
  <p>First paragraph (child of div)</p>
  <section>
    <p>Second paragraph (grandchild of div)</p>
  </section>
</div>
```

CSS:

```css
div p {
  color: blue;
}
```

✅ This will style **both paragraphs** because they are descendants of `<div>`, no matter how deep.

---

### ⚠️ Notes & Best Practices

- **Overusing** descendant combinators can hurt performance and make styles harder to maintain.
    
- If you only need **direct children**, use the **child combinator** (`>`).
    
- Keep the chain short for better readability:  
    ❌ Bad:
    
    ```css
    body main section div ul li a { ... }
    ```
    
    ✅ Better:
    
    ```css
    section a { ... }
    ```
    