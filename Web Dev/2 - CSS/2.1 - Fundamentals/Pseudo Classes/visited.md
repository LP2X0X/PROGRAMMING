---
tags: 
 - css 
 - pseudo
 - class
 - fundamental
---

**`:visited`** = CSS pseudo-class that targets **links the user has already clicked/visited**.

### ✅ Tips & Best Practices:

- **Style difference:** Commonly purple (vs blue for unvisited) so users know where they’ve been.
    
- **Privacy restriction:** Browsers limit what you can style with `:visited` — you can only change **color-related properties** (e.g., `color`, `background-color`), **not layout or content** (like `display`, `width`, or `content`).
    
- **Pair with `:link`:** Define both `:link` and `:visited` together for clarity.
    

### Example:

```css
a:link {
  color: blue;
}
a:visited {
  color: purple;
}
```

👉 **Best practice:** Keep visited links distinguishable but subtle — helps navigation without overwhelming the UI.