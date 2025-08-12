---
tags: css, selector, fundamental
---

### 📖 Definition

The **universal selector** (`*`) in CSS targets **all elements** on a page, applying the same styles to every element unless overridden by more specific selectors.

### 🛠️ Syntax

```css
* {
  /* CSS properties */
}
```

### 💡 Example

HTML:

```html
<h1>Title</h1>
<p>This is a paragraph.</p>
<div>Content in a div</div>
```

CSS:

```css
/* Apply to every element */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
```

✅ This will remove default margins and paddings from all HTML elements and ensure consistent box sizing.

### 📝 Tips & Best Practices

- Useful for **CSS resets** and **global styling**.
    
- Be careful—overusing it can impact **performance** on large pages.
    
- Combine with attribute selectors or pseudo-classes for **more targeted resets**.