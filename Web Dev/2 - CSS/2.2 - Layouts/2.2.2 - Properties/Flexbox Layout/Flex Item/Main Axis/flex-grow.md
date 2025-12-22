---
tags: 
  - css
  - property
  - flex
  - item
  - fundamental
---

# **📖 Definition**

The **flex-grow** property defines how much a **flex item** should grow **relative to the other flex items** inside the same container when **extra space** is available along the main axis.

If no extra space exists, it has no effect.

```ad-important
The effected size here are the size corresponding to the main axis.
```

```ad-note
The children of a row-flexbox container automatically fill the container's vertical space.
```

---

# **🖋️ Syntax**

```css
flex-grow: <number>;
```

- \<number> = unitless proportion (can be 0, 1, 2, etc.).
    
- Default: 0 (items do **not** grow).
    

---

# **💡 Example**

```css
.container {
  display: flex;
  width: 600px;
}

.item {
  flex-grow: 1;
}

.item:first-child {
  flex-grow: 2; /* takes double the space of others */
}
```

HTML:

```html
<div class="container">
  <div class="item">A</div>
  <div class="item">B</div>
  <div class="item">C</div>
</div>
```

👉 Distribution:

- Available extra space is divided into **4 parts** (2 + 1 + 1).
    
- A gets **2/4**, B gets **1/4**, C gets **1/4**.
    

---

# **🛠️ Tips & Best Practices**

- ✅ Use flex-grow: 1 on items you want to **fill remaining space equally**.
    
- ✅ Larger numbers = more space compared to siblings.
    
- ⚠️ Don’t use absolute pixel thinking → it’s **proportional distribution**, not fixed sizing.
    
- ✅ Combine with flex-shrink and flex-basis via shorthand flex: grow shrink basis;.
    
- ⚠️ If all items have flex-grow: 0, no item will grow.
    

---

# **📊 Quick Facts**

|**Property**|**Default**|**Meaning**|
|---|---|---|
|flex-grow|0|Item does not grow|
|flex-grow: 1|Item grows equally with siblings||
|flex-grow: n|Item takes **n times more** available space||

---

👉 In short: **flex-grow controls how flex items share leftover space.**