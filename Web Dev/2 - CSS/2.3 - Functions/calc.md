---
tags: 
 - css
 - function
---

`calc()` lets you **combine different units** (px, %, vw, vh, rem, etc.) using math inside CSS.

---

# **What it can do**

- Add
    
- Subtract
    
- Multiply
    
- Divide
    

Example:

```css
width: calc(100% - 2rem);
```

---

# **Why it's useful**

Because CSS normally can’t mix units directly.  
`calc()` solves layouts like:

- Elements with percent width **minus padding/margins**
    
- Full-height sections:  
    `height: calc(100vh - 80px);`
    
- Responsive gaps
    
- Centering tricks
    

---

# **Rules**

- **Spaces are required** around operators  
    ✔️ `calc(100% - 20px)`  
    ✖️ `calc(100%-20px)`
    
- Works in any property that accepts numeric values
    
- Can mix any unit
    

---

# **Common patterns**

### **1. Fixed header + full height**

```css
main {
  height: calc(100vh - 64px);
}
```

### **2. Element beside sidebar**

```css
.content {
  width: calc(100% - 240px);
}
```

### **3. Centering inside unknown container**

```css
padding-left: calc(50% - 200px);
```

### **4. Fluid typography**

```css
font-size: calc(1rem + 1vw);
```

---

# **When to use**

- When normal CSS cannot express a width or height using a single unit
    
- When layout is partly fixed, partly fluid
    
- When exact spacing matters
    