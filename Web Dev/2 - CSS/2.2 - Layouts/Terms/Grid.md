---
tags:
  - css
  - term
  - layout
  - fundamental
---

### **🧱 CSS Grid Overview**

**CSS Grid** is a powerful layout system that lets you design two-dimensional layouts — controlling both **rows** and **columns** with precision.

![[Pasted image 20250824203940.png|center|500]]

---

## **🔧 Basic Setup**

```css
.container {
  display: grid;
  grid-template-columns: 1fr 2fr;
  grid-template-rows: auto;
  gap: 1rem;
}
```

```html
<div class="container">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>
```

- 1fr 2fr → Two columns: 1/3 and 2/3 width
    
- gap → spacing between grid cells
    

---

## **📐 Common Properties**
### **🔹 On the Grid Container:**

|**Property**|**Description**|
|---|---|
|display: grid|Enables grid layout|
|grid-template-columns|Defines column sizes (e.g., 1fr, 200px, auto)|
|grid-template-rows|Defines row sizes|
|gap|Sets spacing between rows and columns|
|grid-row-gap|Set spacing between rows|
|grid-column-gap|Set spacing between columns|
|justify-items|Aligns content horizontally in cells|
|align-items|Aligns content vertically in cells|
|place-items|Shorthand for align-items + justify-items|

### **🔹 On the Grid Items:**

|**Property**|**Description**|
|---|---|
|grid-column|Start/end column lines (e.g., 1 / 3)|
|grid-row|Start/end row lines|
|grid-area|Name or placement of item|
|justify-self|Horizontal alignment inside the cell|
|align-self|Vertical alignment inside the cell|
|place-self|Shorthand for the above two|

---

## **✨ Auto Layout Example**

```css
.container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
}
```

- Automatically fits as many columns as possible
    
- Each column is at least 200px, and can grow
    

---

## **🧩 Named Areas Example**

```css
.container {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar content";
  grid-template-columns: 200px 1fr;
  grid-template-rows: auto 1fr;
}
```

```html
<div class="container">
  <div style="grid-area: header;">Header</div>
  <div style="grid-area: sidebar;">Sidebar</div>
  <div style="grid-area: content;">Main</div>
</div>
```
