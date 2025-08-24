---
tags: 
 - css
 - technique
---


## **📖 Problem: Floats Collapse Parent Height**

When you float all the children inside a parent, the parent no longer stretches to contain them, because floated elements are **taken out of normal flow**.

Example:

```html
<div class="container">
  <div class="box left"></div>
  <div class="box right"></div>
</div>
```

```css
.container {
  border: 3px solid black;
}

.box {
  width: 100px;
  height: 100px;
}

.left { float: left; background: lightblue; }
.right { float: right; background: lightgreen; }
```

❌ The .container will collapse to 0 height, because floats don’t contribute to its height.

---

## **📖 Solution: Clearfix**


The clearfix hack uses a **pseudo-element** (::after) to insert an invisible element that clears the floats, forcing the parent to stretch properly.

```css
.clearfix::after {
  content: "";
  display: block;
  clear: both;
}
```

Apply it:

```html
<div class="container clearfix">
  <div class="box left"></div>
  <div class="box right"></div>
</div>
```

✅ Now, .container wraps its floated children again.

---

### **🧩 What clearfix does**

When we add:

```css
.clearfix::after {
  content: "";
  display: block;
  clear: both;
}
```

the parent suddenly has a **new last child** (the pseudo-element).

- ::after is inside the parent, so it **participates in normal flow**.
    
- clear: both forces this pseudo-element to be placed **below all floated siblings**.
    
- Because it’s below the floats, the parent now needs to extend its height down to fit it.
    
- Even though the pseudo-element has **no visible height** (content: ""), its position is still “pushed down” past the floats.
    
- Result: the parent stretches just enough to cover all the floats (because the cleared element is at the bottom).
    

---

### **⚡ Metaphor**


Think of floats like balloons tied to the inside walls of a box. If the box only measures its floor space, it won’t know balloons are inside and collapses flat.

  

Clearfix adds an **invisible stick** that touches the bottom of the lowest balloon, so the box expands to include it.

---

### **📖 Why the parent “keeps the original height”**

  

It doesn’t actually “keep” anything—it **regains height** because the pseudo-element forces the parent to **acknowledge the floated children’s vertical space** indirectly.

---

✅ In short: **The clearfix pseudo-element is not floated, so it’s in normal flow. Its cleared position forces the parent to stretch tall enough to include all the floated siblings.**