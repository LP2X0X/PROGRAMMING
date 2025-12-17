---
tags: 
 - css
 - property
 - fundamental
---

# 1. What `color` controls

`color` sets the **foreground text color** of an element.

```
color: red;
color: #333;
color: var(--text-primary);
```

This affects:

- text
    
- text decorations (underline, overline)
    
- SVG stroke/fill when using `currentColor`
    
- form elements (partially, depending on browser)
    

---

# 2. `color` **is inheritable**

This is crucial.

Children automatically inherit unless:

- The child sets its own `color`
    
- The child is a replaced element (e.g., `<input>`, `<img>`)
    
- A CSS reset overrides color
    
- The child lives inside Shadow DOM
    

---

# 3. Acceptable color formats

CSS allows multiple syntaxes:

### A. Named colors

```
color: red;
color: slategray;
```

### B. Hex

```
color: #1e293b;
color: #1E293B80; /* with alpha */
```

### C. RGB/RGBA

```
color: rgb(30 41 59);
color: rgb(30 41 59 / 0.5);
```

### D. HSL

```
color: hsl(210 40% 96%);
```

### E. OKLCH / OKLab (modern, recommended)

```
color: oklch(65% 0.19 258);
```

### F. `currentColor`

Special keyword for SVG and border interactions:

```
border-color: currentColor;
```

It forces border (or SVG stroke/fill) to match the text color.

---

# 4. Inheritance rules

Because `color` inherits:

```
body {
  color: #334155;
}

p {
  /* inherits #334155 automatically */
}
```

If a child overrides it:

```
.child {
  color: black;
}
```

then inheritance stops.

---

# 5. How to force inheritance

Sometimes you explicitly want a nested element to follow the parent:

```
.child {
  color: inherit;
}
```

Useful when resets or modules override color by accident.

---

# 6. `color` vs. `background-color`

Common confusion:

- `color` = foreground text
    
- `background-color` = painted area behind the element
    

They are separate properties.

---

# 7. Special case: replaced elements

Some elements don’t inherit color normally:

- `<input>`
    
- `<textarea>`
    
- `<select>`
    
- `<button>`
    
- `<img>`
    
- `<canvas>`
    
- `<video>`
    

To enforce it:

```css
input, textarea, select, button {
  color: inherit;
  font: inherit;
}
```

---

# 8. Interaction with CSS Modules

In CSS Modules, this:

```css
body {
  color: #334155;
}
```

will **not** apply globally, because `body` becomes a scoped selector.

To apply globally:

```css
:global(body) {
  color: #334155;
}
```

Or put global styles in a true global stylesheet (recommended).

---

# 9. Interaction with Tailwind (if you use it)

Tailwind color classes override inherited color:

```
text-slate-700
text-black
text-white
```

These are _direct overrides_, so inherited body color won't apply.

---

# 10. Examples

## A. Typography system

```
:root {
  --color-text: oklch(35% 0 0);
}

body {
  color: var(--color-text);
  font-family: system-ui, sans-serif;
}
```

## B. Force form controls to inherit

```
input, button, textarea {
  color: inherit;
  font: inherit;
}
```

## C. Using currentColor

```
.button {
  color: var(--color-primary);
  border: 2px solid currentColor; /* border matches text */
}
```

---

# Summary Table

| Property           | Inherited?                      | Typical Use           |
| ------------------ | ------------------------------- | --------------------- |
| `color`            | Yes                             | Text foreground color | 
| `background-color` | No                              | Background            |
| `border-color`     | No (but can use `currentColor`) | Border color          |
