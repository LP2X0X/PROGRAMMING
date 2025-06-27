---
tags: css, responsive
---

- Transitioning to using less absolute units like percentages and viewport units (`vw`/`vh`) is key for a flexible design.

### _**Percentages**_

- Beginners must be careful with percentages. It takes time to understand the concept of parent-child relationships and that when a percentage is given to a child, it is a percentage of the size of its parent/container (interchangeable terms), not the whole screen.

```ad-note
- Another point here is that all the outside elements that seemingly “don’t have a parent” actually do - the `<body>` element. And the body’s size is as follows:
	- Width - the width of the screen
	- Height - the height of the content inside of it (0 if nothing is in the body)
```

---

### _**Viewport width/height (**_`vw`_**/**_`vh`_**)**_

- When you want an element to be sized relative to the screen, thus having no relation to the size of its direct container, you want to use `vw` and `vh`.

- One example is the following. Let’s say your website is meant to have a `<header>` then a `<main>` section, and you want to specifically size the height of the header and have the main section take up the rest of the screen’s height. One way to accomplish this is the following:

```css
header {
  height: 300px;
}

main {
  height: calc(100vh - 300px);
}
```

- One `vh` unit is basically 1% of the viewport height (the height of the screen). Therefore, `100vh` means 100% of the height of the screen, and thus `calc(100vh - 300px)` means “100% of the screen height minus 300px.”

- This ensures the main section will take up the remainder of the height of the screen after the header.

- You could also achieve this result with flex, but I’ll talk about that later. In this specific case, I think either is fine. Maybe one method will prove better as the project grows in complexity.

### **When to use**_ `px`_

- Having these other options and the ones I will detail below definitely do not mean that the `px` unit has no place in CSS nowadays. There are still many situations in which you want something to have a specific size that doesn’t change along with the screen.

- Many elements in a UI design may prefer a specific size that will never change. Often buttons are sized this way, for example.