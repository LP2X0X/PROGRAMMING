### **📱 CSS Media Queries — Overview & Usage**

**Media queries** let you apply different CSS styles depending on the **screen size**, **device type**, or **other characteristics** like resolution.

---

### **🔧 Basic Syntax**

```css
@media (condition) {
  /* CSS rules that apply only if condition is true */
}
```

---

### **📐 Common Use: Responsive Layouts**
  

#### **✅ Example: Adjust layout for mobile screens**

```css
/* Base styles (for desktop or default) */
.video-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}

/* Media query for tablets and smaller */
@media (max-width: 900px) {
  .video-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Media query for phones */
@media (max-width: 600px) {
  .video-grid {
    grid-template-columns: 1fr;
  }
}
```

---

### **🧠 How It Works**

- max-width: 900px → applies styles **when screen width is 900px or less**
    
- You can also use:
    
    - min-width (e.g. min-width: 768px)
        
    - orientation: landscape
        
    - hover: none or pointer: coarse (for touch devices)
        
    

---

### **📋 Common Breakpoints**

|**Device**|**Width Range**|
|---|---|
|Mobile|≤ 600px|
|Tablet|601px – 900px|
|Small Desktop|901px – 1200px|
|Desktop|> 1200px|

---

### **✅ Real Example for Your Project**

  

To make your .video-grid responsive, try:

```css
.video-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 16px;
  padding: 16px;
}
```

You may not even need media queries if you use auto-fill + minmax(), since it adapts automatically.

  

But you can always fine-tune:

```css
@media (max-width: 768px) {
  .video-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 480px) {
  .video-grid {
    grid-template-columns: 1fr;
  }
}
```

---

## **🧩 Common Media Query Conditions (Media Features)**

### **📏 Screen Size**

|**Feature**|**Example**|**Description**|
|---|---|---|
|max-width|@media (max-width: 600px)|Applies when the screen is **≤ 600px** wide|
|min-width|@media (min-width: 768px)|Applies when the screen is **≥ 768px** wide|
|width|@media (width: 1024px)|Applies **only** at exactly 1024px width|
|height|@media (max-height: 500px)|Applies to short/tall screens|

---

### **📱 Device Type (Media Type)**

|**Type**|**Example**|**Use case**|
|---|---|---|
|screen|@media screen and (max-width: 600px)|Normal devices with screens|
|print|@media print|When printing the page|
|all|@media all and (min-width: 1024px)|All devices|

---

### **🧭 Orientation**

|**Feature**|**Example**|**Description**|
|---|---|---|
|orientation|@media (orientation: portrait)|Phones in vertical mode|
||@media (orientation: landscape)|Phones in horizontal mode|

---

### **🖱️ Pointer and Hover**

|**Feature**|**Example**|**Description**|
|---|---|---|
|hover|@media (hover: none)|For touch screens (can’t hover)|
|pointer|@media (pointer: coarse)|Coarse pointer (e.g. finger on touchscreens)|
|any-hover|@media (any-hover: hover)|At least one input supports hover|

---

### **👓 Resolution**

```css
@media (min-resolution: 2dppx) { ... }  /* high-res (retina) displays */
@media (max-resolution: 300dpi) { ... } /* low-res (printers or old screens) */
```

---

### **🧪 Example: Combine Multiple Conditions**

```css
@media screen and (max-width: 768px) and (orientation: portrait) {
  .video-grid {
    grid-template-columns: 1fr;
  }
}
```

---

### **💡 Tip: Logical Operators**

  
You can combine conditions:

- and: @media (min-width: 600px) and (max-width: 1200px)
    
- not: @media not screen and (min-width: 768px)
    
- only: @media only screen and (max-width: 800px)
    
