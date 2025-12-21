---
tags: 
 - css
 - technique
---

## The Structure and Scope of Grid layout

To fully understand how centering works in a grid container, it's important to first understand the structure and scope of grid layout.

The HTML **structure** of a grid container has three levels:

- the container
- the item
- the content

Each of these levels is independent from the others, in terms of applying grid properties.

The **scope** of a grid container is limited to a parent-child relationship.

This means that a grid container is always the parent and a grid item is always the child. Grid properties work only within this relationship.

Descendants of a grid container beyond the children are not part of grid layout and will not accept grid properties. (At least not until the [**`subgrid`**](https://stackoverflow.com/q/47929369/3597276) feature has been implemented, which will allow descendants of grid items to respect the lines of the primary container.)

Here's an example of the structure and scope concepts described above.

Imagine a tic-tac-toe-like grid.

```css
article {
  display: inline-grid;
  grid-template-rows: 100px 100px 100px;
  grid-template-columns: 100px 100px 100px;
  grid-gap: 3px;
}
```

![[Pasted image 20251221203138.png|center|300]]

You want the X's and O's centered in each cell.

So you apply the centering at the container level:

```css
article {
  display: inline-grid;
  grid-template-rows: 100px 100px 100px;
  grid-template-columns: 100px 100px 100px;
  grid-gap: 3px;
  justify-items: center;
}
```

But because of the structure and scope of grid layout, `justify-items` on the container centers the grid items, not the content (at least not directly).

![[Pasted image 20251221203227.png|center|300]]

Same problem with `align-items`: The content may be centered as a by-product, but you've lost the layout design.

```css
article {
  display: inline-grid;
  grid-template-rows: 100px 100px 100px;
  grid-template-columns: 100px 100px 100px;
  grid-gap: 3px;
  justify-items: center;
  align-items: center;
}
```

![[Pasted image 20251221203450.png|center|300]]

To center the content you need to take a different approach. Referring again to the structure and scope of grid layout, you need to treat the grid item as the parent and the content as the child.

```css
article {
  display: inline-grid;
  grid-template-rows: 100px 100px 100px;
  grid-template-columns: 100px 100px 100px;
  grid-gap: 3px;
}

section {
  display: flex;
  justify-content: center;
  align-items: center;
  border: 2px solid black;
  font-size: 3em;
}
```

![[Pasted image 20251221203537.png|center|300]]

---

## Six Methods for Centering in CSS Grid

There are multiple methods for centering grid items and their content.

Here's a basic 2x2 grid:

```css
grid-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-auto-rows: 75px;
  grid-gap: 10px;
}


/* can ignore styles below; decorative only */
grid-container {
  background-color: lightyellow;
  border: 1px solid #bbb;
  padding: 10px;
}
grid-item {
  background-color: lightgreen;
  border: 1px solid #ccc;
}
```

```xml
<grid-container>
  <grid-item>this text should be centered</grid-item>
  <grid-item>this text should be centered</grid-item>
  <grid-item><img src="http://i.imgur.com/60PVLis.png" width="50" height="50" alt=""></grid-item>
  <grid-item><img src="http://i.imgur.com/60PVLis.png" width="50" height="50" alt=""></grid-item>
</grid-container>
```

## Flexbox

For a simple and easy way to center the content of grid items use flexbox.

More specifically, make the grid item into a flex container.

There is no conflict, spec violation or other problem with this method. It's clean and valid.

```css
grid-item {
  display: flex;
  align-items: center;
  justify-content: center;
}
```

Show code snippet

See this post for a complete explanation:

- [How to Center Elements Vertically and Horizontally in Flexbox](https://stackoverflow.com/a/33049198/3597276)

---

## Grid Layout

In the same way that a flex item can also be a flex container, a grid item can also be a grid container. This solution is similar to the flexbox solution above, except centering is done with grid, not flex, properties.

Show code snippet

---

## `auto` margins

Use `margin: auto` to vertically and horizontally center grid items.

```css
grid-item {
  margin: auto;
}
```

Show code snippet

To center the content of grid items you need to make the item into a grid (or flex) container, wrap anonymous items in their own elements ([since they cannot be directly targeted by CSS](https://stackoverflow.com/q/43027851/3597276)), and apply the margins to the new elements.

```css
grid-item {
  display: flex;
}

span, img {
  margin: auto;
}
```

Show code snippet

---

## Box Alignment Properties

When considering using the following properties to align grid items, read the section on `auto` margins above.

- `align-items`
- `justify-items`
- `align-self`
- `justify-self`

[https://www.w3.org/TR/css-align-3/#property-index](https://www.w3.org/TR/css-align-3/#property-index)

---

## `text-align: center`

To center content horizontally in a grid item, you can use the [**`text-align`**](https://developer.mozilla.org/en-US/docs/Web/CSS/text-align) property.

Show code snippet

Note that for vertical centering, `vertical-align: middle` will not work.

This is because the [**`vertical-align`**](https://developer.mozilla.org/en-US/docs/Web/CSS/vertical-align) property applies only to inline and table-cell containers.

Show code snippet

One might say that `display: inline-grid` establishes an inline-level container, and that would be true. So why doesn't `vertical-align` work in grid items?

The reason is that in a [**grid formatting context**](https://www.w3.org/TR/css3-grid-layout/#grid-item-display), items are treated as block-level elements.

> [**6.1. Grid Item Display**](https://www.w3.org/TR/css3-grid-layout/#grid-item-display)
> 
> The `display` value of a grid item is **_blockified_**: if the specified `display` of an in-flow child of an element generating a grid container is an inline-level value, it computes to its block-level equivalent.

In a [**block formatting context**](https://www.w3.org/TR/CSS22/visuren.html#block-formatting), something the `vertical-align` property was originally designed for, the browser doesn't expect to find a block-level element in an inline-level container. That's invalid HTML.

---

## CSS Positioning

Lastly, there's a general CSS centering solution that also works in Grid: _absolute positioning_

This is a good method for centering objects that need to be removed from the document flow. For example, if you want to:

- [Center text over an image](https://stackoverflow.com/q/35871294/3597276), or
- [Place asterisks (*) above required form fields](https://stackoverflow.com/q/45599317/3597276)

Simply set `position: absolute` on the element to be centered, and `position: relative` on the ancestor that will serve as the containing block (it's usually the parent). Something like this:

```css
grid-item {
  position: relative;
  text-align: center;
}

span {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
```

Show code snippet

Here's a complete explanation for how this method works:

- [Element will not stay centered, especially when re-sizing screen](https://stackoverflow.com/q/36817249/3597276)

Here's the section on absolute positioning in the Grid spec:

- [https://www.w3.org/TR/css3-grid-layout/#abspos](https://www.w3.org/TR/css3-grid-layout/#abspos)

---

### References
https://stackoverflow.com/questions/45536537/centering-in-css-grid