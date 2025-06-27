---
tags: css, property, fundamental
---

The `box-sizing` property controls **how the size of an element is calculated**: whether padding and border are included **inside** the width/height or **outside** it.

---

## 🔧 Syntax

```css
box-sizing: content-box | border-box;
```

---

## 🧱 Values

### 1. `content-box` (default)

- **Width/height = content only**
    
- Padding and border are added **outside** the set dimensions
    
- 🚫 Can lead to unexpected layout expansion
    

```css
div {
  width: 200px; /* only content */
  padding: 20px;
  border: 5px solid black;
}
/* Actual rendered size: 200 + 40 + 10 = 250px wide */
```

---

### 2. `border-box`

- **Width/height includes content + padding + border**
    
- Prevents layout overflow
    
- 🧼 Easier to reason about dimensions
    

```css
div {
  box-sizing: border-box;
  width: 200px; /* includes padding and border */
  padding: 20px;
  border: 5px solid black;
}
/* Actual rendered size = exactly 200px wide */
```

---

## 🌐 Best Practice

Add this to your CSS to make layout more predictable:

```css
*, *::before, *::after {
  box-sizing: border-box;
}
```

- Ensures **all elements** use `border-box`
    
- Helps with responsive design, flex/grid layouts, and avoiding box overflows
    

---

## ✅ Summary

|`box-sizing`|Includes Padding & Border in Width/Height?|Default?|
|---|---|---|
|`content-box`|❌ No|✅ Yes|
|`border-box`|✅ Yes|❌ No|
