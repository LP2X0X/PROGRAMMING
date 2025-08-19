---
tags:
  - html
  - attribute
---

The `for` attribute is used in **label elements** to bind a label to a specific **form control** by its `id`.

---

### ✅ Example:

```html
<label for="email">Email:</label>
<input type="email" id="email" />
```

- `for="email"` → refers to the `input` with `id="email"`
    
- Clicking the label will **focus** or **activate** the input field
    

---

### 💡 Why Use It?

- Improves **accessibility**
    
- Increases **click area**
    
- Helpful for screen readers and form usability
    

---

### 🧠 Key Rule:

- The value of `for` **must exactly match** the `id` of a form element on the page.
    

---

### ✅ Works With:

- `<input>`
    
- `<select>`
    
- `<textarea>`
    
- `<progress>`, etc.
    

Use the `for` attribute in `<label>` to create accessible, clickable form controls.