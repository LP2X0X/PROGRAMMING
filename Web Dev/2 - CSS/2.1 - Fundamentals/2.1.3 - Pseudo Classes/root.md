---
tags: 
 - css
 - pseudo
 - class
---

The **`:root`** pseudo-class represents the **topmost element** of the document — in HTML, that’s always the **`<html>`** element.

It behaves like selecting `html`, but with **higher specificity**, making it useful for global variables and theme definitions.

---

## **Why use `:root`?**

### **1. Higher specificity than `html`**

- Styles defined in `:root` override the same properties written on `html`.
    

```ad-note
You can override root font size in DevTools to test the accessibility of your web page.
```

### **2. Best place to define CSS variables**

```css
:root {
  --main-color: #4a90e2;
  --font-size-base: 16px;
}
```

- CSS variables declared in `:root` become **global** and accessible from anywhere in the stylesheet.
    

### **3. Helpful for theming**

```css
:root[data-theme="dark"] {
  --bg: #000;
  --text: #fff;
}

:root[data-theme="light"] {
  --bg: #fff;
  --text: #000;
}
```

---

## **Examples**

### **Global variable usage**

```css
:root {
  --spacing: 1.5rem;
}

button {
  padding: var(--spacing);
}
```

### **Changing variables with media queries**

```css
@media (min-width: 768px) {
  :root {
    --font-size-base: 18px;
  }
}
```

---

## **Key notes**

- `:root` = the document's root element (in HTML → `<html>`).
    
- Preferred over `html` when defining global CSS variables.
    
- Often used for **global theming**, **responsive variable adjustments**, and **root-level styling**.
    