---
tags: 
 - css
 - term
 - advance
---

## 📌 Containing Block (General Definition)

In CSS, every element’s position and size is computed relative to something called its **containing block**.

Think of the containing block as the **reference rectangle** that CSS uses when it calculates things like:

- `top`, `left`, `right`, `bottom` (for positioned elements)
    
- percentage-based `width`, `height`, `padding`, `margin`
    

---

### 🔹 How the Containing Block is Determined

It depends on the element’s **position** property:

1. **Static or Relative positioned elements (`position: static | relative`)**  
    → Their containing block is usually the **content box of their nearest block-level ancestor**.
    
2. **Absolute positioned elements (`position: absolute`)**  
    → Their containing block is the **padding box of the nearest ancestor** that itself has `position` set to `relative`, `absolute`, or `sticky`.  
    → If no such ancestor exists, they fall back to the **initial containing block**.
    
3. **Fixed positioned elements (`position: fixed`)**  
    → Their containing block is the **viewport** (in continuous media like screens).  
    → In paged media (like printing), it’s the page area.
    
4. **Sticky positioned elements (`position: sticky`)**  
    → Their containing block is the **nearest scrollable ancestor** (similar to relative but with scroll awareness).
    

---

## 📌 Initial Containing Block (ICB)

The **initial containing block** is a special case:

- It is the **root reference rectangle** used when no other containing block applies.
    
- In visual media (screens):
    
    - It’s basically the **viewport rectangle**.
        
    - Width = viewport width
        
    - Height = viewport height (not the whole page height).
        
- In paged media (like print):
    
    - It’s the **page area**.
        

The ICB is established by the **root element (`<html>`)**, but it conceptually matches the viewport area for screen rendering.

---

✅ **In short:**

- **Containing block** = the rectangle an element uses as its reference for positioning and percentages.
    
- **Initial containing block** = the very first, root reference rectangle (normally the viewport in browsers).
    

---

### Reference
https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_display/Containing_block#identifying_the_containing_Block