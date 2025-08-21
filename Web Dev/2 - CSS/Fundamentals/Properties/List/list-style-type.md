---
tags: 
 - css
 - property
 - list
---

- Controls the bullet/marker style.  
- Works on `<ul>` and `<ol>`.

```css
ul {
  list-style-type: disc;     /* default bullet */
  list-style-type: circle;   /* hollow circle */
  list-style-type: square;   /* square bullet */
}

ol {
  list-style-type: decimal;  /* 1, 2, 3... */
  list-style-type: decimal-leading-zero; /* 01, 02... */
  list-style-type: upper-roman; /* I, II, III */
  list-style-type: lower-alpha; /* a, b, c */
}
```

👉 [Full list of values](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-type) includes `armenian`, `georgian`, `hebrew`, `cjk-ideographic`, etc.
