---
tags: 
 - css
 - term 
 - fundamental
---

# 📘 Normal Flow in CSS

👉 **Normal flow** is the default way the browser lays out elements **before you apply positioning, floats, or special layout modes**.

It defines how **block-level** and **inline-level** elements are arranged inside their containers.

---

## 1. 🔹 How Elements Flow by Default

### **Block-level elements** (e.g. `<div>`, `<p>`, `<h1>`, `<section>`)

- Stack **vertically**, top to bottom.
    
- Take up the full available width (by default).
    
- Height expands to fit content.
    

```html
<div>One</div>
<div>Two</div>
<div>Three</div>
```

👉 They appear one below the other.

---

### **Inline-level elements** (e.g. `<span>`, `<a>`, `<strong>`)

- Flow **horizontally**, left to right (or according to writing mode).
    
- Wrap to a new line if they run out of space.
    
- Their box height is defined by `line-height` and content.
    

```html
<p>Hello <span>world</span>!</p>
```

👉 Appears as inline text inside a line box.

---

### **Mixing block & inline**

- Inline boxes inside block boxes form **inline formatting context**.
    
- Block boxes inside block boxes form **block formatting context**.
    

---

## 2. 🔹 When You Break Normal Flow

Certain properties **remove elements from normal flow**:

1. **Floats** (`float: left/right`)
    
    - Element is pulled to a side, text and inline content wrap around it.
        
    - Taken out of normal block stacking.
        
2. **Positioning**
    
    - `position: absolute` / `fixed` → element taken out of normal flow entirely.
        
    - `position: relative` → stays in flow, but visually offset.
        
    - `sticky` → hybrid: behaves like relative until threshold reached.
        
3. **Modern layouts**
    
    - `display: flex`, `grid`, `table` → establish **new formatting contexts** → children don’t use normal block/inline flow rules anymore.
        

---

## 3. 🔹 Why Normal Flow Matters

- **Margins collapse** only in normal block flow.
    
- Understanding flow prevents layout bugs (e.g., why elements overlap when absolutely positioned).
    
- **Accessibility**: screen readers follow document order, not visual order (important when breaking normal flow).
    

---

## 4. 🔹 Summary

- **Normal flow = default layout behavior**
    
    - Block-level → vertical stack.
        
    - Inline-level → horizontal text flow.
        
- **Taken out of normal flow**:
    
    - `float`, `position: absolute/fixed`, `flex`, `grid`, etc.
        
- **Still in normal flow but modified**:
    
    - `position: relative`, `sticky`.
        

---

✅ So whenever CSS docs say _“this element is removed from normal flow”_, it means:

> “The browser no longer positions this element according to the usual block/inline stacking rules — it’s being laid out in a special context (float, abs-pos, flex/grid, etc.).”