---
tags: 
 - css
 - tailwind
 - position
---

Here is the "80/20" overview for **Positioning** in Tailwind.

Positioning allows you to take elements out of the normal document flow and stick them exactly where you want (corners, fixed headers, overlays).

---

## 1. The Position Types (The Behavior)

You must apply one of these classes to change how the element behaves.

|**Class**|**CSS Property**|**The "Plain English" Explanation**|
|---|---|---|
|**`static`**|`static`|**Default.** The element sits where it normally would. Use this to _reset_ a position on mobile vs desktop.|
|**`relative`**|`relative`|**The Anchor.** It sits normally, but acts as a reference point for any `absolute` children inside it.|
|**`absolute`**|`absolute`|**The Free Spirit.** It ignores other elements and floats freely. It positions itself relative to the nearest _non-static_ parent (usually `relative`).|
|**`fixed`**|`fixed`|**The Sticker.** Sticks to the **screen (viewport)**. It stays in place even when you scroll the page (Navbars, Modals).|
|**`sticky`**|`sticky`|**The Hybrid.** Acts like normal, but "sticks" to the edge of the screen once you scroll past it.|

---

## 2. Placement (The Coordinates)

Once an element is `absolute`, `fixed`, or `sticky`, you need to tell it _where_ to go using Top, Right, Bottom, Left.

### The "Inset" Shorthand (Best Feature)

Instead of writing `top-0 right-0 bottom-0 left-0`, use:

- **`inset-0`**: Anchors the element to **all four edges**. (Perfect for modal backdrops or full-cover images).
    
- **`inset-x-0`**: Anchors to Left and Right only.
    
- **`inset-y-0`**: Anchors to Top and Bottom only.
    

### Specific Edges

- **`top-0`**, **`right-4`**, **`bottom-10`**, **`left-1/2`**.
    
- **`-top-4`**: Negative values pull the element _outside_ the box (great for notification badges).
    

---

## 3. Stacking Order (Z-Index)

If elements overlap, `z-index` decides which one sits on top.

- **`z-0`**: Default.
    
- **`z-10`**, **`z-20`**, **`z-30`**, **`z-40`**, **`z-50`**: The scale increases by 10.
    
- **`z-auto`**: Resets the value.
    
- **Arbitrary:** `z-[100]` (for when you really need to be on top).
    

---

## ⚡ Cheat Sheet Patterns

### 1. The "Notification Badge" (Relative + Absolute)

This is the classic pattern: The parent (Icon) is `relative` so the child (Red Dot) knows where to sit.

```html
<button class="relative p-2 bg-gray-200 rounded">
  Icon
  
  <div class="absolute -top-1 -right-1 h-3 w-3 rounded-full bg-red-500"></div>
</button>
```

### 2. The "Modal Overlay" (Fixed + Inset)

Covers the entire screen, regardless of scroll position.

```html
<div class="fixed inset-0 z-50 flex items-center justify-center bg-black/50">
  
  <div class="bg-white p-8 rounded-lg shadow-xl">
    Are you sure?
  </div>
  
</div>
```

### 3. The "Sticky Header"

Stays at the top of the list/page as you scroll down.

Note: sticky requires a defined coordinate (usually top-0) to work.

```html
<div class="h-screen overflow-y-auto">
  <header class="sticky top-0 z-10 bg-white shadow-sm p-4">
    I stick to the top!
  </header>
  
  <main class="p-4">
    </main>
</div>
```

### 4. The "Dead Center" (The Old Way)

Before Flexbox/Grid, this was how we centered things. You might still see it.

```html
<div class="relative h-64 bg-gray-100">
  <div class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2">
    Centered
  </div>
</div>
```