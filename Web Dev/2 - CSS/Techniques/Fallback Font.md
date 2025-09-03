---
tags: 
 - css
 - technique
 - font
 - fundamental
---

Fallback fonts are alternative fonts listed after the preferred font in the `font-family` CSS property. If the first font isn’t available, the browser tries the next one until it finds an installed or web-safe option.

---

## 🖋️ Syntax

```css
body {
  font-family: "Open Sans", "Helvetica Neue", Arial, sans-serif;
}
```

### How it works:

1. `"Open Sans"` → preferred custom font (e.g., from Google Fonts).
    
2. `"Helvetica Neue"` → secondary option if the first fails.
    
3. `Arial` → widely available system font fallback.
    
4. `sans-serif` → generic font family (last-resort, guaranteed to exist).
    

---

## 💡 Tips & Best Practices

- **Always end with a generic family** (`serif`, `sans-serif`, `monospace`, `cursive`, or `fantasy`).
    
- **Order matters**: list fonts from most desired → least.
    
- **Match style**: fallback fonts should look similar (e.g., don’t mix serif with sans-serif unless intentional).
    
- **Performance**: using system fonts as fallback ensures the page remains readable while custom fonts load.
    
- **Font loading strategy**: you can use `font-display: swap;` in @font-face to immediately use a fallback and swap when the custom font is ready.
    

---

✅ Example:

```css
h1 {
  font-family: "Roboto Slab", Georgia, "Times New Roman", serif;
}
```

- Tries **Roboto Slab** first.
    
- Falls back to **Georgia** if unavailable.
    
- Then **Times New Roman**.
    
- Finally, defaults to **serif** (browser/system default).
    