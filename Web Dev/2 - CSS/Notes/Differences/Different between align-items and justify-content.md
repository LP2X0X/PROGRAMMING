---
tags:
  - css
  - note
  - distinguish
---

align-items and justify-content are **CSS Flexbox (and Grid)** properties used to align child items inside a container — but they control **different directions**.

---

### **🧭 Quick Summary**

|**Property**|**Aligns in the…**|**Controls alignment of…**|
|---|---|---|
|justify-content|**Main axis** (default: row → horizontal)|Items across the main axis|
|align-items|**Cross axis** (default: column → vertical)|Items across the cross axis|

  

---

### **🔧 justify-content — Horizontal alignment (in row)**

Used to align items **left, right, center, or spaced out** along the **main axis** (the direction of the flex will determine the main axis).

```css
.container {
  display: flex;
  justify-content: center; /* center horizontally */
}
```

#### **Values:**

- flex-start: items at the start (left)
    
- flex-end: items at the end (right)
    
- center: centered
    
- space-between: even spacing, first and last at edges
    
- space-around: equal spacing around items
    
- space-evenly: equal space between and around
    

---

### **🧷** 

### **align-items — Vertical alignment (in row)**

Used to align items **top, bottom, or center** along the **cross axis**.

```css
.container {
  display: flex;
  align-items: center; /* center vertically */
}
```

#### **Values:**

- stretch (default): items stretch to fill container
    
- flex-start: align to the top
    
- flex-end: align to the bottom
    
- center: vertically centered
    
- baseline: align by text baseline
    

---

### **💡 Visual Example:**

If you imagine a row of boxes:

- justify-content: moves them **left ↔ right**
    
- align-items: moves them **top ↕ bottom**
    