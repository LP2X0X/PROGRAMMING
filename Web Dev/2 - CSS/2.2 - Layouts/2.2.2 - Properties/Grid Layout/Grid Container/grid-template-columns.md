---
tags: 
 - css
 - property
 - grid
 - container
 - fundamental
---


## **🔹 What it does**

grid-template-columns defines **the number of columns** in a grid and **their widths**.

- Each value you provide = one column track.
    
- The property accepts **lengths, percentages, fr units, auto, minmax(), repeat()**, etc.
    

---

## **🔹 Syntax examples**

```css
.container {
  display: grid;
  grid-template-columns: 200px 200px 200px;  /* 3 fixed columns */
}

.container {
  grid-template-columns: 1fr 2fr;  /* 2 columns, second takes double space */
}

.container {
  grid-template-columns: repeat(3, 150px); /* 3 equal 150px columns */
}

.container {
  grid-template-columns: 200px auto 1fr;  
  /* 1st = fixed 200px, 2nd = size based on content, 3rd = flexible */
}

.container {
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  /* responsive: fills as many 150px min columns as possible, up to 1fr */
}
```

---

## **🔹 Units you can use**

- **px, em, %, etc.** → absolute or relative lengths.
    
- **fr** → “fraction of leftover space”.
    
- **auto** → size based on content.
    
- **minmax(min, max)** → responsive range.
    
- **repeat(count, track-size)** → shorthand repetition.
    
- **auto-fill / auto-fit** → dynamic column creation until the row is full.
    

---

## **🔹 Visualization example**

```css
grid-template-columns: 100px 1fr 2fr;
```

- Column 1 → fixed 100px.
    
- Column 2 → 1 fraction of leftover.
    
- Column 3 → 2 fractions of leftover (twice as wide as col 2).
    

---

✅ **In short:**

grid-template-columns is the blueprint for your grid’s **column structure**, letting you mix fixed, content-based, and flexible tracks to build anything from simple layouts to fully responsive grids.