---
tags: 
 - css
 - property
 - text
 - fundamental
---

## 📖 Definition

`line-height` controls the amount of vertical space between **baselines of lines of text**.  
It affects the “leading” (line spacing), not the font size itself.

---

## 🖋️ Syntax

```css
selector {
  line-height: normal | <number> | <length> | <percentage>;
}
```

### Values:

1. **`normal`**
    
    - Default (roughly `1.2` × font size, but depends on the browser and font).
        
2. **Number (unitless, best practice)**
    
    ```css
    line-height: 1.5;
    ```
    
    - Multiplies the element’s font size.
        
    - Inherits better because it scales proportionally with the child’s font size.
        
3. **Length**
    
    ```css
    line-height: 24px;
    ```
    
    - Fixed line box height regardless of font size.
        
4. **Percentage**
    
    ```css
    line-height: 150%;
    ```
    
    - Based on the element’s font size (e.g., `1.5 × font-size`).
        

---

## 🔍 Example

```css
p {
  font-size: 16px;
  line-height: 1.5;   /* recommended */
}
```

- Line height = `16px × 1.5 = 24px`.
    

---

## 🛠️ Tips & Best Practices

- ✅ Use **unitless numbers** (e.g., `line-height: 1.5;`) → more consistent across nested elements.
    
- ✅ `line-height: normal;` is okay for small text, but too tight for body copy.
    
- 📖 Common values:
    
    - Headlines: `1.1 – 1.3`
        
    - Paragraphs: `1.4 – 1.6`
        
- ❌ Avoid using fixed `px` values unless you need pixel-perfect control.
    