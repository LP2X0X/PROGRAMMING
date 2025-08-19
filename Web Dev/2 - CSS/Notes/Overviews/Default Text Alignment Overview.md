---
tags:
  - css
  - note
  - summary
---

By default, CSS aligns inline and inline-block elements based on text alignment and the **baseline** of text content. Here’s how it works:

### **1. Inline Elements (Default Behavior)**

• Elements like \<span>, \<a>, and \<img> align with the **baseline** of surrounding text.

• If no text is present, they align at the bottom of their parent container.

---

### **2. Block Elements**

• Block elements (\<div>, \<p>, etc.) take up the full width and align themselves **to the left** by default (depending on the text direction).

• Can be adjusted using text-align, margin, or flexbox/grid.

---

### **3. Inline-Block & Image Alignment**

• If you have an \<img> inside a text block, it aligns to the **baseline** by default, which can cause unwanted gaps.

• Fix it using vertical-align: middle; or display: block; on the image.

---

### **4. Text Alignment (text-align)**

• text-align: left; (default for LTR text)

• text-align: right;

• text-align: center;

• text-align: justify;

---
  
### **5. Flexbox/Grid (Modern Alignment)**

If you want better control over alignment, use display: flex; or display: grid; with justify-content and align-items.

  
