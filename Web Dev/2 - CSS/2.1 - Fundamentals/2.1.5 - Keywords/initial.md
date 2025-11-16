---
tags: 
 - css
 - keyword
 - value
 - fundamental
---

`initial` is a CSS keyword that **resets a property back to its [[Initial Values of CSS Properties|default value]] defined by the CSS spec**, not the browser stylesheet, not your parent element.  
It’s like telling CSS: **“Forget all custom styles—use the original spec-defined default.”**

Example:

```css
button {
  all: initial;
}
```

This wipes all properties on the button back to the spec defaults.

---

## 💡 **Tips**

### **1. Use `initial` when you want predictable resets**

If you're unsure what styles have been inherited or overridden, `initial` gives you a clean baseline.

### **2. Don’t confuse `initial` with `inherit`**

- `initial`: resets to the spec’s default value
    
- `inherit`: copies the parent’s computed value
    
- `unset`:
    
    - `inherit` for inheritable properties
        
    - `initial` for non-inheritable properties
        

### **3. Great for undoing framework or UI-library styles**

Useful when you’re fighting styles from frameworks (Bootstrap, Tailwind base, etc.).

Example:

```css
input {
  all: initial;
  font: inherit;
}
```

### **4. Use sparingly — it can wipe too much**

`initial` resets **every** individual property targeted:

- May break layout
    
- Removes margins/padding/display values
    
- Removes accessibility styles (e.g., outline)
    

### **5. Prefer targeted resets**

Instead of:

```css
div {
  all: initial;
}
```

Do this:

```css
div {
  margin: initial;
  padding: initial;
}
```

This avoids nuking the entire element.

### **6. Combine with `all` for extreme cases**

```css
element {
  all: initial;
}
```

But this is **rare** outside component libraries.

---

## ⭐ **Best Practices**

- ✔ Use when building **fully isolated components** (custom buttons, form fields).
    
- ✔ Use `initial` on **specific properties** rather than globally.
    
- ✔ Combine with `inherit` to restore readability (e.g., fonts).
    
- ✔ Avoid using `initial` inside large global styles — too powerful.
    
- ❌ Don’t rely on it for layout resets; explicitly set `display`, `box-sizing`, etc.
    