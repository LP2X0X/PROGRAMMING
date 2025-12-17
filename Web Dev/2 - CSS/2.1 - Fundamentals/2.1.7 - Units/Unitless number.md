---
tags: 
 - css
 - unit
---

## 1. Definition

A **unitless number** in CSS is a numeric value written **without a length unit** (px, rem, em, %, etc.).  
Example:

```
line-height: 1.5;
flex: 1;
opacity: 0.8;
```

Unitless values are allowed only in specific CSS properties where the specification defines them as valid.

```ad-note
When you use a unitless number, that declared value is inherited, meaning its computed value is recalculated for each inheriting child element.
```

---

## 2. When unitless values are allowed

### A. **Line-height**

Unitless `line-height` is a _multiplier_ of the font size, not a length.

```
line-height: 1.5;   /* Good – scalable, inherits better */
line-height: 24px;  /* Fixed line-height */
```

Best practice: use unitless for responsive typography.

---

### B. **Font-weight**

```
font-weight: 400;
font-weight: 700;
```

These are not lengths; they are predefined numeric keywords.

---

### C. **Opacity**

```
opacity: 0.4;  /* Range: 0–1 */
```

---

### D. **Z-index**

```
z-index: 10;
```

---

### E. **Flex and Grid numeric values**

Certain flex/grid properties accept unitless numbers as pure _fractions_ or _ratios_.

```
flex-grow: 1;
flex-shrink: 0;
flex: 1;          /* flex-grow: 1; flex-shrink: 1; flex-basis: 0 */
```

```
grid-row: 2 / 4;
order: 3;
```

---

### F. **Animation-related numeric values**

```
animation-iteration-count: 3;
```

---

### G. **Scale transforms**

```
transform: scale(1.2);
```

---

## 3. When unitless is **not** allowed

Anything representing a **physical measurement** requires units:

```
width: 20;        /* Invalid */
margin: 10;       /* Invalid */
border-width: 2;  /* Invalid */
```

Correct usage:

```
width: 20px;
margin: 10px;
border-width: 2px;
```

---

## 4. The “magic” of unitless values in inheritance

Especially for **line-height**, unitless values solve a common pitfall:

### Example

```
.parent {
  font-size: 20px;
  line-height: 1.4; /* unitless - multiplier */
}

.child {
  font-size: 14px;
}
```

The child’s actual computed line-height becomes:  
`14px * 1.4 = 19.6px`  
This maintains vertical rhythm across varying font sizes.

If you used pixels (`line-height: 28px;`), children would inherit the fixed 28px, breaking typography.

---

## 5. Why CSS sometimes _requires_ unitless values

Some properties define values as **pure numbers**, not measurements.  
Examples:

- `line-height: <number>`
    
- `flex-grow: <number>`
    
- `order: <integer>`
    
- `opacity: <number>`
    
- `z-index: <integer>`
    

This keeps behavior consistent across context, allows ratio-based relationships, and avoids unnecessary units.

---

## 6. Summary Table

| Property Type                            | Unitless Allowed | Meaning                          |
| ---------------------------------------- | ---------------- | -------------------------------- |
| Typography: `line-height`                | Yes              | Multiplier of font size          |
| Typography: `font-weight`                | Yes              | Weight keyword values            |
| Layout: `flex-grow`, `flex-shrink`       | Yes              | Ratios                           |
| Layout: `flex` shorthand                 | Partially        | First two values may be unitless |
| Opacity                                  | Yes              | 0–1                              |
| Z-index                                  | Yes              | Stacking integer                 |
| Transform: scale                         | Yes              | Factor                           |
| Lengths (width, height, margin, padding) | No               | Must use units                   |