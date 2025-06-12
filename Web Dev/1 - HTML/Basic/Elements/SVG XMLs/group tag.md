---
tags: html, tag, fundamental
---

The \<g> tag in **SVG XML** is used to **group multiple elements** together so you can apply transformations, styles, or interactions to them **as a unit**.

---

## **🔹 Basic Syntax**

```html
<g>
  <!-- grouped elements go here -->
</g>
```

You can apply attributes (like transform, fill, stroke, opacity, etc.) to the \<g> tag, and they will affect all child elements inside the group.

---

## **🔹 Example: Grouping with \<g>** 

```html
<svg width="300" height="150" xmlns="http://www.w3.org/2000/svg">
  <g fill="blue" transform="translate(50, 50)">
    <rect x="0" y="0" width="50" height="50" />
    <circle cx="100" cy="25" r="25" />
    <text x="0" y="80" font-size="16">Grouped!</text>
  </g>
</svg>
```

### **What This Does:**

- Groups a rect, circle, and text together
- Applies fill="blue" to all
- Moves the entire group 50px right and 50px down using transform="translate(50, 50)"

---

## **Transformations with \<g>**

Creating groupings with g elements also lets you apply transformations to all the child elements in a group.

SVG supports several kinds of transformations, including translation, rotation, scaling, and skewing.

```html
<g transform="translate(100, 50)" scale(2, 3) font-family="sans-serif" fill="blue">
```

---

## **🔹 Why Use \<g>?**

|**Use Case**|**How** \<g> **Helps**|
|---|---|
|Grouping shapes into one unit|Easier to move/scale all at once|
|Apply shared styles|Set one fill, stroke, etc.|
|Interactivity|Add a single onclick to a group|
|Nesting structure (like layers)|Organize complex drawings|

---

## **🔸 Example with Event**

```html
<svg width="200" height="100" xmlns="http://www.w3.org/2000/svg">
  <g onclick="alert('Group clicked!')" cursor="pointer">
    <circle cx="50" cy="50" r="30" fill="orange" />
    <text x="50" y="55" text-anchor="middle" fill="black">Click Me</text>
  </g>
</svg>
```

---

## **✅ Summary**

- \<g> groups multiple SVG elements
- You can apply styles, transformations, and events to the group
- Great for organizing and controlling related shapes as one