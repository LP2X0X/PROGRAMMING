---
tags:
  - css
  - property
  - fundamental
---

In CSS, the position property defines how an element is positioned in the document. There are **five main values**:

---

### **🔧 position Values**

| **Value** | **Description**                                                                                                                                                      |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| static    | Default. Element is in the normal document flow.                                                                                                                     |
| relative  | Offsets the element relative to its **normal position**.                                                                                                             |
| absolute  | Removes the element from flow, no space is created for the element in the page layout and positions it **relative to the nearest positioned ancestor** (not static). |
| fixed     | Positions element relative to the **viewport**; stays fixed during scroll.                                                                                           |
| sticky    | Scrolls normally until it reaches a threshold, then sticks in place.                                                                                                 |

```ad-note
The element is positioned relative to its closest positioned ancestor (if any) or to the initial [[The Containing Block|containing block]].
```
  
---

### ❓How it affect the flow?

Normally, elements are laid out one after another, block by block or inline by inline. This is called the **[[Normal Document Flow]].

When an element is **taken out of flow**, it no longer affects the positioning of other elements around it (siblings behave as if it isn’t there).


#### **🖋️ Position Values & Flow Behavior**

- **static** (default)
    - ✅ Stays in normal flow.
    - No special positioning applied.
- **relative**
    - ✅ Stays in normal flow (still occupies its original space).
    - ❗ Moves visually relative to its original position, but surrounding elements act as if it didn’t move.
- **absolute**
    - ❌ Taken out of flow.
    - Positioned relative to the nearest positioned ancestor (relative, absolute, fixed, or sticky).
    - Does not affect sibling layout — they behave as if it’s not there.
- **fixed**
    - ❌ Taken out of flow.
    - Positioned relative to the **viewport** (browser window).
    - Stays in place even when scrolling.
- **sticky**
    - ✅ Initially in normal flow.
    - ❗ Switches to fixed-like behavior when a scroll threshold is reached.
    - Still occupies space in the flow (siblings are aware of it).

#### **💡 Example**

```
.box1 {
  position: relative; /* stays in flow */
  top: 20px;
}
.box2 {
  position: absolute; /* out of flow */
  top: 0;
  right: 0;
}
```

Here, .box1 still affects others’ positions, but .box2 doesn’t — it’s removed from the flow.

#### **🛠️ Tips & Best Practices**

- Use **relative** when you want small offsets but still want the element to “reserve space.”
- Use **absolute/fixed** sparingly for floating UI (tooltips, modals, etc.), not for main layouts.
- Remember that removing an element from flow may cause overlaps, so you’ll often need z-index.


---

### **🧠 Example:**

```html
<div class="box">I'm positioned</div>
```

```css
.box {
  position: absolute;
  top: 50px;
  left: 100px;
}
```

This positions .box **50px from the top** and **100px from the left** of the **nearest non-static parent**.

---

### **🧲 Quick Tips:**

- Always pair absolute, relative, fixed, or sticky with top, left, right, or bottom.
    
- Absolute looks for the nearest ancestor with position: relative, absolute, or fixed.
    
- Sticky works only if its parent has a defined height and overflow allows scrolling.
    
