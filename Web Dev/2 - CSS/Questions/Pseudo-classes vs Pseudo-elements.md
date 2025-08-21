---
tags: 
 - css
 - distinguish
 - fundamental
---

## 🔹 **Pseudo-classes**

- **Define a special _state_ of an element.**
- They don’t create new DOM elements or parts; instead, they apply styles when certain conditions are met.
- Syntax: **one colon `:`**

### Examples:

```css
/* State-based */
button:hover { background: blue; }   /* when mouse is over */
input:focus { border: 2px solid red; } /* when focused */

/* Structural */
p:first-child { color: green; }     /* first child in parent */
li:nth-child(odd) { background: #eee; } /* odd rows */

/* Link state */
a:visited { color: purple; }
```

👉 Think of pseudo-classes as **“if this element is in X state, apply style Y.”**

---

## 🔹 **Pseudo-elements**

- **Style specific parts of an element**, as if you created a new sub-element in the DOM.
- They let you target _portions_ of content (first line, first letter, before/after, etc.).
- Syntax: **two colons `::`** (though older CSS allowed `:` for backward compatibility).

### Examples:

```css
p::first-line { font-weight: bold; }  /* first line only */
p::first-letter { font-size: 2em; }   /* first character */
div::before { content: "→ "; }        /* virtual element before content */
div::after { content: " ←"; }         /* virtual element after content */
```

👉 Think of pseudo-elements as **“pretend sub-elements that don’t exist in the DOM, but can be styled.”**

---

## 🔹 Quick Visual Analogy

- **Pseudo-class**: changes how the whole element behaves in a certain situation (like “this car turns blue when it’s raining”).
- **Pseudo-element**: lets you style a _part_ of that element (like “the left door of this car should be red”).

---

## ✅ **Summary**

- `:hover`, `:focus`, `:nth-child` → **pseudo-classes (state/condition)**
- `::before`, `::after`, `::first-letter` → **pseudo-elements (sub-parts)**