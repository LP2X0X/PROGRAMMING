---
tags: 
 - css
 - property
 - input
---

### `accent-color` — CSS Property Notes

**Purpose**  
`accent-color` allows you to set the highlight color used by certain native form controls. It affects the “accent” parts that browsers normally color with a default system color.

---

### Controls Affected

Applies to **form elements with native UI parts**, including:

- `input[type="checkbox"]`
    
- `input[type="radio"]`
    
- `input[type="range"]`
    
- `progress`
    

It does **not** affect:

- Text inputs (`text`, `email`, etc.)
    
- Custom-styled controls (where native appearance is fully overridden)
    

---

### Syntax

```css
accent-color: <color>;
```

Examples:

```css
accent-color: blue;
accent-color: #3b82f6;
accent-color: hsl(217 91% 60%);
accent-color: auto;
```

---

### Common Usage

```css
:root {
  accent-color: #2563eb;
}
```

This sets a **global accent color** for all supported form controls.

Scoped usage:

```css
input[type="checkbox"] {
  accent-color: green;
}
```

---

### Values

- **`auto`** (default)  
    Uses the browser or OS-defined accent color.
    
- **Any valid CSS color**  
    Hex, RGB, HSL, named colors, `currentColor`, etc.
    

Example with `currentColor`:

```css
input {
  color: rebeccapurple;
  accent-color: currentColor;
}
```

---

### Behavior & Characteristics

- Declarative and **non-invasive**: you keep native accessibility, focus states, and keyboard behavior.
    
- Browser decides **exact rendering**; the property is a _hint_, not pixel-perfect styling.
    
- Works well with system themes (light/dark) and high-contrast modes.
    

---

### Accessibility Considerations

- Ensure sufficient contrast between the accent color and the control background.
    
- Avoid relying on accent color alone to convey meaning (e.g., error vs success).
    

---

### When to Use vs Not Use

**Use when:**

- You want brand-consistent form controls
    
- You want minimal CSS with native behavior intact
    

**Avoid when:**

- You need full visual control (use custom components instead)
    
- You must support very old browsers
    

---

### Browser Support (High-level)

- Supported in all modern evergreen browsers (Chrome, Edge, Firefox, Safari).
    
- Safe to use in production with graceful fallback to default colors.
    

---

### Practical Tip

Prefer `accent-color` over custom checkboxes/radios unless you need advanced visuals—it significantly reduces CSS complexity and improves maintainability.

If you want, I can also provide:

- Tailwind usage patterns
    
- Interaction with dark mode
    
- Comparison with fully custom form controls