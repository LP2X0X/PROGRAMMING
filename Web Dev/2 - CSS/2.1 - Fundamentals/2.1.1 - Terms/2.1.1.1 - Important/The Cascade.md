---
tags: 
 - css
 - term
 - important
---

When multiple CSS declarations could apply to the same element, the browser needs a way to **decide which one wins**. This decision-making process is the **cascade**, and it evaluates six criteria in a specific order:

1. **Stylesheet origin**—Where the styles come from. Your styles are applied in conjunction with the browser’s default styles.
2. **Inline styles**—Whether a declaration is applied to an element via the HTML style attribute or a CSS selector.
3. **Layer**—Styles can be defined in layers, each with a different priority.
4. **Selector specificity**—Which selectors take precedence over which.
5. **Scope proximity**—Whether the styles are scoped to a portion of the DOM.
6. **Source order**—Order in which styles are declared in the stylesheet.

![[Pasted image 20251002213329.png|center|600]]
 <figcaption align="center"><strong>Figure:</strong> High-level flowchart of the cascade showing declaration precedence</figcaption>

---

### 1. **Stylesheet Origin**

- CSS can come from different sources:
    
    - **User agent styles** (the browser’s default stylesheet). Example: `<p>` is `display: block` by default.
        
    - **User styles** (custom styles a user might define in their browser settings, often for accessibility).
        
    - **Author styles** (the CSS written by the web developer, usually in your stylesheets).
        
- Priority order:
    
    - User `!important`
        
    - Author `!important`
        
    - Author normal
        
    - User normal
        
    - Browser default
        

---

### 2. **Inline Styles**

- If a style is applied directly on an element via the `style` attribute, it generally overrides styles from a stylesheet (unless there are `!important` rules in play).
    
- Example:
    
    ```html
    <p style="color: red;">Hello</p>
    ```
    
    → The text will be red, even if your stylesheet says `p { color: blue; }`.
    

---

### 3. **Layer**

- Introduced in **Cascade Layers (CSS @layer)**.
    
- Layers allow you to group styles and give some groups higher priority.
    
- Example:
    
    ```css
    @layer reset, base, theme, components;
    
    @layer reset {
      * { margin: 0; padding: 0; }
    }
    
    @layer theme {
      p { color: blue; }
    }
    
    @layer components {
      p { color: red; }
    }
    ```
    
    → `components` layer styles will win over `theme`, which wins over `reset`.
    

---

### 4. **Selector Specificity**

- Not all selectors are equal—some are more “specific”:
    
    - Inline styles → specificity weight **1000**
        
    - ID selectors (`#id`) → **100**
        
    - Class, attribute, pseudo-class (`.class`, `[attr]`, `:hover`) → **10**
        
    - Type selectors (`div`, `p`, `h1`) and pseudo-elements (`::before`) → **1**
        
- The higher the total, the stronger the rule.
    
- Example:
    
    ```css
    p { color: blue; }             /* 1 */
    .intro p { color: red; }       /* 11 */
    #main .intro p { color: green; } /* 111 */
    ```
    
    → A `<p>` in `#main .intro` will be green.
    

---

### 5. **Scope Proximity**

- With new **scoped styles** (e.g. `@scope` in CSS, or frameworks like Vue/React inline scoping), the **closer a style is scoped to an element, the more priority it has**.
    
- Example:
    
    ```css
    @scope (.card) {
      p { color: blue; }
    }
    
    p { color: red; }
    ```
    
    → A `<p>` inside `.card` will be blue, even though a global `p {}` exists.
    

---

### 6. **Source Order**

- If everything else is equal (same origin, layer, specificity, etc.), the browser uses the **last declared rule** in the stylesheet.
    
- Example:
    
    ```css
    p { color: blue; }
    p { color: red; }
    ```
    
    → The `<p>` will be red, since that rule comes last.
    

---

## ✅ Quick Recap

When resolving conflicts, CSS checks:

1. **Origin** → Which stylesheet (browser, user, author)?
    
2. **Inline** → Does it come directly from the element’s `style` attribute?
    
3. **Layer** → Which cascade layer has priority?
    
4. **Specificity** → How strong is the selector?
    
5. **Scope proximity** → Is the style scoped close to the element?
    
6. **Source order** → If all else is equal, the last one wins.
    
