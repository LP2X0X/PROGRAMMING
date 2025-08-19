---
tags:
  - html
  - element
  - fundamental
---

The \<img> element in HTML is used to **embed images** into a webpage. It is an **empty tag**, meaning it doesn’t have a closing tag, and it relies on attributes to define its content and behavior.

---

**📌 Basic Syntax:**

```html
<img src="image.jpg" alt="Description of image">
```

  

---

**🔑 Important Attributes:**

|**Attribute**|**Description**|
|---|---|
|src|**(Required)** The path to the image file. It can be a relative path, absolute path, or a URL.|
|alt|**(Required for accessibility)** A textual description of the image (shown if the image can’t be loaded, and used by screen readers).|
|width|Sets the width of the image (in pixels or %).|
|height|Sets the height of the image (in pixels or %).|
|title|Shows a tooltip when hovering over the image.|
|loading|Controls loading behavior. Values: lazy, eager. Lazy loading helps with performance.|
|srcset and sizes|Used for **responsive images**, allowing the browser to pick the best image source for different screen sizes or resolutions.|

  

---

**✅ Example:**

```html
<img src="cat.jpg" alt="A cute cat" width="300" height="200" loading="lazy">
```

  

---

**💡 Tips:**

• Always use the alt attribute for **accessibility and SEO**.

• Use loading="lazy" to improve **page load performance**.

• Use srcset for **high-DPI displays or responsive design**.