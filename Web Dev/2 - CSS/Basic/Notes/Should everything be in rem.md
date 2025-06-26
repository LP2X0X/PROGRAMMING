---
tags: css, note, fundamental
---

## ✅ Should everything be in `rem`?

### 👉 **Short answer:**

**No**, not _everything_ should be in `rem`.  
But **most things benefit from being in `rem`**, especially **font sizes, spacing, and layout units** — _with exceptions_.

---

## 🧱 Where `rem` is best (✅ recommended):

|Property|Use `rem`?|Why|
|---|---|---|
|`font-size`|✅ Yes|Scales with root font size (accessibility + responsiveness)|
|`margin`, `padding`|✅ Yes|Keeps spacing proportional to text size|
|`gap`, `line-height`|✅ Yes|Consistent vertical rhythm|
|`max-width`, `width` (for text blocks)|✅ Often|Keeps layouts text-size-aware|

---

## ❌ Where `px` or others are better (Exceptions):

|Property|Prefer `px` or other unit|Why|
|---|---|---|
|`border-width`|✅ `px`|You usually want crisp, consistent borders (e.g. `1px`)|
|`box-shadow`, `text-shadow`|✅ `px`|Pixel precision is important for effects|
|`media queries`|✅ `px` or `em`|Device widths are in pixels (`min-width: 768px`)|
|`images`, icons|✅ `px` or `%`|You may want exact control|
|`transform`, `translate`, `animation steps`|✅ `px`|Often tied to physical distances on screen|

---

## 🧠 Pro tip:

- Use `rem` for **scaling** and **text-aware spacing**
    
- Use `px` for **pixel-perfect control** (borders, shadows, etc.)
    

---

## 🧪 Real-world mix example:

```css
.card {
  font-size: 1rem; /* =16px */
  padding: 1.5rem; /* =24px */
  border-radius: 8px; /* px gives crisp curve */
  border: 1px solid #ccc;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}
```

---

## ✅ Summary: Best Practice

|Use case|Unit|
|---|---|
|Text, spacing, layout|`rem`|
|Borders, shadows|`px`|
|Media queries|`px` or `em`|
|Animations, transforms|`px`|
