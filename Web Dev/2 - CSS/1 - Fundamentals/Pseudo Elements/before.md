---
tags: css, pseudo, element, fundamental
---


- **What it is:** The `::before` pseudo-element is an inline box generated as an immediate child of the element it is associated with, or the "originating element". It is often used to add cosmetic content to an element via the content property, such as icons, quote marks, or other decoration.
- **Usage:** Often used for decorative purposes, icons, labels, or adding extra content via CSS.
- **Requirements:** Needs `content: ...;` in CSS to display anything (even an empty string `""`).
- **Example:**
    ```css
    /* Adds a star before every `<h1>` text.*/
    h1::before {
      content: "★ ";
      color: gold;
    }
    ```
