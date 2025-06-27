---
tags: css, note, distinguish
---

The position: fixed and position: absolute CSS properties are both used to remove an element from the normal document flow and position it relative to something — but **what they’re relative to** is the key difference:

---

### **✅ position: absolute**

- **Relative to:** The nearest positioned ancestor (relative, absolute, or fixed). If none exists, it’s positioned relative to the **initial containing block** (usually <html> or <body>).
    
- **Scrolls with the page:** Yes
    
- **Use case:** Popups, dropdowns, or elements that should move as the page scrolls.
    

```css
.parent {
  position: relative;
}

.child {
  position: absolute;
  top: 10px;
  left: 10px;
}
```

---

### **✅ position: fixed**

- **Relative to:** The **viewport** (i.e., the browser window)
    
- **Scrolls with the page:** No — it stays fixed in place
    
- **Use case:** Sticky headers, floating action buttons, sidebars, modals
    

```css
.fixed-box {
  position: fixed;
  top: 10px;
  right: 10px;
}
```

---

### **🧠 Summary Table**

|**Feature**|absolute|fixed|
|---|---|---|
|Positioned relative to|Nearest positioned ancestor|Viewport (browser window)|
|Scrolls with page|✅ Yes|❌ No|
|Common uses|Tooltips, dropdowns|Sticky navs, modals|

If you want something to stay in place as the user scrolls, use fixed.

If you want something to move with a scroll but stay anchored to a parent element, use absolute.
