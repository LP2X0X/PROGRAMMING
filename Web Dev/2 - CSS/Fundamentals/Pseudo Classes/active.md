---
tags: 
 - css
 - pseudo
 - class
 - fundamental
---

**`:active`** = CSS pseudo-class that applies **while an element (usually a link or button) is being activated** — e.g., the moment you click (mousedown) until you release (mouseup).

---

### ✅ Tips & Best Practices:

- Often used for **“pressed” states** (like a button being clicked).
    
- Define it last in the **LVHA order** (`:link`, `:visited`, `:hover`, `:active`) so it overrides correctly.
    
- Keep styles simple (color change, slight transform) to give **immediate feedback**.
    

---

### Example:

```css
a:active {
  color: red;
  transform: scale(0.98); /* tiny press effect */
}
```

👉 **Best practice:** Combine with `:hover` for smoother UX, but don’t rely only on color for feedback (accessibility).