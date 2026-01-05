---
tags: 
 - css
 - property
 - text
 - fundamental
---

## 🔤 What the `font-size` Property Does

- `font-size` defines the **size of text glyphs**
    
- It also affects **relative units**:
    
    - `em` (relative to parent)
        
    - `rem` (relative to root)
        
    - `%` (relative to parent)
        

```css
p {
  font-size: 16px;
}
```

---

## 📏 Accepted Units for `font-size`

### 🧮 Absolute Units

- `px` — device-independent CSS pixels
    
- `pt`, `cm`, `mm` — rarely used on screens
    

### 🔁 Relative Units

- `%` — relative to parent font size
    
- `em` — relative to parent
    
- `rem` — relative to `html`
    
- `vw` — relative to viewport width
    

---

## 🧠 Inheritance Behavior

- `font-size` **is inherited**
    
- If not explicitly set, elements inherit from their parent
    

```css
body {
  font-size: 16px;
}

p {
  /* inherits 16px */
}
```

---

## 🧩 Why `font-size` Matters Beyond Text

- `em`-based spacing scales with text
    
- Component proportions stay consistent
    
- Typography becomes predictable
    

```css
.card {
  padding: 1.5em;
}
```

---

## ⚠️ Why You Should NOT Set `font-size` on `html`

### 🚫 Overrides User Accessibility Preferences

- Browsers allow users to set a **default font size**
    
- `html { font-size: ... }` **redefines the root unit**, in other word overwrite user-setted font size (because [[1 - The Cascade#1. **Stylesheet Origin**|author declaration has higher priority than user declaration]])
    
- This can shrink or enlarge text **against user intent**
    

```css
html {
  font-size: 62.5%; /* forces 1rem = 10px */
}
```

This pattern:

- Breaks expected scaling
    
- Harms low-vision users
    
- Reduces accessibility compliance
    

---

### 🚫 Changes the Meaning of `rem` Globally

- `rem` is a **root-based contract**
    
- Altering `html` breaks assumptions across:
    
    - Third-party components
        
    - Design systems
        
    - Browser defaults
        

Unexpected side effects are common.

---

### 🚫 Makes Global Scaling Harder to Reason About

- Changing `html` font size affects:
    
    - Typography
        
    - Layout spacing
        
    - Component dimensions
        

This couples unrelated concerns.

---

## ✅ Why Setting `font-size` on `body` Is Preferred

### 🎯 Respects User Defaults

```css
html {
  font-size: 100%;
}

body {
  font-size: 1rem;
}
```

- User-defined base size is preserved
    
- Accessibility remains intact
    

---

### 🧩 Keeps `rem` Stable and Predictable

- `rem` continues to mean “user default”
    
- Global design tokens stay reliable
    
- External libraries behave correctly
    

---

### 🧠 Clear Separation of Concerns

| Selector | Responsibility         |
| -------- | ---------------------- |
| `html`   | Root, rem reference    |
| `body`   | Application typography | 

---

## 🧪 Recommended Modern Pattern

```css
html {
  font-size: 100%;
}

body {
  font-size: 1rem;
  line-height: 1.5;
}
```

Optional responsive scaling:

```css
@media (min-width: 768px) {
  body {
    font-size: 1.125rem;
  }
}
```

---

## ❌ When Setting `html` Font Size MAY Be Acceptable

- Controlled environments (kiosks, dashboards)
    
- Internal tools with no accessibility requirements
    
- Experimental prototypes
    

Even then, document the decision clearly.

---

## 🧠 Final Mental Model

> `html` defines the **unit system**  
> `body` defines the **typography system**

---

## 🧠 One-Sentence Summary

> `font-size` should usually be set on `body`, not `html`, because changing the root font size redefines `rem`, overrides user preferences, and introduces unnecessary global side effects.