---
tags: 
 - css
 - term
 - advance
---

A **formatting context** is an isolated “mini layout environment” that controls how its child boxes are laid out, independent of elements outside it.

👉 Think of it as **a set of rules for arranging boxes inside a container**.

---

## 1. 🔹 Types of Formatting Contexts

### **1.1 Block Formatting Context (BFC)**

- The most common type.
    
- **Block-level boxes** participate in BFC.
    
- Determines vertical layout (block stacking).
    
- **Margins collapse** between boxes inside a BFC.
    

**Created by** (any of these):

```css
float: left/right;
overflow: hidden/auto/scroll/clip;
display: flow-root;
position: absolute/fixed;
display: inline-block;
display: table-cell;
display: table-caption;
```

**Key behaviors:**

- Contain floats (floats don’t escape BFC).
    
- Prevent margin collapse with outside elements.
    
- Establish a new layout context for blocks.
    

---

### **1.2 Inline Formatting Context (IFC)**

- Created when inline-level boxes are inside a block box.
    
- Lays out **text and inline elements** horizontally.
    
- Respects line boxes, `line-height`, `white-space`, etc.
    

**Key behaviors:**

- Boxes line up side by side until wrapping to a new line.
    
- Vertical alignment uses `line-height` and `vertical-align`.
    

---

### **1.3 Flex Formatting Context (FFC)**

- Created by `display: flex` or `inline-flex`.
    
- Children are flex items, not normal block/inline boxes.
    

**Key behaviors:**

- No margin collapsing between flex items.
    
- `margin: auto` distributes free space.
    
- Items can shrink/grow/stretch.
    

---

### **1.4 Grid Formatting Context (GFC)**

- Created by `display: grid` or `inline-grid`.
    
- Children are grid items.
    

**Key behaviors:**

- No margin collapsing between grid items.
    
- Items are placed in grid tracks (rows/columns).
    
- `gap` controls spacing, not margin.
    

---

### **1.5 Table Formatting Context**

- Created by table-related display types (`table`, `table-row`, `table-cell`).
    
- Complex: follows CSS table model.
    

---

### **1.6 Other Contexts**

- **Absolute Positioning Context** → created by `position: absolute/fixed`.
    
- **Multicolumn Context** → `column-count` / `column-width`.
    
- **Containment Contexts** (new CSS Containment spec) → `contain: layout`.
    

---

## 2. 🔹 Why Formatting Contexts Matter

- **Margin Collapsing**
    
    - Happens inside BFC, but **not across FFC or GFC**.
        
- **Floats**
    
    - Escape normal flow unless inside a new BFC.
        
- **Positioning**
    
    - Each context is like a “separate world”.
        
- **Centering**
    
    - Flex/Grid contexts allow powerful alignment, unlike normal BFC.
        

---

## 3. 🔹 Examples

### Prevent Margin Collapse

```css
.parent {
  overflow: auto; /* creates BFC */
}
```

Now child margin won’t collapse into parent.

---

### Clear Floats with BFC

```css
.container {
  overflow: hidden; /* container expands to wrap floats */
}
```

---

### Inline Formatting

```html
<p>Hello <span style="color:red">world</span>!</p>
```

All inside an **inline formatting context**, arranged in line boxes.

---

### Flex Formatting

```css
.container {
  display: flex;
}
.child {
  margin: auto; /* perfectly centers inside */
}
```

---

### Grid Formatting

```css
.container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}
```

Grid items don’t collapse margins — spacing is explicit.

---

## 4. 🔹 Debugging / Tips

✔️ **Know which context you’re in** → determines how margins, floats, and positioning behave.  
✔️ Use `flow-root` to make a **self-contained block** (new BFC).  
✔️ Use **Flex/Grid for alignment** instead of hacks with margins.  
✔️ Remember: **IFC → inline/text rules**, **BFC → blocks/floats**, **FFC/GFC → modern layouts**.

---

## 5. 🔹 Quick Reference Table

| Context        | Trigger                                                                       | Key Features                       |
| -------------- | ----------------------------------------------------------------------------- | ---------------------------------- |
| BFC            | float, overflow≠visible, inline-block, absolute, fixed, table-cell, flow-root | Margins collapse, contains floats  |
| IFC            | inline content in block box                                                   | Line boxes, vertical-align         |
| FFC            | display: flex/inline-flex                                                     | No margin collapse, flex alignment |
| GFC            | display: grid/inline-grid                                                     | No margin collapse, grid tracks    |
| Table FC       | display: table*                                                               | Table layout rules                 |
| AbsPos FC      | position: absolute/fixed                                                      | Out of normal flow                 |
| Multicolumn FC | columns set                                                                   | Flow split into columns            |