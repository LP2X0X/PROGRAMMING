---
tags: 
 - css
 - note
 - pseudo
 - class
---

## 1️⃣ Same rules for multiple pseudo-classes (most common)

Use a **comma-separated selector list**.

```css
button:hover,
button:focus,
button:active {
  background-color: var(--color-primary);
}
```

✔ Applies the same styles to **any** of those states.

---

## 2️⃣ Chaining pseudo-classes (AND condition)

Use **no comma** to require **all conditions at the same time**.

```css
button:hover:focus {
  outline: 2px solid currentColor;
}
```

✔ Only applies when the button is **hovered AND focused**.

---

## 3️⃣ Using `:is()` (recommended, cleaner)

Groups multiple selectors without repetition.

```css
button:is(:hover, :focus, :active) {
  background-color: var(--color-primary);
}
```

✔ Shorter  
✔ Easier to maintain  
✔ Modern browsers supported

---

## 4️⃣ Using `:where()` (zero specificity)

Same as `:is()`, but **adds no specificity**.

```css
button:where(:hover, :focus, :active) {
  background-color: var(--color-primary);
}
```

✔ Best for base styles / resets  
✔ Easy to override later

---

## 5️⃣ Combining with class or attribute selectors

```css
.btn:is(:hover, :focus-visible) {
  box-shadow: 0 0 0 3px color-mix(in srgb, currentColor 30%, transparent);
}
```

---

## 6️⃣ Common mistake ❌

```css
/* WRONG */
button:hover:focus,
:active { }
```

`:active` here applies to **everything**, not just `button`.

---

## Specificity comparison

| Syntax                         | Specificity            |
| ------------------------------ | ---------------------- |
| `button:hover, button:focus`   | Normal                 |
| `button:is(:hover, :focus)`    | Highest inside `:is()` |
| `button:where(:hover, :focus)` | Zero from `:where()`   |

---

## When to use what

| Situation                   | Best choice     |
| --------------------------- | --------------- |
| Simple grouping             | Comma-separated |
| Clean modern CSS            | `:is()`         |
| Low-specificity base styles | `:where()`      |
| Combined states             | Chaining        |

---

### Key takeaway

> Use **commas** for **OR**, **chaining** for **AND**, and prefer **`:is()` / `:where()`** for clean, maintainable multi-state styling.