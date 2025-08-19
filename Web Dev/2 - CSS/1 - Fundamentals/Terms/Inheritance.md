---
tags:
  - css
  - term
  - fundamental
---

Here’s a clear breakdown of **inheritance in CSS** with definitions, examples, tips, and best practices:

---

### 🔹 **Definition**  
**Inheritance** in CSS is the mechanism where certain property values are **passed down** from a parent element to its child elements.

- **Inheritable properties**: Mostly text-related (e.g., `color`, `font-family`, `line-height`).
    
- **Non-inheritable properties**: Layout and box-model-related (e.g., `margin`, `border`, `padding`, `width`, `height`). Properties with *browser default* value(?).
    

---

### 🔹 **How It Works**  
When you don’t set a property for a child element, the browser checks if it can **inherit** it from its parent.  
If the property is inheritable, the child gets the parent’s computed value.

Example:

```css
body {
  color: blue; /* inherited by all text inside body */
}
p {
  /* color is not defined here, so it inherits from body */
}
```

---

### 🔹 **Forcing Inheritance**  
Even for properties that **don’t normally inherit**, you can force inheritance with:

```css
property: inherit;
```

Example:

```css
div {
  border: inherit; /* force child to use parent's border */
}
```

---

### 🔹 **Initial and Unset Keywords**

- `initial` → Resets property to its default CSS value.
    
- `unset` → Acts like `inherit` if the property is inheritable, otherwise acts like `initial`.
    

Example:

```css
span {
  color: unset; /* inherits or resets depending on property */
}
```

---

### 🔹 **Examples**

```css
/* Inherited example */
body {
  font-family: Arial, sans-serif;
}

p {
  font-size: 16px; /* will inherit font-family from body */
}

/* Forced inheritance */
a {
  color: inherit; /* links take the parent's color */
}
```

---

### 💡 **Tips**

- Use `inherit` on elements like `<a>` if you want them to match the surrounding text color instead of default link blue.
    
- Be aware that inherited styles can cause unintended effects in deeply nested elements — check the cascade carefully.
    
- Use `unset` when you want an element to either inherit or revert to its default depending on the property.

```ad-important
Some elements like **form controls** (`input`, `textarea`, `select`, `button`, `optgroup`) don’t always inherit styles like fonts or colors the same way as normal text elements because browsers apply their own default user-agent styles.
```

---

### ✅ **Best Practices**

- Define global styles (e.g., fonts, colors) on high-level elements like `html` or `body` to leverage inheritance.
    
- Avoid overusing `inherit` on non-text properties — it can make layouts unpredictable.
    
- Use CSS variables for global values so that changes cascade consistently.
    