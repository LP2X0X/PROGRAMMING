---
tags:
  - css
  - pseudo
  - class
  - term
---

### 1. What is a pseudo-class?

A **pseudo-class** (e.g., `:hover`, `:focus`, `:checked`) represents a _special state_ of an element.  
It does not create new elements — it’s a **condition applied to an element**.

---

### 2. Pseudo-class can trigger styles on other elements

By combining selectors, you can make one element’s state change another element’s styles.

Example with sibling:

```css
.corner-hotspot:hover ~ .corner-btn {
  opacity: 1;
}
```

- Condition: `.corner-hotspot` is hovered
    
- Target styled: its sibling `.corner-btn`
    

---

### 3. Rule of thumb

- **Rightmost selector = element that gets styled**
    
- **Left selectors + pseudo-classes = conditions** that must be true
    

Think of it as:

> “When X is in state Y, apply styles to Z.”