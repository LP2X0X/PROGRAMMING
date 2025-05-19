### **🎨 box-shadow — CSS Property Overview**

The box-shadow property adds **shadows to elements’ boxes**, which helps create depth, highlight elements (like tooltips, cards), or simulate elevation.

---

### **🔧 Syntax**

```css
box-shadow: offset-x offset-y blur-radius spread-radius color;
```

- **offset-x**: horizontal shadow (positive = right, negative = left)
    
- **offset-y**: vertical shadow (positive = down, negative = up)
    
- **blur-radius** _(optional)_: how soft the shadow is (larger = blurrier)
    
- **spread-radius** _(optional)_: how much to grow/shrink the shadow size
    
- **color**: color of the shadow (can use rgba() for transparency)
    

---

### **✅ Examples**

 **1. Basic Drop Shadow**

```css
box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
```

- Light shadow below the element.
    

#### **2. Hover Card/Tooltip**

```css
box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
```

#### **3. Inset Shadow**

```css
box-shadow: inset 0 2px 5px rgba(0, 0, 0, 0.3);
```

- Shadow goes **inside** the element.
    

#### **4. Multiple Shadows**

```css
box-shadow: 0 2px 4px rgba(0,0,0,0.2), 0 6px 20px rgba(0,0,0,0.19);
```

- Creates a soft layered effect, often used in material design.
    

---

### **💡 Best Practices**

- Use **rgba()** for soft, layered shadows (e.g., rgba(0,0,0,0.15))
    
- Combine with border-radius for clean card/tooltips
    
- Avoid overusing shadows — too many can feel heavy or messy
    
