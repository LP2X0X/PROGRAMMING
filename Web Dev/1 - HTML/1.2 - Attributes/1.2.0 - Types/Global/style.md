---
tags: 
  - html
  - attribute
  - global
  - fundamental
---

### style

The `style` attribute lets you apply **inline CSS styles** directly to an HTML element.

---

### 💡 Key Points:

* Applies **only to the element it's written on**
* Uses **CSS syntax**, but inside double quotes
* Not ideal for large projects — use **external CSS** for maintainability

---

### ✅ Example:

```html
<p style="color: blue; font-size: 18px;">
  This text is blue and 18px.
</p>
```

---

### 🔍 When to Use:

* Quick testing or demo purposes
* Dynamic styling in JavaScript:

  ```js
  element.style.backgroundColor = "red";
  ```

---

### ⚠️ Avoid Overuse:

* Inline styles override external styles
* Harder to maintain and scale

---

Use `style` for **quick, one-off styling**, but prefer **CSS classes** or external stylesheets for clean code.
