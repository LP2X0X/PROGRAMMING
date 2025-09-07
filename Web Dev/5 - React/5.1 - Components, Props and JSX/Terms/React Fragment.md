---
tags:
  - react
  - term
  - fundamental
---

### 🔷 React Fragment – What It Is & Why Use It

A **React Fragment** lets you group **multiple JSX elements** without adding an extra DOM node (like a `<div>`). It helps keep the HTML structure clean.

---

### ✅ Basic Syntax

```jsx
import React from 'react';

function MyComponent() {
  return (
    <React.Fragment>
      <h1>Hello</h1>
      <p>This is a paragraph.</p>
    </React.Fragment>
  );
}
```

---

### ✅ Shorthand Syntax

```jsx
function MyComponent() {
  return (
    <>
      <h1>Hello</h1>
      <p>This is a paragraph.</p>
    </>
  );
}
```

> `<>` and `</>` are the same as `<React.Fragment>`, but shorter.

---

### 🧠 Why use a Fragment instead of a `<div>`?

* It **doesn’t add an extra element** to the DOM.
* Useful when JSX requires a single parent (e.g., in `.map()` or `return`) but you **don’t want extra wrappers**.
* Helps **avoid breaking layouts** (e.g., in table rows or grids).

---

### ✅ Example in `.map()`:

```jsx
{items.map(item => (
  <React.Fragment key={item.id}>
    <dt>{item.term}</dt>
    <dd>{item.description}</dd>
  </React.Fragment>
))}
```

> You still need a `key` even in fragments during `.map()`.