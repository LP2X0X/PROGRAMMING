---
tags: 
 - html
 - element
 - media
---

## 🖼️ `<figure>` Element — Overview

### 📌 Purpose

Represents **self-contained content** that is **referenced from the main flow**, but could be **moved elsewhere without breaking meaning**.

Often paired with `<figcaption>`.

---

### 🧠 Mental Model

> `<figure>` is for **content that illustrates or supports the surrounding text**, but is not the text itself.

---

### 📎 Common Use Cases

- Images with captions
    
- Diagrams
    
- Charts
    
- Code snippets
    
- Illustrations
    
- Videos or audio with descriptions
    

```html
<figure>
  <img src="chart.png" alt="Sales growth chart" />
  <figcaption>Sales growth in 2024</figcaption>
</figure>
```

---

### 🧩 `<figure>` vs `<img>`

- `<img>` alone → just an image
    
- `<figure>` → image + semantic grouping + caption
    

Use `<figure>` when the media needs **context or explanation**.


---

### 🔄 Placement Rules

- Can appear inside:
    
    - `article`
        
    - `section`
        
    - `main`
        
- Can be moved elsewhere without losing meaning
    

This distinguishes `<figure>` from inline media.

---

### ⚠️ When NOT to Use `<figure>`

- Decorative images
    
- Icons
    
- Layout-only visuals
    
- Images that are integral to a paragraph’s text flow
    

---

### 🧠 Accessibility Notes

- Always include meaningful `alt` text
    
- `<figcaption>` improves screen reader context
    
- `<figure>` itself does not replace `alt`
    

---

### 🧩 `<figure>` vs `<section>`

- `<figure>` → supporting content
    
- `<section>` → thematic grouping of text content
    

---

### 🧠 Summary

- `<figure>` groups self-contained, referenced content
    
- Often paired with `<figcaption>`
    
- Content remains meaningful if moved
    
- Improves semantics and accessibility
    