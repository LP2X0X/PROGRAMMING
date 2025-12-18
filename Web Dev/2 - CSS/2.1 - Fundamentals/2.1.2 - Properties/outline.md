---
tags: 
 - css
 - property
 - fundamental
---

# 1. What `outline` is

`outline` draws a **non-box-model, non-offset border** _around_ an element.

Important differences from `border`:

- It does **not occupy space**
    
- It does **not affect layout**
    
- It can overlap other content
    
- It is primarily used for **focus indicators** (accessibility)
    

Example:

```css
outline: 2px solid #2563eb;
```

---

# 2. Longhand properties

`outline` is a shorthand for:

### `outline-width`

```
outline-width: 1px;
outline-width: medium;
```

### `outline-style`

```
outline-style: solid;
outline-style: dotted;
outline-style: dashed;
outline-style: none;
```

### `outline-color`

```
outline-color: red;
outline-color: currentColor;
outline-color: var(--primary);
```

### `outline-offset`

Pushes the outline away from the element:

```
outline-offset: 4px;
```

---

# 3. Shorthand syntax

```
outline: <outline-width> <outline-style> <outline-color>;
```

Example:

```css
outline: 3px dashed oklch(60% 0.13 260);
outline-offset: 3px;
```

---

# 4. Inheritance

`outline` **does NOT inherit**.

Each element must define its own outline—this is why focus rings apply only to the focused element.

---

# 5. Most common usage: focus rings

This is where `outline` is used 95% of the time.

Browser default:

```css
:focus {
  outline: 1px auto -webkit-focus-ring-color;
}
```

Custom focus ring:

```css
button:focus-visible {
  outline: 3px solid var(--color-primary-500);
  outline-offset: 3px;
}
```

---

# 6. `outline` vs `border`

|Feature|Outline|Border|
|---|---|---|
|Affects layout|No|Yes|
|Takes space|No|Yes|
|Part of box model|No|Yes|
|Can overlap content|Yes|No|
|Common use|Focus indicators|UI styling|
|Offset available|Yes|No|

---

# 7. `outline-offset` explained

Moves outline away from the element’s edge.

```css
button:focus-visible {
  outline: 2px solid #000;
  outline-offset: 4px;
}
```

This is a widely used accessibility tactic because:

- It prevents the ring from being hidden behind shadows
    
- It keeps focus clearly visible
    

---

# 8. Using `currentColor`

Often outline should match the text color:

```css
button {
  color: var(--color-primary);
}

button:focus-visible {
  outline: 2px solid currentColor;
}
```

---

# 9. Removing outlines (when necessary)

Removing outlines breaks accessibility unless you replace them.

Bad:

```css
*:focus {
  outline: none;
}
```

Correct:

```css
button:focus {
  outline: none;
}

button:focus-visible {
  outline: 2px solid var(--color-primary-500);
  outline-offset: 2px;
}
```

---

# 10. Advanced: Animated outlines

Example:

```css
.button:focus-visible {
  outline: 3px solid var(--color-primary-500);
  outline-offset: 4px;
  transition: outline-offset 120ms ease, outline-color 120ms ease;
}
```

---

# 11. Example components

### Accessible button

```css
.button {
  padding: 0.5rem 1rem;
  background: var(--color-primary-500);
  color: white;
}

.button:focus-visible {
  outline: 3px solid white;
  outline-offset: 2px;
}
```

### Text field

```css
input {
  border: 1px solid #cbd5e1;
  padding: 0.5rem;
}

input:focus-visible {
  outline: 2px solid var(--color-primary-600);
  outline-offset: 2px;
}
```
