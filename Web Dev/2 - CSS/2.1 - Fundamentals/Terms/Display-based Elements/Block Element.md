---
tags: 
 - css
 - term
 - element
 - type
---

# 🔹 What is a Block Element?

A **block-level element** is an element that:

- **Always starts on a new line**.
    
- By default, takes up the **full available width of its parent** (unless width is set).
    
- Can contain **other block or inline elements**.
    
- Respects **all margins and paddings** (top, right, bottom, left).
    
- Has width and height you can explicitly control.
    

---

# 🔹 Examples of Block Elements

Common block elements:

- **Structural**: `<div>`, `<section>`, `<article>`, `<header>`, `<footer>`, `<aside>`, `<nav>`
    
- **Content**: `<p>`, `<h1>`–`<h6>`, `<ul>`, `<ol>`, `<li>`, `<table>`, `<form>`
    
- **Others**: `<hr>`, `<address>`, `<pre>`, `<blockquote>`
    

---

# 🔹 Behavior of Block Elements

### 1. Line Breaking

- Each block element **forces a line break before and after it**.
    

```html
<p>Paragraph 1</p>
<p>Paragraph 2</p>
```

Output → Paragraph 2 appears **below** Paragraph 1, not next to it.

---

### 2. Width & Height

- Default: block takes up **100% of parent’s width**.
    
- You can set **explicit width/height**.
    

```css
div {
  width: 300px;
  height: 100px;
  background: lightblue;
}
```

---

### 3. Margin & Padding

- **All margins work** (top, right, bottom, left).
    
- **Margin collapsing** can happen between vertical margins of two block elements.
    

Example:

```css
p {
  margin: 20px 0;
}
```

If two `<p>` are stacked, the gap isn’t 40px → it collapses to 20px.

---

### 4. Nesting

- Block elements can contain **both inline and block elements**.
    

```html
<div>
  <h1>Heading</h1>
  <p>Some <span>inline</span> text inside a block.</p>
</div>
```

---

### 5. Alignment

- Block elements align relative to their **container’s width**.
    
- Use `margin: auto;` to center a block horizontally if width is set.
    

```css
div {
  width: 200px;
  margin: 0 auto; /* centers block */
}
```

---

### 6. Formatting Context

- Block elements participate in **block formatting context (BFC)**.
    
- Inside, boxes are stacked vertically.
    
- BFC affects float clearing, overflow, and margin collapsing.
    

---

# 🔹 Differences vs Inline Elements

|Feature|Inline|Block|
|---|---|---|
|Starts new line?|❌ No|✅ Yes|
|Default width|Content only|100% parent|
|Can set width/height?|❌ No|✅ Yes|
|Margin top/bottom work?|❌ Ignored|✅ Works|
|Nesting|Only inline|Inline + Block|

---

# 🔹 Special Cases

1. **Replaced elements** (like `<img>`, `<input>`) behave a bit like block in terms of dimensions, but default display varies.
    
2. You can turn **any element into a block** with CSS:
    
    ```css
    span {
      display: block;
    }
    ```
    
    → Now `<span>` behaves like a `<div>`.
    

---

# 🔹 When to Use Block Elements

✅ Structure/layout (`div`, `section`, `article`)  
✅ Headings (`h1`–`h6`)  
✅ Paragraphs (`p`)  
✅ Lists (`ul`, `ol`, `li`)  
✅ Forms (`form`, `fieldset`)

---

⚡ Quick summary:

- Block = **big boxes that stack vertically**.
    
- Inline = **words/small elements flowing in text**.
    
- Inline-block = mix: inline flow, but block box sizing.
    