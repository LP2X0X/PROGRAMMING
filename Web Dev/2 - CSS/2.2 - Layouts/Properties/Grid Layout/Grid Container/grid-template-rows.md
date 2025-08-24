---
tags: 
 - css
 - property
 - grid
 - container
 - fundamental
---

## **🔹 What it does**

grid-template-rows defines **the number of rows** in a grid and **their heights**.

- Each value = one row track.
    
- Accepts the same units as columns: px, %, fr, auto, minmax(), repeat(), etc.
    

---

## **🔹 Syntax examples**

```css
.container {
  display: grid;
  grid-template-rows: 100px 200px 300px;
  /* 3 rows: fixed heights */
}

.container {
  grid-template-rows: 1fr 2fr;
  /* 2 rows, second is twice the height of first */
}

.container {
  grid-template-rows: repeat(4, 150px);
  /* 4 equal rows of 150px */
}

.container {
  grid-template-rows: 200px auto 1fr;
  /* 1st = fixed 200px
     2nd = based on content
     3rd = takes leftover space */
}

.container {
  grid-template-rows: repeat(auto-fill, minmax(100px, 1fr));
  /* auto-fills rows with at least 100px, but grows flexibly */
}
```

---

## **🔹 Units recap (same as columns)**

- **px, em, %** → fixed or relative sizes.
    
- **fr** → fraction of leftover space.
    
- **auto** → row height based on content.
    
- **minmax(min, max)** → range for responsiveness.
    
- **repeat(n, …)** → shorthand repetition.
    

---

## **🔹 Special note**

- If you don’t explicitly set grid-template-rows, rows are created **automatically** as you place items → their size defaults to auto (content height).
    
- To control auto-generated rows, use **grid-auto-rows** instead. Example:
    

```css
.container {
  grid-auto-rows: 100px; /* all new rows 100px tall */
}
```

---

✅ **In short:**

grid-template-rows is the **row blueprint** of your grid layout, just like grid-template-columns, except it controls **vertical tracks**.