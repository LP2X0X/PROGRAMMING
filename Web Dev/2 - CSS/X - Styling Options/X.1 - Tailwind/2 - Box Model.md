---
tags: 
 - css
 - tailwind
 - box
---

Here is the **"80/20" overview** for the Box Model in Tailwind.

The Box Model controls the size and spacing of every element on your page.

---

## 0. The "Reset" (Important!)

In standard CSS, the default is box-sizing: border-box.

In Tailwind, the default is border-box.

**What this means:** If you set `w-64` and `p-4`, the _total_ width stays 64 units. The padding pushes _inward_, not outward. You don't have to do math.

---

## 1. Spacing (Padding & Margin)

This is the most used part of Tailwind. The scale usually works in multiples of **4px** (0.25rem).

- `1` = 4px
    
- `4` = 16px (standard)
    
- `8` = 32px
    

### Padding (Inside the box)

- **`p-4`**: Padding on all sides.
    
- **`px-4`**: Left & Right only (x-axis).
    
- **`py-4`**: Top & Bottom only (y-axis).
    
- **`pt-4`** / **`pr-4`** / **`pb-4`** / **`pl-4`**: Specific sides (Top, Right, Bottom, Left).
    

### Margin (Outside the box)

- **`m-4`**: Margin on all sides.
    
- **`mx-4`**: Left & Right.
    
- **`my-4`**: Top & Bottom.
    
- **`mt-4`** / **`mr-4`** / **`mb-4`** / **`ml-4`**: Specific sides.
    
- **`-mt-4`**: **Negative Margin.** Pulls the element upwards (useful for overlapping UI).
    
- **`mx-auto`**: Centers a block element horizontally (if it has a width).
    

---

## 2. Sizing (Width & Height)

### Width (`w-`)

- **Fixed:** `w-1` to `w-96` (follows the spacing scale).
    
- **Percentage:** `w-1/2` (50%), `w-full` (100%).
    
- **Viewport:** `w-screen` (100vw - typically causes scrollbars, use with caution).
    
- **Arbitrary:** `w-[500px]`.
    

### Height (`h-`)

- **Fixed:** `h-1` to `h-96`.
    
- **Full:** `h-full` (100% of _parent_ height). **Note:** Parent must have a defined height for this to work.
    
- **Screen:** `h-screen` (100vh - fills the visible window).
    

### Constraints (Min/Max)

- **`max-w-sm`** / **`md`** / **`lg`** / **`xl`**: Restricts width (great for readable text blocks).
    
- **`min-h-screen`**: Forces the element to be _at least_ the height of the screen (great for Page Layouts/Wrappers).
    

---

## 3. Borders

To make a border visible, you usually need three things: **Width, Color, and Radius.**

### Border Width

- **`border`**: 1px border (default).
    
- **`border-0`**: No border.
    
- **`border-2`**, **`border-4`**, **`border-8`**: Thicker borders.
    
- **`border-t-2`**: 2px border on Top only.
    

### Border Color

- **`border-gray-200`**: Standard light border.
    
- **`border-transparent`**: Keeps the border width but makes it invisible (prevents layout shifts on hover).
    

### Border Radius (`rounded`)

- **`rounded`**: Small border radius (4px).
    
- **`rounded-md`**: Medium (6px) - Standard for buttons/cards.
    
- **`rounded-lg`** / **`rounded-xl`**: Larger, softer corners.
    
- **`rounded-full`**: Makes a square into a **Circle**, or a rectangle into a **Pill**.
    

---

## 4. The "Divide" Trick

If you have a list of items and want a border _between_ them (but not on the outside), don't use `border-b` on every item. Use `divide-` on the **parent container**.

```html
<ul class="divide-y divide-gray-200">
  <li>Item 1</li>
  <li>Item 2</li> <li>Item 3</li>
</ul>
```

---

## ⚡ Cheat Sheet Example

### The "Standard Card"

This combines almost every box model concept effectively.

```html
<div class="w-full max-w-sm mx-auto rounded-xl border border-gray-200 p-6">
  <h2 class="mb-2 text-xl font-bold">Card Title</h2>
  <p class="text-gray-600">Card content goes here.</p>
</div>
```