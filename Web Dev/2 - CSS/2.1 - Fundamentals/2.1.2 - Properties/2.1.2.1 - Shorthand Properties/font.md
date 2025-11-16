---
tags: 
 - css
 - font
 - shorthand
 - property
---

`font` is a shorthand that sets **multiple font-related properties in one declaration**.

## ✔ **Required parts**

You **must** include at least these:

- `font-size`
    
- `font-family`
    

If either is missing → the whole declaration is invalid.

---

## ✔ **Full shorthand order**

```
font: [font-style] [font-variant] [font-weight] 
      [font-stretch] <font-size> [/line-height] <font-family>;
```

**Order matters** → the required ones go last.

---

## ✏️ Example (full)

```css
font: italic small-caps 600 condensed 16px/1.4 "Inter", sans-serif;
```

---

## ✏️ Example (common minimal)

```css
font: 16px/1.5 Arial, sans-serif;
```

---

## ✔ Properties affected by `font`

Setting `font:` replaces ALL of these:

- `font-style`
    
- `font-variant`
    
- `font-weight`
    
- `font-stretch`
    
- `font-size`
    
- `line-height`
    
- `font-family`
    

Anything omitted → gets its **initial value**.

---

## ⚠ Important Gotchas

### **1. Missing size or family = whole rule ignored**

This is the most common bug.

### **2. `line-height` must follow `font-size` using `/`**

Correct:

```css
font: 14px/20px Roboto;
```

Incorrect:

```css
font: 14px Roboto / 20px; /* ❌ invalid */
```

### **3. Using `font` overwrites previous indiv. font properties**

This breaks things if you mix them unintentionally:

```css
font-weight: bold;
font: 16px Arial; /* bold is lost */
```

### **4. Shorthand doesn’t include:**

- `font-kerning`
    
- `letter-spacing`
    
- `word-spacing`
    
- `text-rendering`
    
- `text-decoration`
    

These must be set separately.

---

## ⭐ Best practice

- Prefer `font` shorthand for **component-level resets**.
    
- Use longhand when you only change one property to avoid overwriting.
    
- Always include a _fallback family_:  
    `"Inter", system-ui, sans-serif`
    
- A good use for this is to set general styles on the \<body> element.