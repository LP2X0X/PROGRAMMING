---
tags: html, attribute, global
---

The `role` attribute is used to define the **purpose** or **semantic meaning** of an HTML element, especially for **assistive technologies** like screen readers.

---

### 👓 **Why it's useful**

- Helps users with disabilities understand what an element _does_, not just how it _looks_.
    
- Provides semantic meaning when using non-semantic HTML (e.g., `<div>`, `<span>`).
    

---

### 📌 **Common Roles**

|Role|Description|
|---|---|
|`button`|Marks an element as a button|
|`navigation`|Identifies a section as navigation links|
|`main`|Marks the main content of the document|
|`banner`|Marks a site-wide header (1 per page)|
|`contentinfo`|Marks a footer with page or site info|
|`form`|Identifies a group of form elements|
|`dialog`|Represents a modal or popup dialog|
|`alert`|An important message, usually urgent|
|`tooltip`|Additional help text shown on hover/focus|
|`region`|Generic landmark for a group of content|

---

### ✅ **Examples**

```html
<!-- A navigation bar -->
<nav role="navigation">
  <a href="/">Home</a>
  <a href="/about">About</a>
</nav>

<!-- A div acting like a button -->
<div role="button" tabindex="0">Click me</div>
```

---

### 🔍 **When to Use It**

- Use when semantic HTML is **not enough** (e.g., `<div>` needs a defined function).
    
- If you're already using semantic tags (like `<nav>`, `<main>`, `<button>`), you usually **don’t need `role`** — browsers and screen readers already know their meaning.
    

---

### ⚠️ **Best Practice**

Use **semantic HTML** first. Only add `role` when:
- You're using a non-semantic tag for a specific purpose.
- You want to enhance accessibility for screen readers.