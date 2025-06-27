---
tags: css, property, fundamental
---

The display property in CSS controls how an HTML element is displayed on the page. It’s one of the most fundamental properties in layout design.


**Common Values of display**

|**Value**|**Description**|
|---|---|
|block|Element takes up the full width available. Starts on a new line (e.g., \<div>, \<p>).|
|inline|Element takes up only as much width as it needs. Doesn’t start on a new line (e.g., \<span>).|
|inline-block|Like inline, but you can set width and height.|
|none|Hides the element from the page (it won’t take up space).|
|flex|Makes the element a flex container for flexible layout.|
|grid|Makes the element a grid container for grid layout.|
|inline-flex|Same as flex, but behaves like inline.|
|inline-grid|Same as grid, but behaves like inline.|
|table|Makes the element behave like a \<table>.|
|list-item|Makes the element behave like a list item (\<li>).|

**Example**

```css
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}
```
