---
tags:
  - css
  - selector
  - fundamental
---

### 📖 Definition

The **class selector** in CSS targets one or more HTML elements that share the same `class` attribute value. Classes are reusable, making them ideal for applying the same styles to multiple elements across a page.

### 🖋️ Syntax

```css
.className {
  /* styles */
}
```

- `.` is the prefix for class selectors in CSS.
    
- `className` is the value of the element’s `class` attribute (case-sensitive in XML, but not in HTML).
    

````ad-tip
To select element with both classes, do it like this:
```css
.class1.class2 {
/*Your rules here...*/
}
```
````

### 💡 Example

HTML:

```html
<p class="highlight">This is highlighted text.</p>
<p class="highlight">This is also highlighted text.</p>
```

CSS:

```css
.highlight {
  background-color: yellow;
  font-weight: bold;
}
```

Both `<p>` elements with `class="highlight"` will have a yellow background and bold text.

### 🛠️ Tips & Best Practices

- **Reusability**: Use classes for styles that apply to multiple elements.
    
- **Multiple classes**: An element can have more than one class (e.g., `class="btn primary"`).
    
- **Lower specificity**: Class selectors have lower specificity than ID selectors, making them easier to override if needed.
    
- **Semantic naming**: Use descriptive, meaningful names (e.g., `.error-message` instead of `.red`).