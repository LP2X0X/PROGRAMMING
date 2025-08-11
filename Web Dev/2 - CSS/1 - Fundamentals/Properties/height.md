---
tags: css, property, fundamental
---

### 🔹 **Definition**  
The `height` property in CSS specifies the height of an element’s **content box**.  
It does **not** include padding, border, or margin unless `box-sizing: border-box` is applied.

---

### 🔹 **Syntax**

```css
selector {
  height: value;
}
```

- **value** can be:
    
    - `auto` → Default. Height is determined by content.
        
    - `<length>` → Fixed height (e.g., `200px`, `20em`).
        
    - `<percentage>` → Percentage of containing block’s height.
        
    - `max-content`, `min-content`, `fit-content` (more advanced cases).
        

---

### 🔹 **Important Notes**

- If the parent element has no set height, a percentage height may not work as expected.
    
- `min-height` and `max-height` can override `height` in certain conditions.
    
- Setting `height` does **not** affect `line-height` (text spacing).
    
- If content exceeds the fixed height, it may **overflow**, unless handled with `overflow` property.
    

---

### 🔹 **Examples**

```css
.box {
  height: 300px; /* fixed height */
}

.wrapper {
  height: 100%; /* takes full height of parent */
}

.dynamic {
  height: auto; /* grows/shrinks with content */
}
```

---

### 💡 **Tips**

```ad-important
- Use `min-height` instead of a fixed `height` for content that might grow, to avoid overflow.
```
    
- When using percentage heights, ensure the parent has a defined height.
    
- Use `box-sizing: border-box` so that padding and border are included in the height calculation.
    

---

### ✅ **Best Practices**

- Avoid fixed pixel heights for elements containing dynamic text or images.
    
- Combine `height` with `flexbox` or `grid` layout for more responsive designs.
    
- Use `auto` or `min-height` for better adaptability across devices.
    
- For full viewport height layouts, use `height: 100vh` instead of relying on percentages.
    