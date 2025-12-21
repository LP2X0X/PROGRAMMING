---
tags:
  - css
  - property
  - flex
  - container
  - fundamental
---

### **🔧 justify-content in CSS Flexbox**


The justify-content property controls **how flex items are aligned along the **main axis** (horizontal if flex-direction: row, vertical if flex-direction: column).

![[Pasted image 20251221203707.png|center]]

---

### **📐 Syntax:**

```css
.container {
  display: flex;
  justify-content: <value>;
}
```

---

### **🎯 Common Values:**

| **Value**     | **Description**                                     |
| ------------- | --------------------------------------------------- |
| flex-start    | Items align to the start of the main axis (default) |
| flex-end      | Items align to the end of the main axis             |
| center        | Items are centered along the main axis              |
| space-between | Equal space **between** items, none at ends         |
| space-around  | Equal space **around** items (half at ends)         |
| space-evenly  | Equal space **between all** items including ends    |

---

### **💡 Example:**

```html
<div class="container">
  <div class="item">A</div>
  <div class="item">B</div>
  <div class="item">C</div>
</div>
```

```css
.container {
  display: flex;
  justify-content: space-between;
}
```

Result: A    B    C (even spacing between items)
