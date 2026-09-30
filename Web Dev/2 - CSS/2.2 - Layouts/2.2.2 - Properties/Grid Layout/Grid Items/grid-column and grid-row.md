---
tags: 
 - css
 - property
 - grid
 - item
 - fundamental
---

## **🔹 The Basics**

Every Grid has **lines**:

- Vertical lines = define **columns**.
    
- Horizontal lines = define **rows**.
    
    Lines are numbered starting from **1** (by default).
    

👉 grid-column and grid-row let you tell an item where to start and end.

![[debugging-css-grid-3-08a.png|center|500]]

---

## **🔹 Syntax**

```css
.item {
  grid-column: <start-line> / <end-line>;
  grid-row: <start-line> / <end-line>;
}
```

- \<start-line> → which grid line the item should begin at.
    
- \<end-line> → which grid line it should stop before.
    

---

## **🔹 Example**

```css
.grid {
  display: grid;
  grid-template-columns: repeat(4, 100px); /* 4 equal columns */
  grid-template-rows: repeat(3, 80px);     /* 3 equal rows */
  gap: 10px;
}

.item1 {
  grid-column: 1 / 3; /* spans columns 1 → 2 */
  grid-row: 1 / 2;   /* sits in first row */
}

.item2 {
  grid-column: 3 / 5; /* spans columns 3 → 4 */
  grid-row: 1 / 3;    /* spans rows 1 → 2 */
}
```

---

## **🔹 Shorthands**

- grid-column: 2 / 4; ⏩ means grid-column-start: 2; grid-column-end: 4;
    
- grid-row: 1 / span 2; ⏩ means “start at line 1, span 2 rows”
    

---

## **🔹 Special Keywords**

- span N → span across N tracks.
    

```css
grid-column: span 2; /* spans 2 columns from wherever it lands */
```

```ad-warning
**`row-span-*` only works if the grid has _explicit rows_.**  
Auto-created rows cannot be spanned in the way you expect.
```

- -1 → last line.
    

```
grid-column: 1 / -1; /* full width */
```

- auto → let the browser decide (default).
    

---

## **🔹 Visual Mental Model**

  

Think of **Excel**:

- grid-column: 1 / 3 → merge cells across column A and B.
    
- grid-row: 2 / 4 → merge cells across row 2 and 3.
    

---

✅ **Summary:**

- grid-column → controls horizontal placement (left ↔ right).
    
- grid-row → controls vertical placement (top ↔ bottom).
    
- Use span to cover multiple tracks, -1 for “last line,” auto for default placement.
    