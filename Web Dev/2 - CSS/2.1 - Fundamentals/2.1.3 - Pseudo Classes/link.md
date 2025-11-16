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
    

```ad-note
[[The Cascade]] is the reason this order matters: given the same specificity, later styles override earlier styles. If two or more of these states are true of one element at the same time, the last one can override the others. If the user hovers over a visited link, the hover styles take precedence. If the user activates the link (that is, clicks it) while hovering over it, the active styles take precedence.
```

```css
a:link    { color: blue; }
a:visited { color: purple; }
a:hover   { color: red; }
a:active  { color: orange; }
```

👉 Use it mainly for **base styling of links**; don’t over-customize `:visited` (browsers restrict it for privacy).

```ad-tip
You can also use the :any-link pseudo-class to target links that match either :link or :visited.
```
