---
tags: html, element, fundamental
---

Including **SVG in HTML** is straightforward and very powerful for scalable, interactive graphics. Here’s how you can use SVG directly in your HTML pages:

---

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
