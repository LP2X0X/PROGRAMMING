---
tags: 
 - css
 - note
 - value
 - property
---

# ✅ **CSS Initial Values You Should Know**

## 🧱 **Layout & Box Model**

| Property     | Initial value | Why it matters                                    |
| ------------ | ------------- | ------------------------------------------------- |
| `display`    | `inline`      | Many elements collapse if you reset this blindly. |
| `box-sizing` | `content-box` | Content-box often causes unexpected sizing.       |
| `position`   | `static`      | Removes top/left/right/bottom effect.             |
| `float`      | `none`        | Resets any floating.                              |
| `clear`      | `none`        | No clearing by default.                           |
| `overflow`   | `visible`     | Children can spill out.                           |
| `z-index`    | `auto`        | No stacking context.                              |
| `visibility` | `visible`     | Element is shown.                                 |
| `opacity`    | `1`           | Fully opaque.                                     |

---

## 📏 **Spacing**

| Property         | Initial    | Notes                                    |
| ---------------- | ---------- | ---------------------------------------- |
| `margin`         | `0`        | Inline elements ignore vertical margins. |
| `padding`        | `0`        | Important when styling components.       |
| `border`         | `none`     | No border unless specified.              |
| `vertical-align` | `baseline` | Affects inline elements alignment.       |

---

## ✍️ **Typography**

| Property          | Initial                              | Notes                                       |
| ----------------- | ------------------------------------ | ------------------------------------------- |
| `font-size`       | `medium` (browser dependent)         | Usually ~16px.                              |
| `font-weight`     | `400`                                | Normal weight.                              |
| `font-style`      | `normal`                             | No italics.                                 |
| `line-height`     | `normal`                             | Browser-defined calculation.                |
| `font-family`     | Depends on UA                        | Usually a system serif or sans-serif.       |
| `text-align`      | `start` or `left`                    | Based on writing mode.                      |
| `text-decoration` | `none`                               | Links override via UA stylesheet, not spec. |
| `color`           | `black` (spec: user-agent dependent) | Typically black.                            |
| `white-space`     | `normal`                             | Collapses spaces and wraps lines.           |

---

## 🎨 **Background & Visual**

| Property              | Initial       | Notes                                      |
| --------------------- | ------------- | ------------------------------------------ |
| `background-color`    | `transparent` | Important when debugging invisible layers. |
| `background-repeat`   | `repeat`      | Background images tile by default.         |
| `background-position` | `0% 0%`       | Starts at top-left.                        | 
| `background-size`     | `auto`        | Image keeps its natural size.              |
| `box-shadow`          | `none`        | No shadow.                                 |
| `border-radius`       | `0`           | Square corners.                            |

---

## 🧩 **Flex & Grid**

### Flexbox

| Property          | Initial              | Notes                        |
| ----------------- | -------------------- | ---------------------------- |
| `display`         | `inline` (not flex!) | Must set `display: flex`.    |
| `flex-direction`  | `row`                | Horizontal by default.       | 
| `flex-wrap`       | `nowrap`             | Items don’t wrap.            |
| `justify-content` | `flex-start`         | Items grouped left.          |
| `align-items`     | `stretch`            | Children stretch vertically. |
| `gap`             | `0`                  | No spacing between children. |

### Grid

| Property                | Initial | Notes                 |     |
| ----------------------- | ------- | --------------------- | --- |
| `grid-template-columns` | `none`  | No grid defined.      |     |
| `grid-template-rows`    | `none`  | Must define.          |     |
| `gap`                   | `0`     | Consistent with flex. |     | 

---

# 🧠 **If you’re going to memorize only a few initial values, memorize these**

Because they are the most dangerous when using `initial`:

### **1. `display: inline`**

Most layout bugs come from this.

### **2. `box-sizing: content-box`**

You probably want `border-box`.

### **3. `font-size: medium` (≈16px)**

Inline resets bump text size unexpectedly.

### **4. `line-height: normal`**

Can cause vertical alignment issues.

### **5. `background-repeat: repeat`**

Images tile unexpectedly.

### **6. `flex-direction: row` & `flex-wrap: nowrap`**

When turning something into a flex container, always set these explicitly.