---
tags: css, property, fundamental
---

### 🔹 **Definition**  
The `width` property in CSS defines the horizontal size of an element’s content area. It controls how wide the element appears in the layout, without including padding, borders, or margins (unless `box-sizing` changes that behavior).

---

### 🔹 **Possible Values**

1. **Length units** — fixed size
    
    - Examples: `px`, `em`, `rem`, `cm`, etc.
        
    - Example:
        
        ```css
        div { width: 300px; }
        ```
        
2. **Percentage (%)** — relative to the parent element’s content width
    
    - Example:
        
        ```css
        div { width: 50%; }
        ```
        
3. **Viewport units** — relative to viewport size
    
    - `vw` (viewport width) → 1vw = 1% of viewport width
        
    - Example:
        
        ```css
        div { width: 80vw; }
        ```
        
4. **Keywords** — special behaviors
    
    - `auto` → default; lets the browser decide based on content or layout rules
        
    - `max-content` → fits content without wrapping
        
    - `min-content` → smallest possible width without overflowing
        
    - `fit-content` → clamps the size between min and max content sizes
        

---

### 🔹 **`box-sizing` Interaction**

- By default, `width` applies only to the **content box**. Padding and borders are added on top.
    
- Using [[box-sizing|box-sizing: border-box;]] makes `width` include **content + padding + border**.
    
- Example:
    
    ```css
    div {
      width: 300px;
      box-sizing: border-box;
    }
    ```
    

---

### 🔹 **Min & Max Width**  
You can restrict the range of width:

```css
div {
  min-width: 200px;
  max-width: 800px;
}
```

- Useful for making layouts responsive but preventing elements from becoming too small or too large.
    

---

### 🔹 **Default Behavior**

- Block-level elements (e.g., `<div>`) default to `width: auto` and expand to fill the container.
    
- Inline elements (e.g., `<span>`) ignore `width` unless changed to `display: inline-block`, `block`, or `flex`.
    