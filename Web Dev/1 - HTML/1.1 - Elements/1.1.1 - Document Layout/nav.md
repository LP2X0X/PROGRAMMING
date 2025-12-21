---
tags: 
  - html
  - element
  - layout
  - fundamental
---

## 🧭 `<nav>` Element — Overview

### 📌 Purpose

Represents a **section of navigation links** that help users move around a site or within a page.

It is a **semantic landmark** element, especially important for accessibility.

---

### 🧠 What Belongs in `<nav>`

- Primary site navigation
    
- Secondary navigation (sidebar, footer nav)
    
- Pagination links
    
- Table of contents
    
- Breadcrumbs (sometimes)
    

```html
<nav>
  <a href="/">Home</a>
  <a href="/blog">Blog</a>
  <a href="/about">About</a>
</nav>
```

---

### ❌ What Does NOT Belong in `<nav>`

- Random links inside content
    
- Action buttons (submit, delete)
    
- Social media icons unless they are navigation
    
- Inline links inside paragraphs
    

```html
<p>
  Read more <a href="/docs">here</a>.
</p>
<!-- ❌ not nav -->
```

---

### 🧩 `<nav>` vs Other Layout Elements

- `<header>`  
    Container for introductory content; may contain `<nav>`
    
- `<footer>`  
    Container for footer content; may contain `<nav>`
    
- `<section>`  
    Thematic grouping, not specifically navigation
    
- `<div>`  
    No semantic meaning
    

```html
<header>
  <nav>...</nav>
</header>
```

---

### ♿ Accessibility Benefits

- Screen readers announce `<nav>` as a **navigation landmark**
    
- Allows users to jump directly to navigation
    
- Multiple `<nav>` elements are allowed if they serve different purposes
    

Best practice:

```html
<nav aria-label="Primary navigation">
```

---

### 🧭 Multiple `<nav>` Elements

Allowed and encouraged when navigation roles differ.

```html
<nav aria-label="Main navigation">...</nav>
<nav aria-label="Footer navigation">...</nav>
```

---

### 🧠 `<nav>` Mental Model

> Use `<nav>` only when a group of links forms **a navigation system**, not just a list of links.

---

### ⚠️ Common Mistakes

- Wrapping all links on a page in `<nav>`
    
- Using `<nav>` for button groups
    
- Omitting accessible labels when multiple navs exist
    
- Styling `<div>` instead of using semantic `<nav>`
    

---

### 🧩 `<nav>` and Lists

Navigation links are often (but not required to be) in lists:

```html
<nav>
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/blog">Blog</a></li>
  </ul>
</nav>
```

---

### 🧠 Summary

- `<nav>` defines **navigation landmarks**
    
- Use it for **important navigational link groups**
    
- Improves accessibility and document structure
    
- Do not overuse it
    