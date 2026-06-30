---
tags: 
  - css
  - property
  - grid
  - item
  - fundamental
---

## 🎯 What `justify-self` Does

`justify-self` controls **how a single item is aligned along the inline (horizontal) axis** _within its own layout cell_.

It is the **per-item override** of `justify-items`.

---

## 🧩 Where `justify-self` Works

### ✅ CSS Grid

- Fully supported
    
- Applies to **individual grid items**
    
- Aligns content **inside the grid cell**
    

```css
.item {
  justify-self: center;
}
```

---

### ❌ Flexbox

- **Does NOT work on flex items**
    
- Ignored completely
    

Reason:

> Flexbox distributes space _between items_, not _inside item slots_

---

## 🧭 Axis Explained

| Layout | Axis controlled by `justify-self` |
| ------ | --------------------------------- |
| Grid   | Inline axis (left ↔ right in LTR) | 

For the block axis (top ↔ bottom), use:

```css
align-self
```

---

## 🔁 Relationship to Other Properties

| Property        | Scope                  |
| --------------- | ---------------------- |
| `justify-items` | All grid items         |
| `justify-self`  | One grid item          |
| `align-items`   | All items (block axis) |
| `align-self`    | One item (block axis)  |

---

## 🧪 Common Values

```css
justify-self: start | end | center | stretch;
```

Notes:

- `stretch` is default (fills cell width)
    
- `flex-start` / `flex-end` ❌ invalid here
    

---

## 🧠 Why `justify-self` Does NOT Exist in Flexbox

Flexbox uses:

- `justify-content` → aligns **items as a group**
    
- `align-items` → aligns **items individually on cross axis**
    

There is **no concept of an individual horizontal slot**, so `justify-self` would be ambiguous.

---

## ✅ Example: Grid Alignment

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}

.item {
  justify-self: end;
}
```

The item moves to the **right edge of its grid cell**, not the container.

---

## 🚫 Common Mistakes

❌ Expecting `justify-self` to work in Flexbox  
❌ Using `flex-start` instead of `start`  
❌ Confusing it with `justify-content`

---

## 🧠 Mental Model

> **Grid aligns items inside fixed cells → `justify-self` makes sense.**  
> **Flexbox aligns items relative to each other → `justify-self` does not exist.**

---

## 📌 One-line Summary

`justify-self` aligns **a single grid item horizontally within its own grid cell** and works **only in CSS Grid**, not Flexbox.