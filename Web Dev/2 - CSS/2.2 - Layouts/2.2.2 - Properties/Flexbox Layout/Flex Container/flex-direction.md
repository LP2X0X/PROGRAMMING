---
tags: 
  - css
  - property
  - flex
  - container
  - fundamental
---

`flex-direction` controls **the main axis** of a flex container — i.e., **the direction in which flex items are laid out**.

```css
.flex {
  display: flex;
  flex-direction: row; /* default */
}
```

---

# ✅ **The 4 possible values**

## **1. `row` (default)**

- Main axis = **horizontal, left → right**
    
- Cross axis = vertical
    
- Items line up side-by-side.
    

## **2. `row-reverse`**

- Main axis = horizontal, **right → left**
    
- Cross axis = vertical
    
- Items visually reversed, but **DOM order stays the same** (important!).
    

## **3. `column`**

- Main axis = **vertical, top → bottom**
    
- Cross axis = horizontal
    
- Items stack vertically.
    

## **4. `column-reverse`**

- Main axis = vertical, **bottom → top**
    
- Cross axis = horizontal
    
- Again, DOM order stays the same, only visual direction reverses.
    

---

# 🧠 **Key Concepts**

### **1. It switches the main axis**

- Horizontal axis = `row` / `row-reverse`
    
- Vertical axis = `column` / `column-reverse`
    

### **2. All other flex properties change meaning depending on this**

For example:

|Property|When `row`|When `column`|
|---|---|---|
|`justify-content`|horizontal alignment|vertical alignment|
|`align-items`|vertical alignment|horizontal alignment|

So `justify-content` **always** aligns along the main axis, whatever direction it is.

---

# 🎯 **Reverse values do NOT change source order**

Example:

```html
<div style="display: flex; flex-direction: row-reverse">
  <div>A</div>
  <div>B</div>
</div>
```

You still tab A → B.  
Screen readers still read A → B.  
Only the **visual** order changes.

---

# 📦 **In React / Tailwind**

### Tailwind shortcuts:

|CSS|Tailwind|
|---|---|
|`flex-direction: row;`|`flex-row`|
|`flex-direction: row-reverse;`|`flex-row-reverse`|
|`flex-direction: column;`|`flex-col`|
|`flex-direction: column-reverse;`|`flex-col-reverse`|

---

# 🛠️ When to use which?

- `flex-row` → horizontal navbars, tags, horizontal lists.
    
- `flex-col` → vertical stacks, forms, card layouts.
    
- Reverse versions → chat UIs, reverse chronological lists.
    