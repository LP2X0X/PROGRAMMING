---
tags: css, term, fundamental
---

The **CSS Box Model** is a fundamental concept in web development that describes how elements are structured and spaced on a webpage. It consists of four main parts:

1. **Content** – The actual text, images, or other elements inside the box.

2. **Padding** – The space between the content and the border.

3. **Border** – The outline surrounding the padding and content.

4. **Margin** – The space outside the border, separating the element from others.

  

**Visual Representation:**

```
+----------------------+
|      Margin         |  (Space outside the element)
|  +--------------+  |
|  |   Border     |  |  (Surrounds padding and content)
|  |  +--------+  |  |
|  |  | Padding|  |  |  (Space around content)
|  |  | Content|  |  |  (Text, images, etc.)
|  |  +--------+  |  |
|  +--------------+  |
+----------------------+
```

**Example in CSS:**

```
.box {
  width: 200px;
  height: 100px;
  padding: 20px;
  border: 5px solid black;
  margin: 10px;
}
```

**Box-Sizing Property**

  

By default, width and height only apply to the content area. If you want padding and border to be included in the element’s total size, use:

```
.box {
  box-sizing: border-box;
}
```

This ensures the specified width includes padding and border, preventing unexpected layout issues.