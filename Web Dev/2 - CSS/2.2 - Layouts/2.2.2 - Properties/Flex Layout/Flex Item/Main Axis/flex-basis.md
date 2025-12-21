---
tags: 
  - css
  - property
  - flex
  - item
  - fundamental
---

📖 **Definition**

flex-basis sets the **initial main size** (the size corresponding to the main axis) of a flex item before extra space is distributed (via flex-grow) or items shrink (via flex-shrink).

- It overrides width (or height, depending on flex direction) when used inside a flex container.
    

```ad-note
If the item require more space than the flex-basis then it will be override.
```

---

🖋️ **Syntax**

```css
.item {
  flex-basis: <length> | auto;
}
```

- \<length> → e.g., 100px, 30%, 10rem
    
- auto → uses the item’s width / height (or content size if no explicit size is given)
    

---

💡 **Examples**

1. **Fixed initial size**
    

```css
.item {
  flex-basis: 200px;
  flex-grow: 1;
}
```

👉 Each item starts at **200px**, then grows if space remains.

2. **Content-based size (auto)**
    

```css
.item {
  flex-basis: auto;
}
```

👉 Uses the item’s natural content size or its width property as the starting point.

---

🛠️ **Tips & Best Practices**

- 🔄 **Main axis aware**: If flex-direction: row, flex-basis controls **width**. If flex-direction: column, it controls **height**.
    
- 🎯 **Overrides width/height**: If both are set, flex-basis wins inside a flex container.
    
- ⚖️ **Plays with grow/shrink**: Think of it as the “preferred starting size” before the container redistributes space:
	- **If there’s extra space** in the flex container → flex-grow tells how much an item expands beyond its flex-basis.
	- **If there’s not enough space** → flex-shrink tells how much an item shrinks below its flex-basis.
- 📦 **Shortcut**: Often used with the flex shorthand:
    

```css
flex: 1 1 200px; /* grow, shrink, basis */
```

  

---

✨ Mental model:

- width = “how wide I’d be in normal flow.”
    
- flex-basis = “how wide I want to start in flex layout before negotiation.”
    

---

### Why flex-basis needed

They could have done without flex-basis and make us use the already existing width but it would be.. kind of weird.

The width property sets the size of the element and that's it. What actual size will an element with the width property be? Well the size set by width. It doesn't matter if the element is a float, if it positioned, if it is in a grid, in a container with the normal (flow) layout,... The actual width of the element is the value of width that's set, even if it causes overflow.

(Technically min-width and max-width can override width but I consider them part of the same : "here is the size of this element").

There is one big exception to this : flexbox. When using flexbox the width of the items is not, in fact, the size it will actually be. It's merely a suggestion, a basis upon which flexbox decide the actual final size. With flexbox, the width of the items serve as a base the layout use to determine what the final width will be.

Setting the width on a flex item does not actually sets it's width. Weird.

A good way to understand flex is that it's a three steps process:

1. The flexbox lays out all the items one after another on a line at the size they ideally would like to be (as determined by flex-basis)
    
2. When that's done, we take the total space all those items take and we compare that to the space actually available (the size of the flex container). We then know the amount of free space for items to grow or the space we need to gain by shrinking the items.
    
3. we can them shrink/grow the items as needed.
    

So why flex-basis instead of width? Because with flexbox and contrary to all the other scenarios, width does not actually sets the width of an item. flex-basis is introduced instead of using width "against its nature".  
It avoids reading CSS code seeing `width: xxx;` and be surprised when the size is not actually `xxx`.