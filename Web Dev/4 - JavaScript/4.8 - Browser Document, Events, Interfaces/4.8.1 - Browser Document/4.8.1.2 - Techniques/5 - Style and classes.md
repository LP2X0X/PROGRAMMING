---
tags: 
 - js
 - technique
 - style
 - class
 - DOM
---

- You can style elements by **assigning CSS classes** (via HTML or JS), or by **setting inline styles** (via JS).
    
- In general — it’s better to use **CSS classes** when possible (for maintainability, reusability), and use **inline styles** when you need dynamic, script-driven styling (e.g. calculated coordinates, dynamic sizing). 
    

---

## 🧰 Working with Classes

### `className`

- `elem.className` gives or sets the full string of classes on an element.
    
- Setting `className` replaces _all_ classes. Use it if you want to overwrite the full class list. 
    

### `classList`

- `elem.classList` is a special object (a `DOMTokenList`) that lets you **add**, **remove**, **toggle**, or **check** for individual classes without disturbing others. 
    
- Methods include:
    
    - `add(className)` — add a class
        
    - `remove(className)` — remove a class
        
    - `toggle(className)` — add if absent, remove if present
        
    - `contains(className)` — check if a class exists on the element
        
- `classList` is the preferred way to manage classes dynamically because it's safe, clear, and doesn’t risk wiping out unrelated classes. 
    

---

## 🎨 Working with Inline Styles (`style`)

Each element has a `style` object corresponding to its inline `style=""` attribute:

- Style properties use **camelCase**, not CSS hyphenated names.
    
    - `background-color` → `elem.style.backgroundColor`
        
    - `border-left-width` → `elem.style.borderLeftWidth` 
        
```ad-example
background-color  => elem.style.backgroundColor
z-index           => elem.style.zIndex
border-left-width => elem.style.borderLeftWidth
-moz-border-radius => elem.style.MozBorderRadius
-webkit-border-radius => elem.style.WebkitBorderRadius
```

- For browser-specific (prefixed) CSS properties, same rule applies: hyphens become uppercase. 

### Setting and Resetting

- To set a style:
    
    ```js
    elem.style.width = "100px";
    ```
    
- To remove a previously set style (so that CSS classes or default styles apply again), set the property to an empty string:
    
    ```js
    elem.style.display = "";
    ```
    
    Or use `elem.style.removeProperty(...)`. 
    

### Setting many styles at once

- Instead of setting each property individually, you can set the full style string via `style.cssText`. This overwrites all previous inline styles. Use cautiously (may remove styles you wanted to keep).
    

### Units matter

- When setting properties that require units (e.g. width, margin, top, left, etc.), you must provide units — e.g. `"20px"`, not just `20`. Otherwise browsers ignore the assignment. 
    

---

## 🔍 Reading “Computed” Styles: `getComputedStyle`

- `elem.style` only reflects inline styles. It does _not_ show styles coming from external CSS or CSS classes. 
    
- To get the actual final styles applied to an element (after CSS cascade, inheritance, external stylesheets, etc.), use:
    
    ```js
    const cs = getComputedStyle(elem);
    ```
    
    `cs` gives you all resolved style values (e.g. actual width, margin, padding) — readonly.
    
- Use the exact property names (like `paddingLeft`, `marginTop`, etc.) when accessing computed values. 
    

---

## 🧑‍💻 When to Use What: Practical Guidelines

- ✅ Use **CSS classes + `classList`** when you are toggling or switching styles (theme, mode, layout changes, etc.). More maintainable.
    
- ✅ Use **inline styles** when you need **dynamic styling** — e.g. position based on runtime calculations, animation offsets, direct adjustments.
    
- ✅ If you need to **read actual current style** (regardless of inline or class-based), use `getComputedStyle`.
    
- ✅ If you must override many inline styles at once and you’re sure you don’t need previous inline styles — `style.cssText` is an option (but be careful).
    

---

## Summary

To manage classes, there are two DOM properties:

- `className` – the string value, good to manage the whole set of classes.
- `classList` – the object with methods `add/remove/toggle/contains`, good for individual classes.

To change the styles:

- The `style` property is an object with camelCased styles. Reading and writing to it has the same meaning as modifying individual properties in the `"style"` attribute. To see how to apply `important` and other rare stuff – there’s a list of methods at [MDN](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration).
    
- The `style.cssText` property corresponds to the whole `"style"` attribute, the full string of styles.
    

To read the resolved styles (with respect to all classes, after all CSS is applied and final values are calculated):

- The `getComputedStyle(elem, [pseudo])` returns the style-like object with them. Read-only.