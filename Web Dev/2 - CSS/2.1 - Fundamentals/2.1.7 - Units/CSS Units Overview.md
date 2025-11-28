---
tags:
  - css
  - note
  - summary
  - unit
---

```ad-tip
- Use `rem` for **global layout and typography**, and `em` for **local/component-based adjustments**.
    
- Use rems for font sizes, pixels for borders, and ems or rems for most other measures, especially paddings, margins, and border radius.
	- This way, font sizes are predictable, but you’ll still get the power of ems scaling your padding and margins should other factors alter the font size of an element.
	- Pixels make sense for borders, particularly when you want a nice fine line.
```

## **✅ 1. Absolute Units**

These are fixed and don’t change with screen size or parent element.

|**Unit**|**Meaning**|**Example**|
|---|---|---|
|px|Pixels|width: 100px;|
|pt|Points (1pt = 1/72 inch)|Used in print media|
|pc|Picas (1pc = 12pt)|Rarely used|
|in|Inches|width: 1in; = 96px|
|cm|Centimeters|width: 2.54cm; = 1in|
|mm|Millimeters|width: 10mm; = 1cm|

> 🟢 **Use px most often for screens.** Other units are useful for print CSS.

---

## **✅ 2. Relative to Font Size**

These units scale based on font size, either of the element or its parent.

|**Unit**|**Meaning**|**Example**|
|---|---|---|
|em|Relative to the current element’s font size|font-size: 2em; (2× current font size)|
|rem|Relative to the root (\<html>) font size|font-size: 1.5rem;|
|ex|x-height of current font (height of lowercase “x”)|Rare use|
|ch|Width of the “0” character in the font|Good for monospace alignment|

> 🟡 **rem** is generally preferred for consistent scaling across the app.

---

## **✅ 3. Relative to Viewport**

Scale based on screen (viewport) size. Great for responsive design.

|**Unit**|**Meaning**|**Example**|
|---|---|---|
|vw|% of viewport width|width: 50vw; = 50% of screen width|
|vh|% of viewport height|height: 100vh; = full screen height|
|vmin|Smaller of vw or vh|For square-ish scaling|
|vmax|Larger of vw or vh|For wide/tall responsiveness|

> 🔵 Use for **fullscreen layouts**, **text scaling**, or **hero sections**.

---

## **✅ 4. Percentage (%)**

Relative to the **parent element**.

|**Unit**|**Meaning**|**Example**|
|---|---|---|
|%|% of parent|width: 50%; = 50% of parent width|

> 🟠 Useful for fluid layouts, grid/flex children, and paddings.

---

## **✅ 5. Time Units**

Used in animations and transitions.

|**Unit**|**Meaning**|**Example**|
|---|---|---|
|s|Seconds|transition: 0.5s;|
|ms|Milliseconds|animation-duration: 300ms;|

---

## **✅ 6. Angle Units**

Used for rotation, gradients, etc.

|**Unit**|**Meaning**|**Example**|
|---|---|---|
|deg|Degrees|transform: rotate(45deg);|
|rad|Radians|rotate(0.785rad);|
|grad|Gradians (400grad = full circle)|Rare|
|turn|Turns (1turn = 360deg)|rotate(0.5turn); = 180°|

---

## **✅ 7. Resolution Units**

Used in media queries and printing.

|**Unit**|**Meaning**|**Example**|
|---|---|---|
|dpi|Dots per inch|min-resolution: 300dpi;|
|dpcm|Dots per centimeter||
|dppx|Dots per pixel|1dppx = 96dpi|

---

## **🧠 Summary Table for Usage:**

|**Use Case**|**Best Units**|
|---|---|
|Screen layout|px, %, vw, vh|
|Typography|rem, em|
|Responsive design|vw, vh, %, rem|
|Animations|s, ms|
|Rotation/Angles|deg, turn|
|Printing|in, cm, pt, dpi|

---

## **✅ Tips**

- Use rem for consistent sizing across components.
- Use % or vw/vh for fluid layouts.
- Avoid absolute units in responsive web design (except for px).
- Use em inside components when you want to scale relatively to their container.