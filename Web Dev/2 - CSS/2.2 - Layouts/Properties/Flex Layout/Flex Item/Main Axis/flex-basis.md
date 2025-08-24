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
    