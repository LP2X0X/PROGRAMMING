---
tags: css, pseudo, element, fundamental
---

- **What it is:** A pseudo-element inserted **after** the content of an element, as its last child. Usually used for cosmetic purpose.
- **Usage:** Commonly used for styling effects, extra decorations, clearing floats, or inserting generated content.
- **Requirements:** Same as `::before`, it must have `content: ...;` to render.
- **Example:**
```css
/* Adds a star after every `<h1>` text. */
h1::after {
  content: " ★";
  color: gold;
}
```