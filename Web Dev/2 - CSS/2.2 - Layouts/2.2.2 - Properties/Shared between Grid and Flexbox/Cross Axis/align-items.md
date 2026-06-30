---
tags:
  - css
  - property
  - container
  - fundamental
---

The align-items property controls how **flex items are aligned on the cross axis** (perpendicular to the flex-direction).

![[Pasted image 20251221203802.png|center]]

---

### **📐 Syntax:**

```css
.container {
  display: flex;
  align-items: <value>;
}
```

```ad-note
The default value of this property is **stretch** which make all flex items stretched with the direction of the **cross axis**. This also explain why changing it stop the flex item from expanding.
```

---

### **🧭 Cross Axis Depends On:**

- flex-direction: row → cross axis is **vertical**
    
- flex-direction: column → cross axis is **horizontal**
    

---

### **🔑 Common Values:**

| **Value**  | **Description**                                            |
| ---------- | ---------------------------------------------------------- |
| stretch    | (default) Items stretch to fill container’s **cross axis** |
| flex-start | Items align to **start** of the cross axis                 | 
| flex-end   | Items align to **end** of the cross axis                   |
| center     | Items align to **center** of the cross axis                |
| baseline   | Items align based on their **text baseline**               |

```ad-note
align-items: center makes **all children of this flex container shrink to their content width and then get centered horizontally**.
```

---

### **💡 Example:**

```css
<div class="container">
  <div class="item">One</div>
  <div class="item">Two</div>
</div>
```

```css
.container {
  display: flex;
  height: 100px;
  align-items: center;
}
```

Result: All .items are vertically centered in the 100px tall container (assuming flex-direction: row).

---

### References
https://stackoverflow.com/questions/45536537/centering-in-css-grid