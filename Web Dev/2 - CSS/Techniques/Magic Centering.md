---
tags: 
 - css
 - technique
 - margin
---

### Horizontal Centering

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
