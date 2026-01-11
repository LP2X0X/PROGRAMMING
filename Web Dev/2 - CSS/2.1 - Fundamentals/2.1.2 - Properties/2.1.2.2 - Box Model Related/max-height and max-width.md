---
tags: 
 - css
 - property
 - fundamental
---

`max-width` and `max-height` define the **upper limit** for an element’s size.

They constrain how large an element is allowed to grow, **without forcing a fixed size**.

```css
.element {
  max-width: 600px;
  max-height: 400px;
}
```

---

## 🧱 How they work

- They apply **after** `width` / `height`
    
- They **override growth**, not shrinkage
    
- They do not set size by themselves unless the element would exceed them
    

Priority:

```
min-* → preferred size → max-*
```

---

## 📏 `max-width`

### Purpose

- Prevent elements from becoming too wide
    
- Commonly used for readable text layouts
    

```css
.container {
  max-width: 70ch;
  margin-inline: auto;
}
```

📌 `ch` is often used for typography-based widths.

---

## 📐 `max-height`

### Purpose

- Limit vertical growth
    
- Common for collapsible content and modals
    

```css
.panel {
  max-height: 300px;
  overflow: auto;
}
```

---

## 🧠 Interaction with intrinsic sizing

- Content can define a **natural size**
    
- `max-*` clamps that size
    

Example:

```css
img {
  max-width: 100%;
  height: auto;
}
```

➡️ Image scales down but never exceeds its container.

---

## ⚠️ Important behaviors

### 1️⃣ Percentage values

- Relative to the **containing block**
    
- `max-height` percentages require an explicit height on the parent
    

```css
.child {
  max-height: 100%;
}
```

❌ Won’t work unless the parent has a defined height.

---

### 2️⃣ Flex and Grid contexts

- `max-width` / `max-height` **still apply**
    
- They limit item growth even if layout allows more space
    

```css
.flex-item {
  max-width: 300px;
}
```

---

### 3️⃣ Animations & transitions

- `max-height` is animatable
    
- Often used for accordion effects
    

```css
.panel {
  transition: max-height 0.3s ease;
}
```

⚠️ Requires a known max value.

---

## 🆚 `width` vs `max-width`

| Property          | Behavior              |
| ----------------- | --------------------- |
| `width`           | Sets a fixed size     |
| `max-width`       | Sets an upper bound   | 
| `max-width: 100%` | Responsive constraint |

---

## 🧠 Mental model

> `max-*` defines a **ceiling**, not a size.

The element grows naturally until it hits the ceiling.

---

## ✅ Common best practices

- Use `max-width` for readable layouts
    
- Combine `max-height` with `overflow`
    
- Prefer `max-width` over fixed widths for responsiveness
    
- Use `clamp()` for adaptive sizing
    

```css
max-width: clamp(20rem, 80vw, 70rem);
```

---

## 🔑 Key takeaway

`max-width` and `max-height` are **constraints**, not dimensions. They are essential tools for responsive, content-driven layouts.

---

## ✅ Side notes

### 1️⃣ max-width for img element

- When a property of width is used then the image will be displayed at that width, relative to its container if using percentages, regardless of its own inherent size. The result would be that the img would stretch beyond its [[Intrinsic and Extrinsic|intrinsic size]] to fill 100% of its container.
- All this max width-based rule does is stipulate that all images should grow to be a maximum of 100% of their size. Where a containing element (such as the body or a div it sits within) is less than the full intrinsic width of the image, the image will simply scale up to display as large as it can within that constraint.

- If you omit `height: auto`, your image will likely become **distorted** (squashed) when it resizes.

### 1. The Behavior Breakdown

|**Setup**|**Behavior**|**Result**|
|---|---|---|
| **`max-width: 100%` ONLY**             | The width shrinks to fit the screen, but the **height stays fixed** at the original pixel size.  | **Distortion.** The image looks horizontally squashed or vertically stretched. |
| -------------------------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| **`max-width: 100%` + `height: auto`** | The width shrinks, and the browser calculates the new height to match the original aspect ratio. | **Proportional.** The image scales down perfectly.                             |

#### 2. Why does this happen?

It usually happens because of the HTML attributes. Best practice is to include `width` and `height` attributes in your HTML to prevent layout shifts:

```html
<img src="photo.jpg" width="800" height="400" alt="Scenery">
```

If you apply **only** `max-width: 100%` and the screen is only `400px` wide:

1. **CSS:** Overrides the width to `400px`.
    
2. **CSS:** Says nothing about height, so the browser falls back to the HTML attribute: `400px`.
    
3. **Result:** An image that is `400px` wide and `400px` tall. The aspect ratio is broken.
    

By adding `height: auto`, you explicitly tell the CSS to **ignore the HTML height attribute** and calculate a new height based on the current width.

#### 3. The Modern Alternative (`aspect-ratio`)

In modern CSS (2026), you might also see this, which achieves a similar goal but is often used for skeleton loading states:

```css
img {
  max-width: 100%;
  height: auto; /* Still the safest, easiest standard */
}
```

**Recommendation:** Always include `height: auto` in your CSS reset. It is the single most important rule for responsive images.

```css
/* The standard responsive image reset */
img {
  max-width: 100%;
  height: auto;
  display: block; /* Removes the tiny gap below inline images */
}
```
