---
tags: css, selector, fundamental
---

### 📖 Definition

The **ID selector** in CSS targets a single HTML element with a specific `id` attribute value. Since IDs must be unique within a page, an ID selector is used when you want to apply styles to exactly one element.

### 🖋️ Syntax

```css
#idName {
  /* styles */
}
```

- `#` is the prefix for IDs in CSS.
    
- `idName` is the exact value of the element’s `id` attribute (case-sensitive in XML, but not in HTML).
    

### 💡 Example

HTML:

```html
<p id="intro">This is an introduction paragraph.</p>
```

CSS:

```css
#intro {
  color: blue;
  font-weight: bold;
}
```

This will make the paragraph with `id="intro"` blue and bold.

### 🛠️ Tips & Best Practices

- **Uniqueness**: Ensure the `id` value is unique across the HTML document.
    
- **Specificity**: ID selectors have **high specificity** in CSS, which can override many other styles. Use sparingly to avoid difficulty in maintenance.
    
- **Not reusable**: Unlike class selectors, you can’t reuse IDs for multiple elements if you follow best practices.
    
- **Avoid overusing IDs for styling**: Consider using classes for repeated styles and reserve IDs for unique, specific cases.