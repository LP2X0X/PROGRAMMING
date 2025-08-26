---
tags: 
 - css
 - keyword
 - grid
---

## 🔹 `span` in CSS Grid

The **`span` keyword** is used inside **`grid-column`** or **`grid-row`** to make an element cover **multiple tracks** (columns or rows).

---

### 1. Syntax

```css
grid-column: <start> / span <number>;
grid-row: <start> / span <number>;
```

- `<start>` = where the item begins (grid line number, name, or `auto`).
    
- `span <number>` = how many tracks to extend.
    

---

### 2. Examples

#### ✅ Spanning Columns

```css
.item {
  grid-column: 1 / span 2;
}
```

- Start at column line **1**
    
- Span **2 tracks** (so it covers columns 1 and 2).
    

---

#### ✅ Spanning Rows

```css
.item {
  grid-row: 2 / span 3;
}
```

- Start at row line **2**
    
- Span **3 tracks** (so it covers rows 2, 3, and 4).
    

---

#### ✅ Auto Start with Span

```css
.item {
  grid-column: auto / span 3;
}
```

- Browser decides the starting position (auto-placement).
    
- Item will cover **3 columns** from wherever it’s placed.
    

---

### 3. Shorthand

Instead of writing:

```css
grid-column-start: 1;
grid-column-end: span 2;
```

You can write:

```css
grid-column: 1 / span 2;
```

---

## ✅ Tips & Best Practices (for `span`)

1. **Use `span` for flexibility** → better than hardcoding end lines, especially when grid changes.
    
    - Example: `grid-column: 1 / 4;` breaks if columns are rearranged.
        
    - Better: `grid-column: 1 / span 3;`.
        
2. **Good for auto-placement** → if you don’t care where an item starts, `auto / span n` is clean and responsive.
    
3. **Combine with named lines** → more readable grids.
    
    ```css
    grid-template-columns: [sidebar] 200px [content] 1fr [end];
    .sidebar { grid-column: sidebar / span 1; }
    ```
    
4. **Watch overlap** → if multiple items span into the same space, they’ll overlap unless you control placement.
    
5. **Prefer `span` in dynamic layouts** → makes your grid more resilient to future changes (adding/removing tracks).
    