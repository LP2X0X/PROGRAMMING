---
tags: 
 - css
 - property
 - absolute
 - position
---

# 📘 CSS `top`, `right`, `bottom`, `left`

These are **offset properties** that tell the browser _where to place an element_ **when its `position` is not `static`**.  
👉 They do **nothing** on elements with `position: static` (the default).

---

## 1. When they apply

They only have effect if the element has one of these positions:

- `relative` → offsets the element **from where it would normally be**, but still keeps its space in the flow.
    
- `absolute` → offsets relative to the **nearest positioned ancestor** (not static).
    
- `fixed` → offsets relative to the **viewport**.
    
- `sticky` → offsets relative to the **nearest scrollable ancestor**, but only when “stuck”.
    

---

## 2. Behavior of each property

### 🔹 `top`

- Defines the distance from the **top edge of containing block**.
    

```css
.element {
  position: absolute;
  top: 20px; /* 20px down from top */
}
```

### 🔹 `right`

- Defines the distance from the **right edge of containing block**.
    

```css
.element {
  position: absolute;
  right: 10px; /* 10px away from right side */
}
```

### 🔹 `bottom`

- Defines the distance from the **bottom edge of containing block**.
    

```css
.element {
  position: absolute;
  bottom: 15px; /* 15px above bottom */
}
```

### 🔹 `left`

- Defines the distance from the **left edge of containing block**.
    

```css
.element {
  position: absolute;
  left: 30px; /* 30px from left */
}
```

---

## 3. Special cases and tricks

### 🔸 Over-constrained axes

If you set **both sides** (e.g., `left` and `right`):

- Default width is `auto`.
    
- Browser stretches/shrinks element to fit that space.
    

```css
.element {
  position: absolute;
  left: 0;
  right: 0;
}
/* element stretches full width */
```

Same applies to `top + bottom`.

---

### 🔸 Centering with `top/left` + `transform`

Common centering trick:

```css
.element {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

👉 Centers element perfectly inside its containing block.

---

### 🔸 Negative values

Offsets can be negative.

```css
.element {
  position: absolute;
  top: -20px; /* moves 20px above container */
}
```

---

### 🔸 With `relative`

```css
.relative-box {
  position: relative;
  top: 10px;
}
```

👉 Moves element 10px down, **but its reserved spot stays the same**, so it can overlap siblings.

---

## 4. Summary Table

|Property|Meaning|
|---|---|
|`top`|Distance from top edge of containing block|
|`right`|Distance from right edge of containing block|
|`bottom`|Distance from bottom edge of containing block|
|`left`|Distance from left edge of containing block|

✅ Only works with `relative`, `absolute`, `fixed`, `sticky`.  
✅ Can use **one side** or **two sides (stretches)**.  
✅ Negative values allowed.