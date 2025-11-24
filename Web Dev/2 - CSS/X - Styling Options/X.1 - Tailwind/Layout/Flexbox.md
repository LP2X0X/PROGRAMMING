---
tags: 
 - css
 - tailwind
 - layout
 - flex
---

Here is the "80/20" rule overview of Flexbox in Tailwind—the 20% of classes you will use 80% of the time.

---

## 1. The Container (The Parent)

First, you must turn it on.

|**Class**|**CSS Property**|**Use Case**|
|---|---|---|
|**`flex`**|`display: flex`|**Required.** Turns the `div` into a flex container.|
|**`inline-flex`**|`display: inline-flex`|Like `flex`, but the container itself sits inline with text.|

### Direction (The Axis)

This defines your "Main Axis".

|**Class**|**CSS Equivalent**|**Visual Result**|
|---|---|---|
|**`flex-row`**|`row`|➡️ Left to Right (Default).|
|**`flex-col`**|`column`|⬇️ Top to Bottom (Stacking).|
|**`flex-wrap`**|`wrap`|Allow items to break onto a new line if they run out of space.|

---

## 2. Alignment (The "Centering" Stuff)

This is where most people get stuck. Remember:

- **`justify-`** controls the **Main Axis** (defined by `flex-row` or `flex-col`).
    
- **`items-`** controls the **Cross Axis** (the opposite direction).
    

### Justify (Main Axis)

_If `flex-row`, this moves items Left/Right. If `flex-col`, this moves items Up/Down._

- **`justify-start`**: Pack to the start (Default).
    
- **`justify-center`**: Pack to the center.
    
- **`justify-end`**: Pack to the end.
    
- **`justify-between`**: Push first item to start, last item to end, equal space between.
    
- **`justify-around`**: Equal breathing room around all items.
    

### Items (Cross Axis)

_If `flex-row`, this aligns Top/Bottom. If `flex-col`, this aligns Left/Right._

- **`items-start`**: Align to the top/start.
    
- **`items-center`**: Center along the cross line.
    
- **`items-end`**: Align to the bottom/end.
    
- **`items-stretch`**: Stretch to fill the container (Default, unless height is set).
    

### Gap (Spacing)

Stop using margins (`mr-4`) between items. Use gap.

- **`gap-x-4`**: Horizontal space.
    
- **`gap-y-4`**: Vertical space.
    
- **`gap-4`**: Space in both directions.
    

---

## 3. The Items (The Children)

Control how individual children behave inside the parent.

|**Class**|**Meaning**|**Use Case**|
|---|---|---|
|**`flex-1`**|"Take up all remaining space"|Great for the middle content of a navbar or the main body of a page layout.|
|**`flex-none`**|"Don't grow, don't shrink"|Keeps an item strictly at its defined size (e.g., a fixed-width sidebar).|
|**`grow`**|`flex-grow: 1`|Allows item to expand to fill space.|
|**`shrink-0`**|`flex-shrink: 0`|**Crucial.** Prevents an item (like an icon or image) from getting squashed when space is tight.|

---

## ⚡ Cheat Sheet Examples

### The "Perfect Center"

Centers content vertically and horizontally.

```html
<div class="flex h-screen items-center justify-center">
  <div>I am centered</div>
</div>
```

### The "Navbar"

Logo on the left, Links on the right.

```html
<div class="flex items-center justify-between p-4">
  <div class="logo">Logo</div>
  <div class="links">Login</div>
</div>
```

### The "Sidebar Layout"

Fixed sidebar, fluid content area.

```html
<div class="flex h-screen">
  <div class="w-64 flex-none bg-gray-200">Sidebar</div>
  
  <div class="flex-1 bg-white">Main Content</div>
</div>
```

### The "Card with Icon"

Icon stays round and doesn't squish; text flows next to it.

```html
<div class="flex gap-4">
  <div class="h-10 w-10 shrink-0 rounded-full bg-blue-500"></div>
  
  <div>
    <h3>Title</h3>
    <p>Long description text that might wrap...</p>
  </div>
</div>
```