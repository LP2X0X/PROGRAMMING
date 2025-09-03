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