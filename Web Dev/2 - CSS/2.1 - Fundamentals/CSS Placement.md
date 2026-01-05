---
tags: 
 - css
 - fundamental
---

CSS placement refers to **where and how CSS rules are defined and delivered to the browser**.

---

## 🧱 Inline CSS

### 📍 Definition

CSS written directly inside an HTML element using the `style` attribute.

```html
<button style="color: white; background: blue;">Click</button>
```

### ⚙️ Characteristics

- Highest specificity (except `!important`)
    
- Applies to a single element
    
- Not reusable
    
- Hard to maintain at scale
    

### ✅ Use Cases

- One-off overrides
    
- Styles generated dynamically via JavaScript
    
- Email templates (limited CSS support)
    

### ❌ Drawbacks

- Violates separation of concerns
    
- Poor readability
    
- Difficult to refactor
    

---

## 🧩 Internal (Embedded) CSS

### 📍 Definition

CSS written inside a `<style>` tag, usually in the `<head>`.

```html
<style>
  body {
    font-family: system-ui;
  }
</style>
```

### ⚙️ Characteristics

- Scoped to a single HTML document
    
- Cleaner than inline CSS
    
- Not shared across pages
    

### ✅ Use Cases

- Small static pages
    
- Prototypes
    
- Demos and teaching examples
    

### ❌ Drawbacks

- No caching across pages
    
- Not suitable for large applications
    

---

## 📁 External CSS

### 📍 Definition

CSS stored in a separate `.css` file and linked via `<link>`.

```html
<link rel="stylesheet" href="styles.css">
```

### ⚙️ Characteristics

- Best maintainability
    
- Browser caching
    
- Clean separation of concerns
    

### ✅ Use Cases

- Production websites
    
- Design systems
    
- Multi-page applications
    

### ❌ Drawbacks

- Additional HTTP request (usually negligible)
    
- Needs proper organization
    

---

## ⚙️ Programmatic CSS (JavaScript-Driven)

### 📍 Definition

CSS generated or applied dynamically via JavaScript.

```js
element.style.color = "red";
```

### ⚙️ Characteristics

- Runtime evaluation
    
- High flexibility
    
- Often framework-dependent
    

### ✅ Use Cases

- Dynamic theming
    
- Interactive UI states
    
- Runtime calculations
    

### ❌ Drawbacks

- Harder to debug
    
- Potential performance cost
    

---

## 🧩 CSS-in-JS

### 📍 Definition

CSS written in JavaScript files, scoped to components.

```jsx
const Button = styled.button`
  background: blue;
`;
```

### ⚙️ Characteristics

- Component-scoped
    
- Avoids global CSS collisions
    
- Enables dynamic styling via props
    

### ✅ Use Cases

- React and component-based architectures
    
- Design systems
    
- Theme-based styling
    

### ❌ Drawbacks

- Runtime overhead (some solutions)
    
- Tooling complexity
    
- Vendor lock-in
    

---

## 🎭 CSS Modules

### 📍 Definition

CSS files where class names are locally scoped by default.

```css
/* Button.module.css */
.primary {
  color: blue;
}
```

```js
import styles from "./Button.module.css";
```

### ⚙️ Characteristics

- Compile-time scoping
    
- No runtime cost
    
- Familiar CSS syntax
    

### ✅ Use Cases

- React, Vue, Next.js
    
- Teams wanting CSS isolation without JS styling
    

### ❌ Drawbacks

- Slightly more verbose
    
- No dynamic logic inside CSS
    

---

## 🌐 User & Browser Styles (CSS Origins)

### 📍 Browser (User Agent) Styles

- Default styles applied by browsers
    
- Example: margins on `<body>`, font sizes
    

### 📍 User Styles

- Custom styles set by users (accessibility)
    

### 📍 Author Styles

- CSS written by developers
    

These participate in the **CSS cascade**.

---

## 🧠 Placement vs Cascade (Important Distinction)

| Concept      | Meaning                    |
| ------------ | -------------------------- |
| Placement    | Where CSS is defined       |
| Cascade      | How conflicts are resolved |
| Specificity  | Rule priority              |
| Source order | Last rule wins             | 

---

## 🧠 Best Practices Summary

| Scenario        | Recommended Placement    |
| --------------- | ------------------------ |
| Production site | External CSS             |
| Component UI    | CSS Modules or CSS-in-JS |
| One-off tweak   | Inline (sparingly)       |
| Prototyping     | Internal CSS             |

---

## 🧠 Mental Model

> CSS placement answers **“Where does the style live?”**  
> The cascade answers **“Which style wins?”**

---

## 🧠 One-Sentence Summary

> CSS can be applied inline, embedded, external, or programmatically, each with different trade-offs in maintainability, scope, and performance.
