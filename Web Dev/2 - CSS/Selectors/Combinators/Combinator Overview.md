---
tags: css, selector, overview
---

### 📌 Description

A **combinator** is a special character or sequence of characters in CSS selectors that defines the **relationship between two selectors**.

- It **connects** two (or more) selectors to specify **how** elements are related in the HTML structure.
    
- Combinators determine **which elements are matched** based on hierarchy or positioning.
    

---

### 📍 Types of Combinators

#### 1. **Descendant Combinator** (space )

Selects elements **inside** another element, at **any depth**.

```css
div p {
  color: blue;
}
```

✅ Selects **all `<p>`** inside a `<div>` (children, grandchildren, etc.).

---

#### 2. **Child Combinator** (`>`)

Selects **direct children only**.

```css
div > p {
  color: red;
}
```

✅ Selects `<p>` elements **directly inside** `<div>`, but not deeper.

---

#### 3. **Adjacent Sibling Combinator** (`+`)

Selects the **first sibling** immediately following the specified element.

```css
h2 + p {
  font-style: italic;
}
```

✅ Selects the **first `<p>`** that comes immediately after `<h2>`.

---

#### 4. **General Sibling Combinator** (`~`)

Selects **all siblings** after the specified element.

```css
h2 ~ p {
  color: gray;
}
```

✅ Selects **all `<p>`** after `<h2>` (same parent).

---

### 📌 Key Points

- Combinators **do not match elements on their own** — they define the **relationship** between selectors.
    
- **Descendant (space)** is the most common but can be **less efficient** if overused.
    
- **Child (`>`)** is stricter and often more performant.
    
- **Sibling combinators** (`+` and `~`) are useful for styling elements based on nearby content.
    

---

### ⚠️ Best Practices

- Use **child combinator (`>`)** when you want **precise control** over hierarchy.
    
- Avoid deep descendant chains (e.g., `div div ul li span`) — they can be slow and fragile.
    
- For predictable layouts, prefer **structural CSS** over too many combinators.
    
