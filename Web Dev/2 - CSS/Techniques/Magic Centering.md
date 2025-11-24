---
tags: 
 - css
 - technique
 - margin
---

### Horizontal Centering

By setting a left and right margin of [[auto]], the margins will automatically expand as much as necessary to fill the remaining width available in the outer container.

```css
.block {
  width: 200px;
  margin: 0 auto; /* centers horizontally */
}
```

Works only for **block elements with a defined width**.

### Flexbox Centering

```css
.parent {
  display: flex;
}
.child {
  margin: auto; /* centers both vertically & horizontally */
}
```
