---
tags:
  - css
  - selector
  - fundamental
---

### 📌 Description

The **adjacent sibling combinator** (`+`) selects an element that is the **immediately following sibling** of another element.

- Think of it as: **"the very next sibling"**.
    
- The two elements must have the **same parent**.
    
- If there’s any other element in between, it **won’t match**.
    

---

### 📍 Syntax

```css
A + B {
  /* styles */
}
```

- `A` = **reference element**.
    
- `B` = **target element** directly after `A`.
    
- `+` = **adjacent sibling combinator**.
    

---

### 📍 Example

HTML:

```html
<div>
  <h2>Title</h2>
  <p>First paragraph</p>
  <p>Second paragraph</p>
</div>
```

CSS:

```css
h2 + p {
  color: green;
}
```

✅ Only the **first paragraph** after `<h2>` will be green.  
🚫 The second `<p>` won’t be styled because it’s not **directly next to** `<h2>`.

---

### ⚠️ Notes & Best Practices

- Use when you need to target the **first element after** a certain one.
    
- Often used for spacing adjustments between elements (`margin-top`, etc.).
    
- If you want **all following siblings**, use the **General Sibling Combinator (`~`)** instead.
    