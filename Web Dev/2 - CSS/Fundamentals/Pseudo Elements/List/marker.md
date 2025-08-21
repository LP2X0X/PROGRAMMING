---
tags: 
 - css
 - pseudo
 - element
 - list
---

- Modern bullet styling.
- Since CSS Lists Module Level 3, you can style list **markers directly**:

```css
li::marker {
  color: red;
  font-size: 1.2em;
  font-weight: bold;
  content: "👉 "; /* custom text/emoji */
}
```

- Works with `<ul>`, `<ol>`, and `<summary>` markers.
- Limited styling (no background, no transforms). For more control, use `::before`.