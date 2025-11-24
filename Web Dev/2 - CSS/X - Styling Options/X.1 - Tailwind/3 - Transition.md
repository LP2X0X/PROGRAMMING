---
tags: 
 - css
 - tailwind
 - transition
---

Here is the "80/20" overview for **Transitions & Animation** in Tailwind.

To make an element animate, you generally need **three ingredients** on the same element:

1. **The Property:** What to animate (e.g., `transition-colors`).
    
2. **The Duration:** How long it takes (e.g., `duration-300`).
    
3. **The Ease:** How it moves (e.g., `ease-in-out`).
    

---

## 1. The Activator (The Property)

You must add one of these to tell the browser _what_ to watch for changes.

|**Class**|**CSS Property**|**Use Case**|
|---|---|---|
|**`transition`**|`background`, `color`, `border`, `transform`, `opacity`, `shadow`|**Default.** Covers 95% of use cases. Use this unless you need to be specific.|
|**`transition-all`**|`all`|Animates _everything_. Can be performance heavy, but useful if you are changing padding or margins.|
|**`transition-colors`**|`background-color`, `border-color`, `color`|Great for buttons and links where only color changes.|
|**`transition-transform`**|`transform`|Use for movement, rotation, or scaling.|
|**`transition-opacity`**|`opacity`|Use for fading elements in/out.|

---

## 2. The Speed (Duration)

How fast does the change happen? The number represents milliseconds.

|**Class**|**Speed**|**Feel**|
|---|---|---|
|**`duration-75`**|Instant|Micro-interactions (active clicks).|
|**`duration-150`**|Fast|Standard hover states (buttons).|
|**`duration-300`**|Normal|Larger movements (modals, dropdowns).|
|**`duration-500`**|Slow|"Cinematic" reveals or large layouts shifting.|
|**`duration-1000`**|Very Slow|Loading bars or slow fades.|

---

## 3. The Feel (Ease)

The "curve" of the animation.

|**Class**|**Effect**|**Best For...**|
|---|---|---|
|**`ease-in-out`**|Slow start, slow end.|**Default.** Good for almost everything.|
|**`ease-out`**|Fast start, slow end.|**Entering.** Things appearing on screen (modals, menus). It feels "responsive" immediately.|
|**`ease-in`**|Slow start, fast end.|**Exiting.** Things leaving the screen. They slowly start moving then "get out of the way."|
|**`ease-linear`**|Constant speed.|Loading spinners or continuous loops.|

---

## 4. Common Transform Effects

Transitions are useless without changing something. These are the most common properties you will animate.

- **`scale-105`**: Zooms the element to 105% size.
    
- **`rotate-180`**: Rotates 180 degrees (upside down).
    
- **`-translate-y-1`**: Moves the element UP slightly.
    
- **`opacity-0`** / **`opacity-100`**: Invisible / Visible.
    

---

## ⚡ Cheat Sheet Examples

### 1. The "Smooth Button" (Color Change)

The standard button hover.

```html
<button class="bg-blue-500 px-4 py-2 text-white transition duration-300 hover:bg-blue-700">
  Hover Me
</button>
```

### 2. The "Interactive Card" (Lift & Shadow)

When hovered, the card floats up slightly and the shadow gets deeper.

```html
<div class="shadow-md transition duration-500 hover:-translate-y-1 hover:shadow-xl">
  Card Content
</div>
```

### 3. The "Dropdown Arrow" (Rotation)

Common for FAQs or Accordions.

```html
<svg class="h-5 w-5 transition-transform duration-300 aria-expanded:rotate-180" ... >
  </svg>
```

### 4. The "Fade In" (Opacity)

Useful for modals or overlays.

```html
<div class="opacity-0 transition-opacity duration-700 ease-out data-[visible=true]:opacity-100">
  I fade in slowly
</div>
```

### 💡 Pro Tip: Where to put the class?

Always put the `transition` and `duration` classes on the **base state**, not the `hover:` state.

- ✅ **Correct:** `<div class="transition hover:opacity-50">` (Animates on mouse in **and** mouse out).
    
- ❌ **Wrong:** `<div class="hover:transition hover:opacity-50">` (Animates mouse in, but snaps abruptly on mouse out).