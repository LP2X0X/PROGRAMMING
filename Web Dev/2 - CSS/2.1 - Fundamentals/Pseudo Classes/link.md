---
tags: 
 - css 
 - pseudo
 - class
 - fundamental
---

**`:link`** = CSS pseudo-class that targets **unvisited hyperlinks** (anchors with `href` that the user hasn’t clicked yet).

### ✅ Tip & Best Practice:

- Always define `:link` **together with** `:visited`, `:hover`, and `:active` to keep link styling consistent.
    
- Follow the **LVHA order** (`:link`, `:visited`, `:hover`, `:active`) so styles cascade correctly.
    

```css
a:link    { color: blue; }
a:visited { color: purple; }
a:hover   { color: red; }
a:active  { color: orange; }
```

👉 Use it mainly for **base styling of links**; don’t over-customize `:visited` (browsers restrict it for privacy).
