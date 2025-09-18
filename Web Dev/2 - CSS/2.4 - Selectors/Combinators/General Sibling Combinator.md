---
tags:
  - css
  - selector
  - fundamental
---

### 📌 Description

The **general sibling combinator** (`~`) selects elements that share the **same parent** and appear **after** a specified element — **not necessarily immediately after**.

- Think of it as: **"any following sibling"**.
    
- The matched elements must have the **same parent** as the first selector.
    

---

### 📍 Syntax

```css
A ~ B {
  /* styles */
}
```

- `A` = **reference element**.
    
- `B` = **target sibling element(s)** appearing **after** `A`.
    
- `~` = **general sibling combinator**.
    

---

### 📍 Example

HTML:

```html
<div>
  <h2>Title</h2>
  <p>First paragraph</p>
  <p>Second paragraph</p>
  <span>Not a paragraph</span>
</div>
```

CSS:

```css
h2 ~ p {
  color: blue;
}
```

✅ Both paragraphs will be blue because they appear **after** `<h2>` and share the **same parent `<div>`**.  
🚫 The `<span>` will not be styled because it’s not a `<p>`.

---

### ⚠️ Notes & Best Practices

- Use when you need to style **all later siblings** of a certain type after a given element.
    
- Avoid overusing it with large DOM structures — it can match **many elements** unintentionally.
    
- For just **the next sibling**, use the **Adjacent Sibling Combinator (`+`)** instead.
    