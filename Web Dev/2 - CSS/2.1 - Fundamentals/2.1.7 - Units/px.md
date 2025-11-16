---
tags: 
 - css
 - unit
 - absolute
---

### **`px` (Pixels) in CSS**

**Definition:**

- `px` stands for **pixels**, a unit of length used to define sizes in CSS, such as widths, heights, margins, padding, font sizes, borders, etc.
    
- It represents an **absolute unit** based on the screen’s resolution. Traditionally, 1px equals 1/96 of an inch, but on modern screens with varying pixel densities, it’s more about **CSS pixels**, not actual device pixels.
    

---

### **Characteristics**

1. **Absolute Unit:**
    
    - Unlike relative units (`em`, `rem`, `%`), `px` does not scale based on parent elements or font size.
        
    - It gives **precise control** over layout dimensions.
        
2. **Common Usage:**
    
    - `font-size: 16px;` → Sets text size to 16 pixels.
        
    - `margin: 10px;` → Sets margin of 10 pixels.
        
    - `border-width: 2px;` → Sets border thickness.
        
3. **Pros:**
    
    - Predictable and consistent across elements.
        
    - Easy to use when exact pixel precision is needed.
        
4. **Cons:**
    
    - Not responsive: won’t scale automatically with user settings or screen size.
        
    - Accessibility: users who zoom the page might not see text scale if only `px` is used.
        

---

### **Best Practices**

- Use `px` for **borders, precise spacing, or images** where exact size matters.
    
- For **font sizes and layouts**, consider using **relative units** (`em`, `rem`, `%`) to improve accessibility and responsiveness.
    
