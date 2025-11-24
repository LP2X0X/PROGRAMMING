---
tags: 
 - css
 - term
 - fundamental
---


**Browser styles** (also called **user agent styles**) are the **default CSS rules** that every browser applies to HTML elements before your own CSS is loaded.

These built-in styles exist so that a plain HTML page (with no CSS at all) is still **readable and usable**.

---

## **🖋️ Examples of Browser Styles**

- Headings \<h1> … \<h6> → bold, with descending sizes.
- Paragraph \<p> → block display, has margin above and below.
- Lists \<ul> and \<ol> → indented with bullets/numbers.
- Links \<a> → blue, underlined (purple when visited).
- Buttons \<button> → styled according to system theme.
- Forms → \<input> fields, checkboxes, and dropdowns all have default UI.

If you open a blank HTML file with just:

```html
<h1>Hello</h1>
<p>This is a paragraph.</p>
<ul>
  <li>Item 1</li>
  <li>Item 2</li>
</ul>
```

It will already look styled — thanks to browser styles.

---

## **🔎 Where Do They Come From?**

- They are defined in the browser’s **user agent stylesheet** (hidden CSS file shipped with Chrome, Firefox, Safari, etc.).
    
- You can inspect them in **DevTools → Styles panel**, usually shown under _“user agent stylesheet”_.
    

---

## **💡 Best Practices**

- **Reset/normalize CSS**: Many developers use CSS reset libraries (like Normalize.css) to remove inconsistencies between browsers.
    
- **Don’t fight defaults unnecessarily**: Sometimes it’s easier to build on defaults (e.g., list indentation) instead of resetting everything.
    
- **Be aware of inheritance**: e.g., body { font-family: sans-serif; } overrides the browser’s default serif font.
    

---

✅ Example (resetting styles):

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
```

This removes the browser’s default margins/paddings and makes styles consistent across elements.

---

## ☝️ Here are the **most common browser styles to watch out for**:

## 🔹 Text & Headings

- **`<p>`** → `margin-top: 1em; margin-bottom: 1em;`
    
- **`<h1>–<h6>`** → Large `font-size`, with `margin-block-start` / `margin-block-end` in `em`.
    
- **`<blockquote>`** → Extra left/right margins.
    
- **`<pre>`** → `font-family: monospace; white-space: pre;`
    

---

## 🔹 Lists

- **`<ul>, <ol>`** → `margin-block-start: 1em; margin-block-end: 1em; padding-left: 40px;`
    
- **`<li>`** → `display: list-item;` with default bullets/numbers.
    

---

## 🔹 Forms

- **`<button>, <input>, <select>, <textarea>`**
    
    - Have **padding, border, and font styles** applied differently per browser.
        
    - Example: Chrome adds `padding: 1px 6px` and `border: 2px outset`.
        
- **`<label>`** → `cursor: default;`
    

---

## 🔹 Links

- **`<a>`** → blue text + `text-decoration: underline;`
    
- **`:visited`** → purple, `:active` → red by default.
    

---

## 🔹 Body & Document

- **`<body>`** → `margin: 8px;` (that’s why elements aren’t flush with the viewport edge).
    
- **`<html>`** → `display: block;`
    

---

## 🔹 Tables

- **`<table>`** → `display: table; border-collapse: separate; border-spacing: 2px;`
    
- **`<th>, <td>`** → `padding: 1px;`
    

---

## 🔹 Media

- **`<img>`** → `display: inline;` (causes baseline gap under images).
    
- **`<video>, <audio>`** → default UI controls.
    

---

## ✅ Common Practice

Developers often “reset” or “normalize” these styles to avoid surprises:

- **CSS Reset** (e.g. Eric Meyer Reset, Josh Comeau’s Modern Reset).
    
- **Normalize.css** → keeps useful defaults but makes them consistent across browsers.
    