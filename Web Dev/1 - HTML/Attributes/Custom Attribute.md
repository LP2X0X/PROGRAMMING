---
tags: html, custom, attribute
---

**Custom attributes** in HTML are attributes **you define yourself**, usually using the `data-` prefix, and are commonly called **data attributes**.

---

### ✅ Syntax:

```html
<div data-user-id="42" data-role="admin">John Doe</div>
```

- `data-user-id` and `data-role` are **custom attributes**.
    
- They don’t affect how the page looks but can store **extra info** for use in JavaScript.
    

---

### 🔍 Accessing in JavaScript:

```js
const user = document.querySelector('[data-user-id]');
console.log(user.dataset.userId); // "42"
console.log(user.dataset.role);   // "admin"
```

- Accessed via `.dataset` object
    
- Converts `data-user-id` → `userId` (camelCase)
    

---

### ⚠️ Rules:

- Must start with `data-` to be valid in HTML5
    
- No spaces or special characters (except hyphens)
    

---

### ✅ Use Cases:

- Pass data to JS without inline scripts
    
- Store settings, IDs, types, or flags
    
- Useful in interactive UI elements like dropdowns, tabs, modals
    

---

### ❌ Invalid (non-data) Custom Attributes:

```html
<div my-custom="value"></div> <!-- ❌ Not valid HTML5 -->
```

These are not guaranteed to work across browsers and **fail validation**.

---

Use `data-*` attributes for **safe, flexible, and valid** custom metadata in your HTML!