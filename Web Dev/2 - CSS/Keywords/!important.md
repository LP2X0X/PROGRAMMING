---
tags: css, keyword, fundamental
---

- If you write `!important` at the end of a CSS property, it will take precedence over every other rule. Here's an example.

```html
<p id="my-text">Some text</p>
```

```html
p {
  color: red !important;
}

#my-text {
  color: blue;
}
```

- While the second selector has more specificity, the text will be **red** because we have added `!important` to the end of the property value. This can be done on a property-by-property basis. For example:

```html
p {
  color: red !important;
  font-size: 18px;
}

#my-text {
  color: blue;
  font-size: 16px;
}
```

- In this case, the color will be red but the font size will be `16px`.

**Now let's have a little chat** about this `!important` keyword. While it does provide you with an easy way to get a CSS style working without having to think much, it is **usually a bad practice**. There is no definitive guideline as to when this is acceptable to use, but here is a good rule of thumb.

```ad-tip
Use !important if doing so is the ONLY viable method for styling an element
```

- This could occur if you are trying to override styles applied from an **external stylesheet** (CSS frameworks, UI libraries, etc.).