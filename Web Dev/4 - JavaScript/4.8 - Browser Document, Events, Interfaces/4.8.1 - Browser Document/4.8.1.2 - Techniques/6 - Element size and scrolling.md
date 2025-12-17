---
tags: 
 - js
 - technique
 - DOM
---

## Sample element

```html
<div id="example">
  ...Text...
</div>
<style>
  #example {
    width: 300px;
    height: 200px;
    border: 25px solid #E8C48F;
    padding: 20px;
    overflow: auto;
  }
</style>
```

![[Pasted image 20251211100446.png|center|500]]

```ad-note
The picture above demonstrates the most complex case when the element has a scrollbar. Some browsers (not all) reserve the space for it by taking it from the content (labeled as “content width” above).

So, without scrollbar the content width would be `300px`, but if the scrollbar is `16px` wide (the width may vary between devices and browsers) then only `300 - 16 = 284px` remains, and we should take it into account. That’s why examples from this chapter assume that there’s a scrollbar. Without it, some calculations are simpler.
```

---

## Geometry 

- Here’s the overall picture with geometry properties:
![[Pasted image 20251211100757.png|center|500]]

### offsetParent, offsetLeft/Top

- The **offsetParent** is the nearest ancestor that the browser uses for calculating coordinates during rendering.
	- That’s the nearest ancestor that is one of the following: CSS-positioned (position is absolute, relative, fixed or sticky), or \<td>, \<th>, or \<table>, or \<body>.

- Properties **offsetLeft/offsetTop** provide x/y coordinates relative to offsetParent upper-left corner.

```html
<main style="position: relative" id="main">
  <article>
    <div id="example" style="position: absolute; left: 180px; top: 180px">...</div>
  </article>
</main>
<script>
  alert(example.offsetParent.id); // main
  alert(example.offsetLeft); // 180 (note: a number, not a string "180px")
  alert(example.offsetTop); // 180
</script>
```

![[Pasted image 20251211101200.png|center|500]]

### offsetWidth/Height

- These two properties are the simplest ones. They provide the “outer” width/height of the element. Or, in other words, its full size including borders.

![[Pasted image 20251211101408.png|center|500]]

### clientTop/Left

To measure borders, there are properties clientTop and clientLeft.

In our example:

    clientLeft = 25 – left border width
    clientTop = 25 – top border width

![[Pasted image 20251211101813.png|center|500]]

```ad-note
`clientLeft` also includes the scrollbar width.
```

### clientWidth/Height

- These properties provide the size of the area inside the element borders.

- They include the content width together with paddings, but without the scrollbar:

![[Pasted image 20251211102130.png|center|500]]

```ad-note
**If there are no paddings, then `clientWidth/Height` is exactly the content area, inside the borders and the scrollbar (if any).**
```

### scrollWidth/Height

- These properties are like clientWidth/clientHeight, but they also include the scrolled out (hidden) parts:

![[Pasted image 20251211104841.png|center|500]]

- We can use these properties to expand the element wide to its full width/height. Like this:
```js
// expand the element to the full content height
element.style.height = `${element.scrollHeight}px`;
```

### scrollLeft/scrollTop

- Properties scrollLeft/scrollTop are the width/height of the hidden, scrolled out part of the element.

```ad-note
In other words, `scrollTop` is “how much is scrolled up”.
```

![[Pasted image 20251211105155.png|center|500]]