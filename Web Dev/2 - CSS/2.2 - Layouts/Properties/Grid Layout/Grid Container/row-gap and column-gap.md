---
tags: 
 - css
 - property
 - grid
 - container
 - fundamental
---


## **🔹 What they are**

- **row-gap** → space **between rows** (vertical gap).
    
- **column-gap** → space **between columns** (horizontal gap).
    
- Together, they replace the old grid-row-gap / grid-column-gap (now deprecated).
    
- Shorthand → gap: row-gap column-gap;
    

---

## **🔹 Syntax**

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(2, 100px);

  row-gap: 20px;      /* vertical spacing */
  column-gap: 40px;   /* horizontal spacing */
}
```

Shorthand:

```css
.container {
  gap: 20px 40px; /* row-gap = 20px, column-gap = 40px */
}

.container {
  gap: 30px; /* both row-gap and column-gap = 30px */
}
```

---

## **🔹 Works in**

- **Grid** → between rows and columns.
    
- **Flexbox** → works between flex items (from Flexbox Level 1 spec update).
    
- **Multi-column layout** (columns: …;) → controls gaps between text columns.
    

---

## **🔹 Key Notes**

1. Gaps **don’t add space at the container edges**, only between items.
    (Margin would affect edges; gap doesn’t.)
    
2. gap is **better than margins** because:
    
    - No “extra margin” on the last item.
        
    - Cleaner, consistent spacing.
        
    
3. Default is 0 (no gap).
    

---

✅ **In short:**

- row-gap = vertical spacing between rows.
    
- column-gap = horizontal spacing between columns.
    
- Use gap shorthand for cleaner code.
    