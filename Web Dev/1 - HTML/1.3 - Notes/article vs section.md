---
tags: 
 - html
 - note
 - distinguish
---

## 🧱 Core Difference (One Sentence)

**`<article>` is for standalone, reusable content.**  
**`<section>` is for grouping related content within a larger context.**

If you remove it from the page:

- `<article>` → still makes sense on its own
    
- `<section>` → usually does not
    

---

## 📰 `<article>` — Standalone Content

### Use `<article>` when the content:

- Can be **independently distributed or reused**
    
- Makes sense outside the current page
    
- Could appear in:
    
    - RSS feeds
        
    - Search results
        
    - Emails
        
    - Shared links
        

### Typical examples

- Blog post
    
- News article
    
- Forum post
    
- Product card
    
- Comment
    
- User review
    

```html
<article>
  <h2>React 19 Released</h2>
  <p>New features include...</p>
</article>
```

---

## 📦 `<section>` — Thematic Grouping

### Use `<section>` when the content:

- Groups related information
    
- Depends on surrounding context
    
- Is part of a larger whole
    
- Needs a heading but is **not standalone**
    

### Typical examples

- Page sections
    
- Feature groups
    
- Chapters of an article
    
- Form sections
    

```html
<section>
  <h2>Shipping Information</h2>
  <p>Delivery takes 3–5 days.</p>
</section>
```

---

## 🧠 Decision Rule (Very Important)

Ask yourself:

> **“If I copy-paste this block somewhere else, does it still make sense?”**

- **Yes** → `<article>`
    
- **No** → `<section>`
    

---

## 🧩 Nesting Rules (This clarifies confusion)

- An `<article>` can contain `<section>`s
    
- A `<section>` can contain `<article>`s
    

Example:

```html
<article>
  <h1>Blog Post</h1>

  <section>
    <h2>Introduction</h2>
  </section>

  <section>
    <h2>Main Content</h2>
  </section>
</article>
```

---

## ⚠️ Common Mistakes

- Using `<section>` when `<div>` is enough
    
- Using `<article>` for layout containers
    
- Using `<section>` without a heading
    
- Treating them as interchangeable
    

---

## 🧠 Mental Model (Memorable)

- `<article>` → **a thing**
    
- `<section>` → **a part of a thing**
    

---

## 📊 Quick Comparison Table

|Aspect|`<article>`|`<section>`|
|---|---|---|
|Standalone|Yes|No|
|Reusable independently|Yes|Usually no|
|Requires heading|Recommended|Required|
|Syndication-friendly|Yes|No|
|Typical use|Post, card, item|Grouping, chapter|

---

## ✅ Summary

- Use `<article>` for **self-contained, reusable content**
    
- Use `<section>` for **logical grouping inside a context**
    
- If neither fits, use `<div>`
    
