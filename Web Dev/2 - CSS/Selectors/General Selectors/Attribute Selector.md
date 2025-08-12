---
tags: css, selector, fundamental
---

### 📖 Definition

The **attribute selector** in CSS targets HTML elements based on the presence or value of their attributes. This allows styling elements without adding extra classes or IDs.

### 🛠️ Syntax

```css
/* Has attribute */
[elementName[attribute]] { }

/* Exact attribute value */
[elementName[attribute="value"]] { }

/* Attribute value contains word */
[elementName[attribute~="value"]] { }

/* Attribute value starts with */
[elementName[attribute^="value"]] { }

/* Attribute value ends with */
[elementName[attribute$="value"]] { }

/* Attribute value contains substring */
[elementName[attribute*="value"]] { }
```

### 💡 Example

HTML:

```html
<input type="text" placeholder="Enter name">
<input type="password" placeholder="Enter password">
<a href="https://example.com">Example</a>
```

CSS:

```css
/* Targets all input elements with a type attribute */
input[type] {
  border: 1px solid gray;
}

/* Targets only password input fields */
input[type="password"] {
  border: 2px solid red;
}

/* Targets links that start with https */
a[href^="https"] {
  color: green;
}

/* Targets links that end with .com */
a[href$=".com"] {
  font-weight: bold;
}
```

✅ This makes it possible to style elements dynamically based on their attributes, especially useful for form inputs and links.

### 📝 Tips & Best Practices

- Use attribute selectors for **dynamic content** where classes or IDs aren’t available.
    
- Keep in mind they have **higher specificity than element selectors** but lower than class selectors.
    
- For performance, avoid overly complex attribute selectors on very large DOM trees.