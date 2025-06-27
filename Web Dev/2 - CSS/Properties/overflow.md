---
tags: css, property, fundamental
---

### **🧾 CSS overflow Property — Overview**

The overflow property in CSS controls **what happens when content overflows** the dimensions (usually width or height) of its container.

---

### **📌 Syntax**

```css
.element {
  overflow: visible | hidden | scroll | auto | clip;
}
```
  

---

### **🎯 Values & What They Do**

|**Value**|**Description**|
|---|---|
|visible (default)|Overflowing content is **not clipped** and spills out|
|hidden|Overflowing content is **clipped**, and not accessible|
|scroll|Always shows **scrollbars** (even if not needed)|
|auto|Scrollbars appear **only when necessary**|
|clip|Like hidden, but **no scroll mechanism** (for precise control)|
  

---

### **🔁 Axis Control**

  

You can target **specific axes**:

- overflow-x: horizontal overflow
    
- overflow-y: vertical overflow
    

```css
.element {
  overflow-x: auto;
  overflow-y: hidden;
}
```

  
---

### **💡 Example**

```html
<div class="box">Very long content that overflows...</div>
```

```css
.box {
  width: 200px;
  height: 100px;
  overflow: auto;
}
```


---

### **⚠️ Tips**

- Use overflow: hidden to **contain float elements** or fix **margin collapsing** (like your previous issue).
    
- overflow: auto is best for scrollable sections.
    
- Combine with white-space: nowrap for horizontal scroll.
    
