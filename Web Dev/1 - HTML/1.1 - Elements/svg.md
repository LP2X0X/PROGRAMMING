---
tags: 
 - html
 - element
 - fundamental
---

# 🖼️ The `<svg>` Element in HTML

SVG (**Scalable Vector Graphics**) is an XML-based format for creating **vector graphics** directly in HTML. Unlike raster images (PNG, JPG), SVGs scale without losing quality.

An `<svg>` element defines an **SVG container**, inside which you can draw shapes, paths, text, and more.

---

## 🔹 Common Attributes of `<svg>`

|Attribute|Purpose|Example|
|---|---|---|
|`xmlns`|XML namespace, required when embedding inline SVGs.|`xmlns="http://www.w3.org/2000/svg"`|
|`width` / `height`|Sets the size of the SVG on the page.|`width="100" height="100"`|
|`viewBox`|Defines the internal coordinate system: `min-x min-y width height`.|`viewBox="0 0 24 24"`|
|`fill`|Default **fill color** for child shapes.|`fill="red"`|
|`stroke`|Default **stroke color** (outline).|`stroke="black"`|
|`stroke-width`|Width of the stroke.|`stroke-width="2"`|
|`class`|Lets you style via CSS.|`class="icon"`|
|`style`|Inline CSS styles.|`style="display:block;"`|

---

## 🔹 Common Child Elements Inside `<svg>`

|Element|Purpose|
|---|---|
|`<path>`|Defines a shape using a `d` attribute (move, line, curve commands).|
|`<circle>`|Draws a circle (`cx`, `cy`, `r`).|
|`<rect>`|Draws a rectangle (`x`, `y`, `width`, `height`, `rx`, `ry` for rounded corners).|
|`<line>`|Draws a straight line (`x1`, `y1`, `x2`, `y2`).|
|`<polygon>`|Draws a closed shape with multiple points.|
|`<polyline>`|Similar to polygon but open-ended.|
|`<text>`|Renders text.|

---

## 🔹 Example SVG in HTML

```html
<svg xmlns="http://www.w3.org/2000/svg" 
     width="100" height="100" 
     viewBox="0 0 24 24" 
     fill="none" stroke="black" stroke-width="2">
  
  <!-- Circle -->
  <circle cx="12" cy="12" r="10" />

  <!-- Line -->
  <line x1="12" y1="6" x2="12" y2="18" />

  <!-- Path -->
  <path d="M6 12h12" />

</svg>
```

🔎 This draws:

- A circle at the center
    
- A vertical line
    
- A horizontal line  
    Together, they form a “plus” inside a circle.
    

---

## 🔹 Why `viewBox` is important

`viewBox="0 0 24 24"` means:

- Top-left corner = (0, 0)
    
- Width = 24 units
    
- Height = 24 units
    

If you scale the SVG (e.g., width="100px"), the browser maps the **100px box** onto this **24×24 coordinate system**.

---

✅ In short:

- `<svg>` is the container.
    
- Attributes like `viewBox`, `fill`, and `stroke` define **defaults and scaling**.
    
- Child elements (`<path>`, `<circle>`, `<rect>`, etc.) are the actual drawing instructions.
    

----

## **🔹 1. Inline SVG (Recommended for Control & Styling)**

You write the SVG code directly inside your HTML file:

```html
<svg width="200" height="100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="40" fill="skyblue" stroke="black" stroke-width="2" />
  <text x="50" y="55" font-size="16" text-anchor="middle" fill="black">Hello</text>
</svg>
```

✅ Can be styled with CSS
✅ Can be animated or changed with JavaScript
✅ Supports interaction (e.g., onclick)

---

## **🔹 2. SVG via \<img> Tag (Good for Static Images)**

```
<img src="icon.svg" width="100" height="100" />
```

✅ Easy to use
❌ Cannot be styled or interacted with (it’s just an image)

---

## **🔹 3. SVG as Background (in CSS)**

```
div {
  background-image: url("image.svg");
  width: 100px;
  height: 100px;
}
```

✅ Good for decorative purposes
❌ No interactivity or JavaScript access

---

## **🔹 4. Embed SVG with \<object> or \<iframe>**

```html
<object type="image/svg+xml" data="file.svg" width="300" height="200"></object>
```

✅ Keeps SVG separate
❌ Harder to manipulate from the main HTML page

---

## **🔸 Tip: Style Inline SVG with CSS**

```html
<style>
  circle {
    transition: fill 0.3s;
  }

  circle:hover {
    fill: orange;
  }
</style>
```

---

## **✅ Best Use Cases**

|**Goal**|**Best Method**|
|---|---|
|Interactive graphics|Inline SVG|
|Icons/logos|\<img src="...">|
|Data visualizations|Inline + JS (or D3.js)|
|Reusable UI assets|\<symbol> + \<use>|