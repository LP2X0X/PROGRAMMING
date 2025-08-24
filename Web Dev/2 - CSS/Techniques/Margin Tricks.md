---
tags: 
 - css
 - technique
 - margin
---

### 🔸 Center without Flex/Grid

```css
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 0; bottom: 0; left: 0; right: 0;
  margin: auto; /* perfectly centers element */
}
```

### 🔸 Remove Last Item’s Margin

Common for menus or lists:

```css
ul li:last-child {
  margin-right: 0;
}
```

### 🔸 Equal Spacing with Gap

Instead of margins between flex/grid items, use:

```css
display: flex;
gap: 1rem;
```

✅ Avoids double margins at edges.

### 🔸 Negative Margin Fix

For grid/gutter hacks:

```css
.container {
  margin: -10px;
}
.item {
  margin: 10px;
}
```

✅ Ensures equal spacing inside without extra edge padding.
