---
tags:
  - css
  - term
  - layout
  - fundamental
---

## **🧱 What is Flexbox?**
  

**Flexbox** (Flexible Box Layout) is a CSS *layout model* that arranges items in a **row or column** and makes it easy to:

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

### **2. Flex Items**
- Children elements inside the flex container element.

### **2. Main Axis vs Cross Axis**

- **Main Axis**: Direction items are laid out (row or column)
    
- **Cross Axis**: Perpendicular to the main axis
    

Set direction with:

```css 
flex-direction: row | row-reverse | column | column-reverse;
```

  ![[Pasted image 20250824185355.png|center|600]]

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

```css
.item {
  flex: 1; /* grows to fill available space */
}
```

  

---

## **🎨 Common Layout Patterns**
### **Centering:**

```css
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

### **Responsive Columns:**

```css
.container {
  display: flex;
  flex-wrap: wrap;
}
.item {
  flex: 1 1 200px; /* grow, shrink, base width */
}
```

---

## **🛠️ Tips & Best Practices**

- Prefer **Flexbox for one-dimensional layouts**, Grid for two-dimensional (rows + columns).
    
- Use gap instead of margin for consistent spacing between flex items.
    
- Avoid relying heavily on order → can confuse screen readers and tab navigation.
    
- Combine flex-wrap: wrap; + flex-basis for responsive grids.
    
- Use min-width on items to prevent shrinking too small.
    