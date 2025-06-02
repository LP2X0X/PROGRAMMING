---
tags: css, property, fundamental
---

### **🧾 white-space CSS Property — Overview**

The white-space property controls **how whitespace inside an element is handled**, including:

- **Spaces**
    
- **Tabs**
    
- **Line breaks**
    

---

### **🔧 Syntax**

```css
white-space: normal | nowrap | pre | pre-wrap | pre-line;
```

---

### **📚 Common Values**

|**Value**|**Behavior**|
|---|---|
|normal|Collapses whitespace. Lines break as needed. (default)|
|nowrap|Collapses whitespace. **Prevents line breaks** (text stays on one line).|
|pre|Preserves whitespace. **No line wrapping**, line breaks follow \n.|
|pre-wrap|Preserves whitespace **and** wraps text to next line if needed.|
|pre-line|Collapses whitespace, but **respects line breaks** (\n).|

---

### **🔍 Examples**

#### **✅ white-space: normal**

```css
p {
  white-space: normal;
}
```

- Text wraps automatically
    
- Extra spaces/tabs are collapsed
    

#### **✅ white-space: nowrap**

```css
.button-label {
  white-space: nowrap;
}
```

- Keeps text on one line
    
- Useful for preventing buttons from expanding in height due to wrapping
    

#### **✅ white-space: pre-wrap**

```css
.code-block {
  white-space: pre-wrap;
}
```

- Maintains spacing and newlines
    
- Still wraps if it reaches container edge
    

---

### **💡 Use Cases**

- nowrap: Prevent label or UI text from wrapping
    
- pre: Show literal text (like code or ASCII art)
    
- pre-wrap: Chat bubbles, code blocks, or markdown rendering
    
- normal: Default for most text
    