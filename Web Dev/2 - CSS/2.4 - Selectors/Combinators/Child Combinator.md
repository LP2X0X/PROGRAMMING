---
tags:
  - css
  - selector
  - fundamental
---

### 📌 Description

The **child combinator** (`>`) selects elements that are **direct children** of a specified element.

- It **does not** select grandchildren or deeper descendants.
    
- The relationship is **parent → child only**.
    

---

### 📍 Syntax

```css
A > B {
  /* styles */
}
```

- `A` = **parent** selector.
    
- `B` = **direct child** selector.
    
- `>` = **child combinator**.
    

---

### 📍 Example

HTML:

```html
<div>
  <p>First paragraph (direct child of div)</p>
  <section>
    <p>Second paragraph (grandchild of div)</p>
  </section>
</div>
```

CSS:

```css
div > p {
  color: red;
}
```

✅ This will style **only the first paragraph** because it is a **direct child** of `<div>`.  
🚫 The second paragraph is ignored because it is a **grandchild**.

---

### ⚠️ Notes & Best Practices

- Use `>` when you **only want to target immediate children**, avoiding unintended deeper matches.
    
- This can help **keep styles more specific and predictable** compared to the descendant combinator ( ).
    
- Be careful when HTML structure changes — a direct child may become a grandchild and lose styles.
    