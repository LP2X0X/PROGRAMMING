---
tags: 
 - css
 - advance
---

CSS **custom properties** (aka CSS variables) let you define reusable values directly in CSS. They are powerful for theming, consistency, and dynamic updates.

---

## 🔹 Syntax

**Define:**

```css
:root {
  --main-color: #3498db;
  --padding-size: 1rem;
}
```

**Use:**

```css
button {
  background-color: var(--main-color);
  padding: var(--padding-size);
}
```

---

## 🔹 Key Features

- **Scope**
    
    - Declared in `:root` → global.
        
    - Declared in an element → scoped to that element and its children.
        

```css
.card {
  --card-bg: white;
  background: var(--card-bg);
}
```

---

- **Fallbacks**
    

```css
color: var(--undefined-color, black);
```

---

- **Dynamic Updates (JavaScript)**
    

```js
document.documentElement.style.setProperty('--main-color', 'tomato');
```

---

- **Inheritance**
    

```css
.parent {
  --font-size: 18px;
}
.child {
  font-size: var(--font-size); /* 18px */
}
```

---

- **Runtime, not pre-processed**
    
    - Unlike Sass/LESS, CSS variables **exist in the browser**.
        
    - Work with **media queries, states, and JS**.
        

```css
@media (prefers-color-scheme: dark) {
  :root {
    --bg-color: black;
    --text-color: white;
  }
}
```

---

## 🔹 Common Practices, Tips & Tricks

✅ **Keep variables in `:root` for global usage**

```css
:root {
  --primary-color: #4caf50;
  --font-body: "Inter", sans-serif;
}
```

---

✅ **Use semantic naming**  
Bad 👎

```css
--blue: #3498db;
```

Good 👍

```css
--color-primary: #3498db;
--color-primary-light: #5dade2;
--color-primary-dark: #2e86c1;
```

---

✅ **Combine with calc() for flexible layouts**

```css
:root {
  --gutter: 16px;
}
.container {
  padding: calc(var(--gutter) * 2);
}
```

---

✅ **Theme switching made easy**

```css
:root {
  --bg: white;
  --text: black;
}
[data-theme="dark"] {
  --bg: black;
  --text: white;
}
body {
  background: var(--bg);
  color: var(--text);
}
```

Switch theme with JS:

```js
document.documentElement.setAttribute("data-theme", "dark");
```

---

✅ **Use fallbacks when necessary**

```css
button {
  background: var(--btn-bg, #666); /* default if not set */
}
```

---

✅ **Debugging trick with `outline`**  
Temporarily assign a custom property to outline elements:

```css
* {
  outline: 1px solid var(--debug-color, red);
}
```

---

## 🔹 Why Use Them?

- Centralized **design tokens**.
    
- **Dynamic theming** (light/dark, brand colors).
    
- Work with **JavaScript** at runtime.
    
- Improve **maintainability & scalability**.
    