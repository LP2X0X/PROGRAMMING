---
tags: 
 - html
 - element
 - heading
 - fundamental
---

### 🎯 Purpose

The **`<hgroup>`** [HTML](https://developer.mozilla.org/en-US/docs/Web/HTML) element represents a heading and related content. It groups a single [`<h1>–<h6>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/Heading_Elements) element with one or more [`<p>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/p).

The `<hgroup>` element allows the grouping of a heading with any secondary content, such as subheadings, an alternative title, or tagline. Each of these types of content represented as a `<p>` element within the `<hgroup>`.

The `<hgroup>` itself has no impact on the document outline of a web page. Rather, the single allowed heading within the `<hgroup>` contributes to the document outline.

The `<hgroup>` element represents a **heading and its related content**.

It groups:

- **Exactly one** heading element (`<h1>`–`<h6>`)
    
- With **one or more related elements**, typically `<p>` (e.g. subtitle, tagline, description)
    

---

### 📦 What `<hgroup>` contains

- **One heading element** (`<h1>`–`<h6>`)
    
- **Associated content**, commonly:
    
    - `<p>`
        
    - phrasing content related to the heading
        

```html
<hgroup>
  <h1>CSS Layout</h1>
  <p>Flexbox, Grid, and Positioning</p>
</hgroup>
```

✔ This represents **one logical heading**, not multiple headings.

---

### 🧠 Semantic meaning

- The heading (`<h1>`–`<h6>`) defines the **section’s rank**
    
- The accompanying content provides **context or clarification**
    
- Assistive technologies treat the group as:
    
    > “One heading with supporting text”
    

---

### 🗂 Document outline behavior

- `<hgroup>` **does not create a new section**
    
- Only the contained heading contributes to the document outline
    
- The related content does **not** affect heading hierarchy
    

---

### ♿ Accessibility benefits

- Prevents subtitles or taglines from being mistaken as separate sections
    
- Improves screen reader announcements by keeping heading context intact
    

---

### ❌ What `<hgroup>` is NOT

- ❌ Not a container for multiple headings
    
- ❌ Not a general-purpose wrapper
    
- ❌ Not a layout element
    

---

### 🛠 Historical note (important)

- Earlier specs defined `<hgroup>` as grouping **multiple headings**
    
- That definition was **removed**
    
- The element was later **reintroduced with new semantics**:
    
    - **one heading + related content**
        

This is the source of widespread confusion.

---

### ✅ When to use

- A title with a subtitle
    
- A heading with a short description
    
- Article or section headers with supporting text
    

---

### 🧠 Key takeaway

> `<hgroup>` groups **one heading** with **its related descriptive content** ensuring they are interpreted as a **single logical heading unit**.
