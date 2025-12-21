---
tags: 
 - html
 - element
---

## HTML `<section>` — compact overview

#### **What it is**  
The \<section> HTML element represents a generic standalone section of a document, which doesn't have a more specific semantic element to represent it. Sections should always have a heading, with very few exceptions.

---

#### **When to use it**

- Use when content has a clear topic or purpose and benefits from its **own heading**.
    
- Use to break long pages into logical parts (intro, features, FAQ, contact).
    
- Use inside articles, or to structure page sections.
    

---

#### **When _not_ to use it**

- Not for styling or layout only — use `<div>`.
    
- Not for self-contained, independently distributable content — use `<article>`.
    
- Not for navigation or side content — use `<nav>` or `<aside>`.
    

---

#### **Key characteristics**

- Creates a “section” in the **document outline** (if it has a heading).
    
- Can contain other sectioning elements (nested sections allowed).
    
- Works best when paired with a heading (`<h2>`, `<h3>`, etc.).
    

---

#### **Semantic differences**

- `<section>` = part of a larger whole; usually needs context.
    
- `<article>` = self-contained piece of content that stands on its own.
    
- `<div>` = generic container with no semantic meaning.
    
- `<aside>` = tangential or supplementary content.
    

--- 

#### **Best practices**

- Always include a heading to define the section's purpose.
    
- Don’t overuse; use only when it adds semantic meaning.
    
- Use `<section>` to logically group content, not to wrap every block.
    

---

#### **Small example**

```html
<section id="features">
  <h2>Main Features</h2>
  <p>Feature description...</p>
  <ul>
    <li>Fast</li>
    <li>Secure</li>
    <li>Lightweight</li>
  </ul>
</section>
```