---
tags:
  - css
  - property
  - fundamental
---

## 1. What `overflow` controls (core concept)

`overflow` determines **how content that exceeds an element’s box is handled**.

```css
overflow: visible | hidden | clip | scroll | auto;
```

It applies to **block containers**, **flex items**, and **grid items** (with caveats).

---

## 2. The five values (precise behavior)

### `visible` (default)

```css
overflow: visible;
```

- Content spills outside the box
    
- No scrollbars
    
- **Does not create a new formatting context**
    

**Quirk:**  
Children can visually escape the parent, but still affect layout in surprising ways.

---

### `hidden`

```css
overflow: hidden;
```

- Clips overflowing content
    
- No scrollbars
    
- **Creates a new block formatting context (BFC)**
    

**Quirks:**

- Hides focus rings, box-shadows, tooltips
    
- Often breaks `position: sticky`
    
- Clips absolutely positioned children
    

---

### `clip` (modern)

```css
overflow: clip;
```

- Like `hidden`, but **no scrolling at all**
    
- **Does NOT create a BFC**
    
- Cannot be scrolled programmatically
    

**Use when:** you want hard clipping without layout side effects.

---

### `scroll`

```css
overflow: scroll;
```

- Always shows scrollbars (even if not needed)
    
- Creates a scroll container
    

**Quirk:** Can cause layout shifts due to scrollbar width.

---

### `auto`

```css
overflow: auto;
```

- Scrollbars appear **only when needed**
    
- Most common value
    
- Creates a scroll container
    

---

## 3. Axis-specific control

```css
overflow-x: hidden;
overflow-y: auto;
```

**Important rule:**  
If one axis is `visible` and the other isn’t, `visible` behaves as `auto`.

---

## 4. Overflow and layout contexts (VERY important)

### A. Overflow creates a **Block Formatting Context (BFC)**

When overflow is not `visible`, the element:

- Stops margin collapse
    
- Contains floats
    
- Establishes a new layout boundary
    

```css
.container {
  overflow: hidden; /* classic clearfix hack */
}
```

---

### B. Overflow and `position: sticky` (common bug)

`position: sticky` **fails** if _any ancestor_ has:

```css
overflow: hidden | auto | scroll;
```

**Rule:**  
Sticky only works inside the **nearest scroll container**.

---

## 5. Overflow and scrolling behavior

### Scroll container definition

An element becomes scrollable when:

- `overflow` is `auto | scroll`
    
- AND content exceeds its box
    

This creates:

- a new scrolling context
    
- a new containing block for sticky children
    

---

## 6. Overflow with Flexbox & Grid (major gotcha)

### Flexbox quirk

Flex items have:

```css
min-height: auto; /* default */
```

This often prevents scrolling.

**Fix:**

```css
.flex-child {
  min-height: 0;
  overflow: auto;
}
```

This is one of the most common overflow bugs in modern CSS.

---

### Grid quirk

Same issue applies:

```css
.grid-child {
  min-height: 0;
  overflow: auto;
}
```

---

## 7. Overflow and absolute positioning

```css
.parent {
  overflow: hidden;
  position: relative;
}
```

- Absolutely positioned children **are clipped**
    
- Tooltips, dropdowns often get cut off
    

**Fix options:**

- Use portals (React)
    
- Move overlay outside clipping parent
    
- Use `position: fixed`
    

---

## 8. Overflow and focus / accessibility

### Focus ring clipping

```css
button:focus {
  outline: 2px solid blue;
}
```

If parent has `overflow: hidden`, the focus ring may be clipped.

**Fix:**

- Use `outline-offset`
    
- Avoid overflow on focusable parents
    
- Use Tailwind-style ring (`box-shadow`)
    

---

## 9. Overflow and painting / performance

- Scroll containers trigger **compositor layers**
    
- Excessive nested scroll areas hurt performance
    
- `overflow: hidden` can enable GPU compositing
    

---

## 10. Overflow vs `contain`

```css
contain: paint;
```

- Clips painting like `overflow: hidden`
    
- No scrolling
    
- Fewer side effects
    

---

## 11. Overflow + text

```css
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;
```

All three are required for ellipsis to work.

---

## 12. Practical patterns

### A. Scrollable panel

```css
.panel {
  max-height: 300px;
  overflow-y: auto;
}
```

---

### B. Flex layout scroll fix

```css
.layout {
  display: flex;
  height: 100vh;
}

.content {
  min-height: 0;
  overflow-y: auto;
}
```

---

### C. Prevent page scroll

```css
body {
  overflow: hidden;
}
```

---

## 13. One-page cheat sheet

|Problem|Fix|
|---|---|
|Sticky not working|Remove ancestor overflow|
|Flex scroll not working|`min-height: 0`|
|Tooltip clipped|Remove overflow / use portal|
|Focus ring cut|Use outline-offset or box-shadow|
|Ellipsis not showing|Add `white-space: nowrap`|

---

## 14. One sentence to remember

> **`overflow` is not just clipping — it changes layout, scrolling, and positioning behavior.**

---

If you want, I can:

- Analyze your layout and point out hidden overflow bugs
    
- Explain scrolling contexts visually
    
- Compare `overflow` vs `clip-path` vs `contain`