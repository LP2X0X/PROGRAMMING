---
tags: 
 - html
 - attribute
 - type
 - global
---

### tabindex

The `tabindex` attribute controls the **keyboard navigation order** (tabbing order) of HTML elements.

---

### ✅ Values:

| Value                    | Behavior                                                               |
| ------------------------ | ---------------------------------------------------------------------- |
| `0`                      | Includes the element in the natural tab order (where it normally fits) |
| Positive `n` (e.g., `1`) | Custom tab order; lower numbers are focused first. *(Not recommended)* |
| `-1`                     | Makes the element **focusable**, but **not** reachable via Tab key     |

---

### 🧾 Example:

```html
<input type="text" placeholder="First" tabindex="2" />
<input type="text" placeholder="Second" tabindex="1" />
<button tabindex="0">Submit</button>
```

* Tabbing order: Second → First → Submit

---

### 💡 Common Use Cases:

* Making non-focusable elements (like `<div>`) keyboard accessible:

  ```html
  <div tabindex="0">Focusable div</div>
  ```
* Skipping elements from the tab order with `-1`

---

### ⚠️ Best Practices:

* Use `tabindex="0"` to include custom elements in the tab order.
* Avoid positive values unless you **have a strong reason** — they can confuse users.

---

Use `tabindex` to improve **keyboard navigation** and **accessibility**.
