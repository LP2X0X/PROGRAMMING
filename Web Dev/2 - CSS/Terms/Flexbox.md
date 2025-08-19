---
tags:
  - css
  - term
  - fundamental
---

## **🧱 What is Flexbox?**

  

**Flexbox** (Flexible Box Layout) is a CSS layout model that arranges items in a **row or column** and makes it easy to:

- Align items
    
- Distribute space
    
- Handle dynamic sizing and spacing
    

  

Apply it by setting:

```css
display: flex;
```

on a container.

---

## **🔑 Main Concepts**

  

### **1. Flex Container**

```css
.container {
  display: flex; /* enables flexbox */
}
```

### **2. Main Axis vs Cross Axis**

- **Main Axis**: Direction items are laid out (row or column)
    
- **Cross Axis**: Perpendicular to the main axis
    

  

Set direction with:

```css 
flex-direction: row | row-reverse | column | column-reverse;
```

  

---

## **📦 Flex Properties**

  

### **On the Container (.container)**

|**Property**|**What it does**|
|---|---|
|flex-direction|Row or column layout|
|flex-wrap|Wrap items or keep in one line|
|justify-content|Align items along **main axis**|
|align-items|Align items along **cross axis**|
|align-content|Align multiple rows (when wrapping)|
|gap|Sets spacing between flex items|


### **On the Items (.item)**


|**Property**|**What it does**|
|---|---|
|flex|Shorthand for flex-grow, flex-shrink, flex-basis|
|align-self|Overrides align-items for one item|
|order|Changes the order of an item|

Example:

```
.item {
  flex: 1; /* grows to fill available space */
}
```

  

---

## **🎨 Common Layout Patterns**

  

### **Centering:**

```
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

### **Responsive Columns:**

```
.container {
  display: flex;
  flex-wrap: wrap;
}
.item {
  flex: 1 1 200px; /* grow, shrink, base width */
}
```