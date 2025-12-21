---
tags: 
  - css
  - property
  - flex
  - item
  - fundamental
---

## **🔹 What is flex ?**

The flex property is shorthand for **three properties**:

```css
flex: <flex-grow> <flex-shrink> <flex-basis>;
```

```ad-note
The children of a main-axis-flexbox container automatically fill the container's cross axis space.
```

---

## **🔹 The three parts**

1. **flex-grow** → how much the item grows if there’s extra space
    
2. **flex-shrink** → how much the item shrinks if space is tight
    
3. **flex-basis** → the starting size before growing/shrinking
    

```ad-note
If you omit the `flex` property (and longhand properties), the initial values are as follows:

- `flex-grow: 0`
- `flex-shrink: 1`
- `flex-basis: auto`

This is known as `flex: initial`.
```

---

## **🔹 Common values**

- flex: 1; → expands equally with siblings (shorthand for flex: 1 1 0%)
    
- flex: auto; → (flex: 1 1 auto) → grow/shrink allowed, basis = content size
    
- flex: none; → (flex: 0 0 auto) → no grow, no shrink, basis = content size
    
- flex: 2; → (flex: 2 1 0%) → grows twice as fast as flex: 1 items
    

---

## **🔹 Examples**

```css
.container {
  display: flex;
  width: 600px;
}

.item1 {
  flex: 1; /* grows to share space equally */
}
.item2 {
  flex: 2; /* grows twice as fast */
}
```

👉 In a 600px container:

- item1 gets 200px
    
- item2 gets 400px
    

---

## **✅ Rule of thumb**

- **flex: 1** is the most common — it makes all items grow equally to fill space.
    
- **flex: auto** respects the content size but still grows/shrinks.
    
- **flex: none** locks the size to content/basis.
    

---

⚡ So: flex = **“how this item negotiates space with others.”**