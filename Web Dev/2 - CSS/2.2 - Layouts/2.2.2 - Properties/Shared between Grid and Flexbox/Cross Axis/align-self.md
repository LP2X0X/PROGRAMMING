---
tags: 
  - css
  - property
  - item
  - fundamental
---

# **📖 Definition**

The **align-self** property in CSS allows you to **override the container’s align-items value** for an individual flex/grid item.

It controls the **cross-axis alignment** of a single item inside a flexbox or grid.

---

# **🖋️ Syntax**

```css
align-self: auto | flex-start | flex-end | center | baseline | stretch;
```

- auto → uses the parent container’s align-items value (default).
    
- flex-start → aligns item to the **start** of the cross axis.
    
- flex-end → aligns item to the **end** of the cross axis.
    
- center → aligns item in the **center** of the cross axis.
    
- baseline → aligns item’s text baseline with others.
    
- stretch → stretches item to fill the cross axis (if no fixed size is set).
    

---

# **💡 Example**

```css
.container {
  display: flex;
  align-items: stretch; /* default for all children */
  height: 200px;
}

.item {
  flex: 1;
  background: lightblue;
}

.item.special {
  align-self: center; /* overrides align-items for this item only */
}
```

---

# **🛠️ Tips & Best Practices**

- ✅ Use align-self only when one or a few items need different alignment.
    
- ✅ Useful for “highlight” items (e.g., a special button in a navbar).
    
- ⚠️ Works **only on flex/grid items** (children of a flex/grid container).
    
- ⚠️ For inline content alignment, use vertical-align instead.
    

---

# **📊 Quick Comparison: align-items vs align-self**

| **Property** | **Scope**       | **Affects**                             |
| ------------ | --------------- | --------------------------------------- |
| align-items  | Container-level | All flex/grid items (unless overridden) |
| align-self   | Item-level      | One specific flex/grid item             |

---

👉 In short: **align-items = group rule** and **align-self = exception rule**.