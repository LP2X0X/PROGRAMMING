---
tags: 
 - css
 - at
 - rule
 - supports
---

In CSS, `@support` is likely a typo or shorthand for **`@supports`**, which is an **at-rule** used for **feature queries**. It lets you write CSS that only applies if the browser supports a certain CSS feature. This is very handy for progressive enhancement.

### Syntax

```css
@supports (property: value) {
  /* CSS rules if the feature is supported */
}
```

You can also use logical operators like `and`, `or`, and `not`:

```css
@supports (display: grid) and (display: flex) {
  /* CSS applies only if both grid and flex are supported */
}

@supports not (display: grid) {
  /* CSS for browsers that do NOT support grid */
}
```

```ad-note
It’s always wise to include fallback styles to ensure proper display in browsers that don’t support certain CSS features.
```

### Example

```css
.button {
  background-color: gray;
}

/* Apply fancy styles only if CSS variables are supported */
@supports (--custom: property) {
  .button {
    background-color: var(--main-color);
  }
}
```

**Key points:**

- It’s used to **detect CSS feature support**, not JavaScript features.
    
- Browsers that don’t understand `@supports` just ignore the block, so it’s safe to use.
    
- Can be nested like other CSS rules.
    