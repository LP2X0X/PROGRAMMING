---
tags: 
  - css
  - property
  - flex
  - container
  - fundamental
---

`flex-wrap` controls **whether flex items are allowed to move to the next line** when they run out of horizontal space.

By default, **flex items never wrap** and will shrink to fit.

---

# ✔️ The Values

### **1. `flex-wrap: nowrap` (default)**

All items stay on **one single line**, even if they must shrink.

```
[ A ][ B ][ C ][ D ]  ← all forced into one line
```

### **2. `flex-wrap: wrap`**

Allows items to wrap onto **multiple lines** when they don’t fit.

```
[ A ][ B ][ C ]
[ D ][ E ]
```

### **3. `flex-wrap: wrap-reverse`**

Same as `wrap`, but the **new line appears above** the previous line.

```
[ D ][ E ]
[ A ][ B ][ C ]
```

---

# ✔️ Why `flex-wrap` Matters

Without wrapping:

- flex items can shrink so small they break UI
    
- grids built with Flexbox may collapse
    

With wrapping:

- items maintain size better
    
- you can create responsive row-based layouts
    

---

# 🌟 Interaction With `flex-direction`

`flex-direction` decides **the axis**, and `flex-wrap` wraps _perpendicular_ to that axis:

- `flex-direction: row` → wrap makes _new rows_
    
- `flex-direction: column` → wrap makes _new columns_
    

This is a big reason why wrapping is useful only when direction = **row**.

---

# ✔️ Tailwind Version

|CSS|Tailwind|
|---|---|
|`flex-wrap: nowrap`|`flex-nowrap`|
|`flex-wrap: wrap`|`flex-wrap`|
|`flex-wrap: wrap-reverse`|`flex-wrap-reverse`|

Example:

```jsx
<div className="flex flex-wrap gap-2">
  <button className="p-2 bg-gray-200">A</button>
  <button className="p-2 bg-gray-200">B</button>
  <button className="p-2 bg-gray-200">C</button>
  <button className="p-2 bg-gray-200">D</button>
</div>
```

---

# 🚀 TL;DR

- Flexbox normally **keeps everything on one line**.
    
- `flex-wrap` lets items **break into multiple lines**.
    
- Use `flex-wrap` + `gap` to build responsive button rows, tag lists, etc.
    