The pointer-events CSS property controls how HTML elements respond to **mouse, touch, and pen interactions**. It’s especially useful when you want to:

- Disable or ignore clicks on an element.
    
- Let clicks “pass through” an element to whatever is behind it.
    

---

### **✅ Common Values of pointer-events**

|**Value**|**Description**|
|---|---|
|auto|Default. The element reacts to pointer events normally.|
|none|The element **does not receive** pointer events — clicks pass through it.|

---

### **🧪 Example: Making an Element Unclickable**

```html
<div class="click-blocker">I won’t block clicks</div>
<div class="under">I’m clickable underneath</div>
```

```css
.click-blocker {
  position: absolute;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  pointer-events: none; /* This line makes it transparent to clicks */
}
```

---

### **🧠 Use Cases**

- **Tooltips or overlays** that shouldn’t block mouse interaction.
    
- **Custom cursors or highlights** that float above the page but allow interaction underneath.
    
- **Disabling interactions** temporarily (e.g., during loading).
    

---

### **⚠️ Tip**

  
Setting pointer-events: none also disables **hover**, **:active**, and **:focus** styles or events on that element.
