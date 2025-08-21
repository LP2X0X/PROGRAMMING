---
tags: 
 - css
 - property
 - list
---

- Use an image instead of default bullets.

```css
ul {
  list-style-image: url("star.png");
}
```

⚠️ Downside: can’t easily resize/recolor. Often better with `::marker` or `::before`.
