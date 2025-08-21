---
tags: 
 - web dev
---

- Shorthand: `list-style: type position image;`
```css
ul {
  list-style: square inside url("icon.png");
}
```

- Removing Default Bullets/Numbers.
```css
ul, ol {
  list-style: none;   /* remove markers */
  margin: 0;          /* remove browser default spacing */
  padding: 0;
}
```
- Often used when making **navigation menus**.