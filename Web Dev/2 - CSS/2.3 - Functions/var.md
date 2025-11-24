---
tags: 
 - css
 - function
---

`var()` is the function used to **read CSS custom properties (variables)**.  
Think of it as **“insert the value of this variable here.”**

---

## ✅ **1. What `var()` Actually Does**

`var(--name)` **retrieves the value** of a custom property that starts with `--`.

Example:

```css
:root {
  --brand-color: #4f46e5;
}

button {
  background: var(--brand-color);
}
```

Custom properties:

- are **dynamic** (can be changed at runtime using classes, media queries, JS…)
    
- inherit like normal CSS properties
    
- can be used in _any_ property (color, widths, transforms, animations, etc.)
    

---

## ✅ **2. Syntax**

```css
var(--custom-property, fallback)
```

- **first argument**: the variable name
    
- **second argument (optional)**: the fallback value if the variable is missing
    

Example:

```css
color: var(--text, black);
```

If `--text` is _not_ defined, it uses **black**.

⚠️ Fallback is used ONLY if the entire variable is missing.  
If the variable is defined but invalid → fallback is ignored.

---

## ✅ **3. Why `var()` is powerful (in detail)**

### ### 🌟 **A. They cascade**

Variables follow the CSS cascade:

```css
.card {
  --spacing: 1rem;
  padding: var(--spacing);
}

.card.large {
  --spacing: 2rem; /* overrides */
}
```

Inheritance means child elements also get the variable unless overwritten.

---

### 🌟 **B. They work inside other functions**

You can put `var()` basically anywhere:

```css
width: calc(100% - var(--sidebar-width));
transform: translateX(var(--offset));
box-shadow: 0 0 0 var(--outline-width) blue;
```

---

### 🌟 **C. They can store ANYTHING**

Unlike Sass variables, CSS variables are not typed.

You can store:

```css
--duration: 300ms;
--shadow: 0 2px 8px rgba(0,0,0,.2);
--grid-template: 1fr auto 2fr;
--scale: scale(1.2);
--random-json: "hello";
```

CSS doesn't care — it just injects the text.

---

### 🌟 **D. They update instantly without recompiling**

Change a class → all variables re-evaluate in real time:

```css
.dark {
  --bg: black;
  --text: white;
}
```

Toggle `.dark` → UI updates instantly.

This is how Tailwind’s theming and most dark modes work internally.

---

### 🌟 **E. They work with media queries**

Responsive design + variables = extremely powerful:

```css
:root {
  --padding: 1rem;
}

@media (min-width: 900px) {
  :root {
    --padding: 2rem;
  }
}

section {
  padding: var(--padding);
}
```

---

### 🌟 **F. They can be animated**

You CANNOT animate variables directly,  
but **properties that use them** will animate smoothly:

```css
:root {
  --blur: 0px;
}

img {
  filter: blur(var(--blur));
  transition: filter .3s;
}

img:hover {
  --blur: 5px;
}
```

---

## ✅ **4. The Fallback Argument — Detailed View**

Fallback value is used ONLY if the variable is not defined:

```css
width: var(--w, 100px);
```

Cases:

|Variable state|Result|
|---|---|
|variable missing|fallback used|
|variable defined but empty|invalid → fallback **NOT** used|
|variable defined but invalid|invalid → fallback **NOT** used|

Example of invalid:

```css
--w: ;
width: var(--w, 100px); /* invalid, fallback ignored */
```

Why?  
CSS sees variable exists → tries to use it → it's invalid → but fallback is only for undefined, not invalid.

---

## ✅ **5. Where to Define Variables**

### **A. Global**

```css
:root {
  --brand: #2563eb;
}
```

### **B. Component scoped**

```css
.card {
  --radius: 12px;
  border-radius: var(--radius);
}
```

### **C. State-based**

```css
.card:hover {
  --radius: 20px;
}
```

---

## ✅ **6. Common Patterns**

### 💡 **Dynamic themes**

```css
.light { --bg: white; --text: black; }
.dark  { --bg: black; --text: white; }
```

---

### 💡 **Spacing scale**

```css
:root {
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 16px;
}
```

---

### 💡 **Store parts of calc operations**

```css
--header: 64px;
main { height: calc(100vh - var(--header)); }
```

---

## ✅ **7. Limitations**

- Cannot be used in media queries (e.g., `@media (min-width: var(--bp))` ❌)
    
- Invalid values break the whole property
    
- Cannot store selectors
    
- Cannot do logic (if/else)
    

---

# ✔️ Summary

`var()`:

- retrieves CSS custom properties
    
- works everywhere
    
- cascades and inherits
    
- supports fallback
    
- enables themes, responsive design, dynamic calculations, and component-level styling
    
- extremely powerful because variables update **live** without recompiling
    
