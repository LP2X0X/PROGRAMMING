---
tags: 
 - css
 - transition
 - overview
---

A **CSS transition** lets you smoothly change a CSS property’s value over a specified duration — instead of having it change instantly.  
They’re perfect for things like hover effects, button animations, menu reveals, etc.

---

## ✨ Basic Syntax

```css
selector {
  transition: property duration timing-function delay;
}
```

### Example:

```css
button {
  background-color: #3498db;
  color: white;
  transition: background-color 0.3s ease-in-out;
}

button:hover {
  background-color: #2ecc71;
}
```

When you hover over the button, its background color changes smoothly over **0.3 seconds**.

---

## ⚙️ Transition Properties

| Property                     | Description                   | Example                                              |
| ---------------------------- | ----------------------------- | ---------------------------------------------------- |
| `transition-property`        | Which CSS property to animate | `background-color`, `width`, `transform`, `all`      |
| `transition-duration`        | How long the transition lasts | `0.5s`, `200ms`                                      |
| `transition-timing-function` | The animation’s speed curve   | `ease`, `linear`, `ease-in-out`, `cubic-bezier(...)` |
| `transition-delay`           | Wait time before starting     | `0s`, `1s`                                           |

You can also combine them in one shorthand:

```css
transition: all 0.5s ease-in 0.2s;
```

---

## ⏱️ Timing Functions

Timing functions define **how** a transition progresses over time.

|Function|Description|
|---|---|
|`ease`|Starts slow, speeds up, slows down (default)|
|`linear`|Constant speed|
|`ease-in`|Starts slow then speeds up|
|`ease-out`|Starts fast then slows down|
|`ease-in-out`|Slow → fast → slow|
|`cubic-bezier(x1, y1, x2, y2)`|Custom curve control|

Example:

```css
div {
  transition: transform 0.5s cubic-bezier(0.68, -0.55, 0.27, 1.55);
}
```

---

## 🎨 Multiple Transitions

You can animate **multiple properties** at once by separating them with commas:

```css
div {
  transition: width 0.5s ease, height 0.5s ease, background-color 1s ease;
}
```

---

## 🔄 Common Use Cases

1. **Hover effects**
    
    ```css
    a:hover {
      color: gold;
      transform: scale(1.1);
    }
    ```
    
2. **Dropdown menus**
    
    ```css
    .menu {
      max-height: 0;
      overflow: hidden;
      transition: max-height 0.3s ease-out;
    }
    .menu.open {
      max-height: 500px;
    }
    ```
    
3. **Smooth page element reveal**
    
    ```css
    .fade {
      opacity: 0;
      transition: opacity 0.6s ease;
    }
    .fade.show {
      opacity: 1;
    }
    ```
    

---

## 🧠 Tips & Best Practices

- Use `transform` and `opacity` for **better performance** (they don’t trigger layout reflows).
    
- Avoid transitioning **layout-heavy** properties like `width`, `height`, or `top` if performance matters.
    
- Always define a **duration**, or transitions won’t occur.
    
- You can trigger transitions with **hover**, **focus**, **class toggling**, or **JavaScript** (using `element.classList.add()` etc.).
    