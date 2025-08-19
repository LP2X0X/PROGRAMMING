---
tags:
  - css
  - property
  - fundamental
---

In CSS, top, left, right, and bottom are **offset properties** used **with positioned elements** — they tell the browser **how far to move the element** from its reference edge.

---

### **📌 How They Work (Only With Certain position Values) **

They only work if the element has a position of:

- relative
    
- absolute
    
- fixed
    
- sticky (with some specific behavior)
    

```ad-note
If we set a pair of opposite direction, the component which the property is setted will be stretch.
```

---

### **📘 Examples:**

#### **1. Relative Positioning**

```css
.box {
  position: relative;
  top: 20px;
  left: 30px;
}
```

This moves .box **20px down** and **30px right** _from its original position_. The space it originally occupied remains.

#### **2. Absolute Positioning**

```css
.box {
  position: absolute;
  top: 0;
  right: 0;
}
```

This positions .box at the **top-right corner** of the nearest positioned ancestor (not static).

#### **3. Fixed Positioning**

```css
.box {
  position: fixed;
  bottom: 10px;
  left: 10px;
}
```

This pins .box to the **bottom-left of the screen**, and it stays there while scrolling.

---

### **🎯 Summary:**

|**Property**|**Meaning**|
|---|---|
|top|Distance from the top edge of the container|
|left|Distance from the left edge|
|right|Distance from the right edge|
|bottom|Distance from the bottom edge|
