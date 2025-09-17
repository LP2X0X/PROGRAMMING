---
tags:
  - css
  - term
  - fundamental
---

### 🎯 **CSS Specificity**

CSS specificity determines **which rule is applied** when multiple rules target the same element. It is a scoring system where different types of selectors are given different weights.

---

### 📊 **Specificity Calculation Rules**

Specificity is calculated using a **four-part value** `(A, B, C, D)`:

| Selector Type                                                        | Score Contribution                |
| -------------------------------------------------------------------- | --------------------------------- |
| Inline styles                                                        | **A = 1** (Highest)               |
| IDs (`#id`)                                                          | **B = Number of IDs**             |
| Classes (`.class`), pseudo-classes (`:hover`), attributes (`[attr]`) | **C = Number of these selectors** |
| Elements (`div`, `p`, etc.) and pseudo-elements (`::before`)         | **D = Number of these selectors** |
| Universal selector (\*), Inheritance                                 | **Lowest**                                  |
| !important                                                           | **Override everything**                                  |

```ad-important
If there are multiple of the same types, then the last of them will be applied.
```

---

### 📏 **Examples of Specificity Calculation**

```css
/* (0, 0, 1, 0) - 1 Element */
h1 {
  color: blue;
}

/* (0, 1, 0, 0) - 1 ID */
#title {
  color: red;
}

/* (0, 0, 1, 1) - 1 Class + 1 Element */
h1.subtitle {
  color: green;
}

/* (1, 0, 0, 0) - Inline style (Highest) */
<h1 style="color: orange;">Hello</h1>
```

**Outcome:**

1. Inline styles (**1,0,0,0**) → Orange
2. `#title` (**0,1,0,0**) → Red (if no inline style)
3. `.subtitle` (**0,0,1,1**) → Green (if no ID or inline)
4. `h1` (**0,0,0,1**) → Blue (least specific)

---

### 🥇 **Specificity Hierarchy (Highest to Lowest)**

1. **Inline styles** → `style="color: red;"`
2. **ID selectors** → `#id`
3. **Class, attribute, and pseudo-classes** → `.class`, `[type="text"]`, `:hover`
4. **Element and pseudo-elements** → `div`, `p`, `::before`
5. **Universal selector** → `*` (No specificity value)
6. **Inherited styles** (lowest priority)

---

### 🧠 **Specificity Battles**

1. **Higher specificity wins**:
    
    ```css
    p { color: blue; }         /* (0,0,0,1) */
    #main p { color: red; }    /* (0,1,0,1) */
    ```
    
    ✅ Output: **Red** (higher specificity).
    
2. **If specificity is equal, the last rule wins**:
    
    ```css
    .btn { color: green; }
    .btn { color: orange; }
    ```
    
    ✅ Output: **Orange** (last rule applied).
    

---

### 🚀 **Tips to Manage Specificity**

1. **Avoid overusing `!important`** (overrides all).
2. **Use class selectors instead of IDs** (easier to manage).
3. **Follow the Cascade Order** – Last matching rule applies if specificity is equal.
4. **Keep it simple** – Avoid deeply nested selectors.