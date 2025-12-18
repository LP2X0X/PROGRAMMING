---
tags: 
 - css
 - property
 - fundamental
---

# 1. Definition

**letter-spacing** controls the spacing between characters (tracking).  
It accepts length values (positive or negative).

```css
letter-spacing: 0.05em;
letter-spacing: 1px;
letter-spacing: -0.02em;
```

---

# 2. Accepted Units

### A. **`em` (recommended)**

- Scales with font size
    
- Most common in design systems
    
- Easy to keep typography responsive
    

### B. **`px`**

- Fixed spacing
    
- Useful for pixel-perfect logos or tight UI text
    

### C. **Unitless?**

**No** — letter-spacing **must have a unit**.

---

# 3. How values behave

### Positive values

```
letter-spacing: 0.1em;
```

Characters move farther apart.

### Negative values

```
letter-spacing: -0.02em;
```

Characters move closer together.

---

# 4. Common UI Guidelines

- **Headings:** often slightly tighter
    
    ```
    h1 { letter-spacing: -0.02em; }
    ```
    
- **All-caps text:** usually needs spacing
    
    ```
    .nav-label { letter-spacing: 0.08em; }
    ```
    
- **Body text:** rarely adjusted
    
    ```
    p { letter-spacing: normal; }
    ```
    

---

# 5. Default Value

```
letter-spacing: normal;
```

This means the browser applies its built-in kerning.

---

# 6. Tailwind Equivalent

If you’re using Tailwind concepts:

| CSS                        | Tailwind class   |
| -------------------------- | ---------------- |
| `letter-spacing: -0.02em;` | `tracking-tight` | 
| `letter-spacing: 0.05em;`  | `tracking-wide`  |
| `letter-spacing: 0.1em;`   | `tracking-wider` |

---

# 7. Example

```css
.title {
  font-size: 2rem;
  font-weight: 600;
  letter-spacing: -0.015em;
}

.small-label {
  text-transform: uppercase;
  letter-spacing: 0.1em;
  font-size: 0.75rem;
}
```