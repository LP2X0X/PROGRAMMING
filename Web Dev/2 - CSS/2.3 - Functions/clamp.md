---
tags: 
 - css
 - function
---

# **1. What `clamp()` Does**

`clamp()` lets you define a value with:

```
clamp(min, preferred, max)
```

The browser chooses a value that:

- **Never goes below** the minimum
    
- **Never exceeds** the maximum
    
- **Uses the preferred value** if it's within the range
    

It is commonly used for:

- Responsive font sizes
    
- Responsive spacing
    
- Fluid layout constraints
    

---

# **2. Syntax**

```css
font-size: clamp(1rem, 2vw + 0.5rem, 2rem);
```

Meaning:

- Minimum: 1rem
    
- Preferred (fluid): 2vw + 0.5rem
    
- Maximum: 2rem
    

---

# **3. How It Behaves**

If the “preferred” middle value is smaller than the minimum, the final value becomes **the minimum**.

If the preferred value is larger than the maximum, the final value becomes **the maximum**.

If it is in between, the preferred value is used.

---

# **4. Common Use Cases**

## A. **Fluid Typography**

```css
:root {
  --font-fluid: clamp(1rem, 1.2vw + 0.5rem, 2rem);
}

h1 {
  font-size: var(--font-fluid);
}
```

This creates typography that grows with screen size but within limits.

---

## B. **Responsive Container Width**

```css
.container {
  width: clamp(320px, 80%, 1200px);
}
```

Meaning:

- Never shrink below 320px
    
- Ideally use 80% width
    
- Never exceed 1200px
    

---

## C. **Spacing / Padding**

```css
section {
  padding: clamp(1rem, 3vw, 3rem);
}
```

---

# **5. Why `clamp()` is Better Than Media Queries**

Without clamp:

```css
h1 {
  font-size: 1rem;
}

@media (min-width: 600px) {
  h1 {
    font-size: 2rem;
  }
}
```

This has a **hard jump** between sizes.

With clamp:

```css
h1 {
  font-size: clamp(1rem, 4vw, 2rem);
}
```

This is **smooth and continuous** without media queries.

---

# **6. Typography Helper Version (Copy/Paste)**

```css
:root {
  --step--2: clamp(0.56rem, 0.48rem + 0.3vw, 0.64rem);
  --step--1: clamp(0.75rem, 0.70rem + 0.25vw, 0.90rem);
  --step-0: clamp(1rem, 0.95rem + 0.35vw, 1.25rem);
  --step-1: clamp(1.33rem, 1.20rem + 0.5vw, 1.67rem);
  --step-2: clamp(1.77rem, 1.60rem + 0.8vw, 2.35rem);
  --step-3: clamp(2.37rem, 2.20rem + 1vw, 3.03rem);
  --step-4: clamp(3.16rem, 2.90rem + 1.5vw, 4.05rem);
}
```

---

# **7. Summary**

- `clamp()` picks a value between min → preferred → max.
    
- Commonly used for fluid fonts and responsive layout.
    
- Eliminates many media queries.
    
- Supports arithmetic like `2vw + 0.5rem`.
    
- Works in all modern browsers.
    