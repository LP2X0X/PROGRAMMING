---
tags: 
  - css
  - property
  - text
  - fundamental
---

---

### Prerequisite
[[Formatting Context]]

---

- The **`text-align`** property sets the horizontal alignment of the **inline-level content** inside a **block element** or table-cell box.
    
- Works within a **block formatting context**
    

```css
.container {
  text-align: center;
}
```

```ad-important
`text-align` **affects the inline-level children** of an element.
```

---

## 📦 What it affects

- Text nodes
    
- Inline elements (`span`, `a`)
    
- Inline-block elements
    
- Inline content inside table cells
    

---

## 🚫 What it does NOT affect

- Block-level elements
    
- Element positioning or layout flow
    
- Flexbox or Grid item alignment
    
- Absolute positioning
    

---

## 📐 Common values

- `left` — align to start edge (default in LTR)
    
- `right` — align to end edge
    
- `center` — center inline content
    
- `justify` — distribute text across line width
    
- `start` / `end` — writing-mode aware
    

---

## 🧠 Inheritance

- `text-align` **inherits**
    
- Child elements use the computed value unless overridden
    

---

## 🧩 Typical use cases

- Centering text in headers
    
- Aligning inline icons with text
    
- Centering inline buttons or links
    
- Aligning content in table cells
    

---

## ⚙️ Interaction with layout systems

- Works in **normal flow**
    
- Ignored by:
    
    - Flexbox alignment rules
        
    - Grid alignment rules
        
    - Positioned layout (`absolute`, `fixed`)
        

---

## 🧠 Mental model

> **`text-align` aligns inline content inside a container, not the container itself**

---

## ✅ Key takeaway

Use `text-align` for **text and inline content alignment**.  
Use layout properties (flex, grid, margins) for **element positioning**.


----

### Pitfall

An **inline `<a>` element cannot “text-align itself”**, because it does not establish a formatting context for alignment.

Let’s break this down rigorously.

---

## 1. Why `text-align` does not apply _to the link itself_

### Default behavior

```css
a {
  display: inline;
}
```

An inline element:

- Is **only as wide as its content**
    
- Does **not create a box that spans available width**
    
- Has **no extra inline space** inside itself to align text within
    

So this is meaningless:

```css
a {
  text-align: center; /* no visible effect */
}
```

Because there is nothing _to align against_.

The text already fills the inline box.

---

## 2. Why `text-align: center` on the **parent** works

```css
body {
  text-align: center;
}
```

This aligns **inline-level children**, including:

- text nodes
    
- inline `<a>`
    
- inline-block elements
    

The browser aligns the **inline formatting context**, not the link itself.

Visually:

```
|----------------------------------|
|        [ JOIN NOW ]               |
|----------------------------------|
```

The link is centered _within the parent’s line box_.

---

## 3. When `text-align` _would_ work on the link

You must make the link **generate a box with width**:

### Option A — `inline-block`

```css
a {
  display: inline-block;
  width: 200px;
  text-align: center;
}
```

Now the anchor:

- Has a box
    
- Has extra inline space
    
- Can align text inside itself
    

### Option B — `block`

```css
a {
  display: block;
  text-align: center;
}
```

This aligns text **inside** the anchor, but the anchor itself will still be left-aligned unless centered by layout rules.

---

## 4. Important distinction (common confusion)

|Goal|Correct tool|
|---|---|
|Center the **link element**|layout (`margin: auto`, flex, grid)|
|Center **text inside** the link|`text-align`|
|Center an **inline link**|`text-align` on parent|

---

## 5. Mental model (use this)

> **`text-align` never moves elements.  
> It aligns inline content inside a formatting context.**

Your button is centered because:

- `<a>` is inline
    
- `body` is the formatting context
    
- The anchor behaves like a word in a sentence
    

---

## Final takeaway

You are absolutely right:

> An inline link is only as big as its content,  
> so it cannot meaningfully align itself with `text-align`.

Centering it is **the parent’s job**, not the link’s.

This is not a quirk — it is fundamental to how inline formatting works.