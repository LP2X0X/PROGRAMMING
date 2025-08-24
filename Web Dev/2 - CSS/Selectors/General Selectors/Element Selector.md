---
tags:
  - css
  - selector
  - fundamental
---

## **📖 Definition**

The **element selector** in CSS targets all instances of a given HTML tag. It applies styles directly to every occurrence of that tag within the page.

---

## **🖋️ Syntax**

```css
element {
  /* styles */
}
```

- element is the HTML tag name (e.g., p, h1, ul, a).
    
- No prefix is needed — just the tag name.
    

---

## **💡 Example**

HTML:

```html
<p>This is a paragraph.</p>
<p>Another paragraph.</p>
```

CSS:

```css
p {
  color: gray;
  line-height: 1.6;
}
```

This will style **all \<p> elements** to be gray with increased line spacing.

---

## **🛠️ Tips & Best Practices**

- **Global styles**: Best for base styling (e.g., body font, link colors, reset margins).
    
- **Don’t overuse**: Avoid styling broad tags like div or span directly — use classes for reusable styles.
    
- **Inheritance awareness**: Styles cascade down, so applying color or font on an element may affect children.
    
- **Keep it simple**: Avoid overly specific nested selectors (div ul li span) — use classes instead.
    
- **Combine with resets/normalizers** to remove inconsistent browser defaults (e.g., margins on h1, bullet points on ul).
    