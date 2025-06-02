---
tags: css, property, fundamental
---

### **🔧 What is flex-shrink?**

flex-shrink defines how much a **flex item** is allowed to **shrink** when there isn’t enough space in the container.

---

### **📐 Syntax**

```css
.item {
  flex-shrink: <number>; /* Default: 1 */
}
```

- \<number> is a **unitless ratio**
    
- A value of 0 means **“don’t shrink at all”**
    
- A value of 1 (default) means **“shrink if needed”**
    
- Higher values mean the item **shrinks more** compared to others
    

---

### **🧠 How it works**

When the total width of flex items **exceeds the container**, Flexbox tries to shrink them.

- If all items have flex-shrink: 1, they shrink **equally** (based on size)
    
- If one item has flex-shrink: 0, it won’t shrink at all
    
- If one item has a higher shrink factor, it shrinks **more aggressively**
    

---

### **💡 Common Use Cases**

|**Use Case**|**What to do**|
|---|---|
|Keep element fixed size|flex-shrink: 0|
|Allow flexible layout|flex-shrink: 1 (default)|
|Shrink one item more than others|Use higher flex-shrink (e.g. 2)|

---

### **🧪 Example**

```css
.container {
  display: flex;
  width: 300px;
}

.item1 {
  width: 200px;
  flex-shrink: 0;
}

.item2 {
  width: 200px;
  flex-shrink: 1;
}
```

- Total width = 400px > 300px container
    
- item1 stays at 200px
    
- item2 shrinks by 100px to fit
    

---

### **✅ Best Practice**

When you want an element like an icon or profile picture to **never shrink**, use:

```
flex-shrink: 0;
```