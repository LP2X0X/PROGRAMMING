---
tags: 
 - css
 - technique
 - list
---

## 1. 🔹 Custom Bullets / Icons

Using `::before`:

```css
ul.custom li {
  list-style: none;
  position: relative;
}

ul.custom li::before {
  content: "✔";
  color: green;
  font-weight: bold;
  position: absolute;
  left: -1.5em;
}
```

You can replace with:

- Emojis → ✅, ⭐, 🔹
    
- Unicode → `\2022` (•)
    
- SVG or font icons (Font Awesome, etc.)
    

---

## 2. 🔹 Numbering Tricks with `counter()`

Custom ordered lists:

```css
ol.custom {
  list-style: none;
  counter-reset: item;
}

ol.custom li {
  counter-increment: item;
}

ol.custom li::before {
  content: counters(item, ".") " "; /* 1.1.1 style */
  font-weight: bold;
}
```

👉 Great for **nested outlines** (e.g., 1.2.3 headings).

---

## 3. 🔹 Horizontal Lists

Good for menus or inline nav:

```css
ul.inline {
  list-style: none;
  display: flex;
  gap: 1rem; /* modern spacing */
}

ul.inline li {
  display: inline;
}
```

---

## 4. 🔹 Styling Description Lists (`<dl>`)

Default:

```html
<dl>
  <dt>HTML</dt>
  <dd>A markup language</dd>
  <dt>CSS</dt>
  <dd>Stylesheets</dd>
</dl>
```

Custom style:

```css
dt {
  font-weight: bold;
  color: navy;
}
dd {
  margin-left: 2em;
  color: gray;
}
```

---

## 5. 🔹 Nested Lists

```css
ul ul {
  list-style-type: circle;
}
ul ul ul {
  list-style-type: square;
}
```

You can **indent** or reset margins for readability.

---

## 6. 🔹 Accessibility & Best Practices

- Use `<ul>` for unordered items, `<ol>` when order matters.
    
- Don’t remove semantics by replacing lists with `<div>`.
    
- If removing bullets, keep semantic list structure for screen readers.
    
- Use `role="list"` if you override too much with custom styling.
    

---

## 7. 🔹 Advanced Effects

### Hanging Indent

Align multiline list items neatly:

```css
ul {
  list-style-position: outside;
  padding-left: 2em;
}
li {
  text-indent: -1em;
  padding-left: 1em;
}
```

### Fancy Counters

Roman numerals in uppercase red:

```css
ol.roman {
  list-style: upper-roman;
}
ol.roman li::marker {
  color: crimson;
  font-weight: bold;
}
```

### Grid/List Hybrids

Turn lists into **cards**:

```css
ul.cards {
  list-style: none;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1rem;
}
ul.cards li {
  background: #f0f0f0;
  padding: 1rem;
  border-radius: 10px;
}
```
