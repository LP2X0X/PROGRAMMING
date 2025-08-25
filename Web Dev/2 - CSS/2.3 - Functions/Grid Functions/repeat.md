---
tags: 
 - css
 - function
 - grid
---

## 🔹 What `repeat()` Does

The `repeat()` **function** is a shorthand that lets you avoid writing the same track size multiple times in `grid-template-columns` or `grid-template-rows`.

### 1. Basic Usage

```css
grid-template-columns: repeat(3, 100px);
```

= `100px 100px 100px`

---

### 2. Mixing Fixed + Repeated Tracks

```css
grid-template-columns: 200px repeat(2, 1fr) 100px;
```

= `200px 1fr 1fr 100px`

---

### 3. Repeating Multiple Sizes

```css
grid-template-rows: repeat(2, 100px 50px);
```

= `100px 50px 100px 50px`

---

### 4. Responsive Layouts with Auto-Fill & Auto-Fit

#### `auto-fill`

```css
grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
```

- Fills row with as many 150px–1fr tracks as fit.
    
- Extra space → leaves empty (invisible) tracks.
    

---

#### `auto-fit`

```css
grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
```

- Similar to `auto-fill`, but collapses empty tracks.
    
- Remaining tracks stretch to fill available space.
    

---

## ✅ Tips & Best Practices (for `repeat()`)

1. **Use `fr` instead of `%`** → avoids rounding issues, distributes space more evenly.
    
2. **Combine with `minmax()`** → ensures tracks don’t shrink too small (e.g., `minmax(200px, 1fr)`).
    
3. **Prefer `auto-fit` for fluid layouts** → content expands naturally, good for cards/dashboards.
    
4. **Use `auto-fill` for grids with fixed-width content** → like image galleries where spacing matters.
    
5. **Keep it readable** → don’t overuse `repeat()` inside complex grids, consider `grid-template-areas`.
    
