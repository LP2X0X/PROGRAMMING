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

| Property | Meaning                                       |
| -------- | --------------------------------------------- |
| `top`    | Distance from top edge of containing block    |
| `right`  | Distance from right edge of containing block  |
| `bottom` | Distance from bottom edge of containing block |
| `left`   | Distance from left edge of containing block   |

✅ Only works with `relative`, `absolute`, `fixed`, `sticky`.  
✅ Can use **one side** or **two sides (stretches)**.  
✅ Negative values allowed.

</br>
----

# Notes

## 1. Understand the auto value of theses property

> **Your element is not being centered by Flexbox.**  
> It is being positioned by **absolute positioning rules**, and Flexbox is only acting as the _containing block_.

---

## What is actually happening

You likely have something equivalent to this:

```css
.container {
  display: flex;
  align-items: center;
  position: relative;
}

.item {
  position: absolute;
  right: 4px;
}
```

### Why the element appears vertically centered

When an element is `position: absolute` **and you only set `right`**, then:

- `top` = `auto`
    
- `bottom` = `auto`
    

According to CSS positioning rules:

> If both `top` and `bottom` are `auto`, the element stays at its **static position**.

### What is the _static position_ here?

👉 The static position is where the element **would have been placed if it were not absolute**.

Because:

- The container is `display: flex`
    
- `align-items: center` is set
    

The element’s **hypothetical flex position** is vertically centered.

So:

- Flexbox computes layout first
    
- Then `position: absolute` removes the element from layout
    
- But its **static position** is preserved
    
- `right: 4px` shifts it horizontally only
    

---

## Important clarification

|Property|Effect|
|---|---|
|`align-items: center`|❌ Not aligning the absolute element|
|Static position|✅ Provides the reference vertical position|
|`right: 4px`|✅ Overrides horizontal placement only|

This is why it _looks_ like Flexbox is centering it.

---

## Proof: break the illusion

Add this:

```css
.item {
  position: absolute;
  top: 0;
  right: 4px;
}
```

➡️ The element will **snap to the top**, ignoring `align-items`.

Or remove flex:

```css
.container {
  display: block;
}
```

➡️ The element will no longer be centered vertically.

---

## Mental model (remember this)

> **Absolute positioning starts from the element’s static position unless you override it.**

Flexbox can still influence that static position — **indirectly**, not by alignment.

---

## Summary

- `align-items` does **not** align absolutely positioned elements
    
- The element appears centered because:
    
    - Its static position is calculated using Flexbox
        
    - Only `right` is overridden
        
- This is **not a Flexbox feature**, but a CSS positioning rule
    

---

### Final takeaway

> If an absolutely positioned element appears aligned by Flexbox, you are seeing the effect of its **static position**, not Flexbox alignment itself.