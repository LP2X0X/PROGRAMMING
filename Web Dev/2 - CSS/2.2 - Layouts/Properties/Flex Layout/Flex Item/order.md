---
tags: 
  - css
  - property
  - flex
  - item
  - fundamental
---

# **📖 Definition**

The **order** property in CSS defines the order in which **flex items** (or grid items) appear inside their container.

By default, all items have order: 0. Items are laid out in ascending order of this value.

  

It **does not change the source order in HTML**, only the _visual order_.

---

# **🖋️ Syntax**

```css
order: <integer>;
```

- Can be positive, negative, or zero.
    
- Default is 0.
    

---

# **💡 Example**

```css
.container {
  display: flex;
}

.item {
  flex: 1;
  padding: 10px;
  background: lightblue;
}

.item:nth-child(2) {
  order: -1; /* moves second item to the front */
}
```

If HTML is:

```html
<div class="container">
  <div class="item">1</div>
  <div class="item">2</div>
  <div class="item">3</div>
</div>
```

The display order will be **2, 1, 3**.

---

# **🛠️ Tips & Best Practices**

- ✅ Use order for small reordering adjustments (like moving a button or icon).
    
- ⚠️ Don’t rely on it for major layout changes → Screen readers and keyboard navigation still follow **HTML source order**, not visual order.
    
- ✅ Negative values are allowed and useful for pushing items to the start.
    
- ✅ Combine with media queries for responsive reordering.
    

---

# **📊 Quick Facts**

|**Property**|**Default**|**Affects**|
|---|---|---|
|order|0|Position of one flex/grid item relative to siblings|

---

👉 In short: **order lets you shuffle items visually without touching HTML.**