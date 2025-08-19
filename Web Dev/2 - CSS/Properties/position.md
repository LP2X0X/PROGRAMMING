---
tags:
  - css
  - property
  - fundamental
---

In CSS, the position property defines how an element is positioned in the document. There are **five main values**:

---

### **🔧 position Values**

|**Value**|**Description**|
|---|---|
|static|Default. Element is in the normal document flow.|
|relative|Offsets the element relative to its **normal position**.|
|absolute|Removes the element from flow and positions it **relative to the nearest positioned ancestor** (not static).|
|fixed|Positions element relative to the **viewport**; stays fixed during scroll.|
|sticky|Scrolls normally until it reaches a threshold, then sticks in place.|

  
---

### **🧠 Example:**

```html
<div class="box">I'm positioned</div>
```

```css
.box {
  position: absolute;
  top: 50px;
  left: 100px;
}
```

This positions .box **50px from the top** and **100px from the left** of the **nearest non-static parent**.

---

### **🧲 Quick Tips:**

- Always pair absolute, relative, fixed, or sticky with top, left, right, or bottom.
    
- absolute looks for the nearest ancestor with position: relative, absolute, or fixed.
    
- sticky works only if its parent has a defined height and overflow allows scrolling.
    
