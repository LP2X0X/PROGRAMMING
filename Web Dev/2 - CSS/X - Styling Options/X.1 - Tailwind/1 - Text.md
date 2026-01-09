---
tags: 
 - css
 - tailwind
 - flex
---

Here is the "80/20" overview for **Text & Typography** in Tailwind.

One big "Gotcha" to remember: **Tailwind removes all default styling.** An `<h1>` tag looks exactly like a `<p>` tag until you add classes to it.

---

## 1. Size & Weight (The Basics)

Control how big and how thick the text is.

| **Class**                  | **CSS Property** | **Use Case**                           |
| -------------------------- | ---------------- | -------------------------------------- |
| **`text-xs`**              | `0.75rem`        | Legal text, tiny labels.               |
| **`text-sm`**              | `0.875rem`       | Subtitles, meta-data, secondary text.  |
| **`text-base`**            | `1rem`           | **Default.** Standard body text.       |
| **`text-lg`**              | `1.125rem`       | Intro paragraphs, large buttons.       |
| **`text-xl`** to **`9xl`** | `1.25rem`+       | Headings.                              |
| **`font-normal`**          | `400`            | Default weight.                        |
| **`font-medium`**          | `500`            | Subtle emphasis (great for UI labels). |
| **`font-semibold`**        | `600`            | Section headers.                       |
| **`font-bold`**            | `700`            | Main titles.                           |

### The "Arbitrary Value"

Need exactly 13px?

- `text-[13px]`
    

---

## 2. Color & Opacity

The syntax is always `text-{color}-{shade}`.

- **`text-blue-600`**: Standard text color.
    
- **`text-white`**: White text.
    
- **`text-slate-500`**: The standard "muted" grey text.
    
- **`text-transparent`**: Makes text invisible (used often with background clips for gradients).
    
- **`text-blue-600/50`**: Sets the color to Blue 600 at **50% opacity**.
    

---

## 3. Alignment & Layout

|**Class**|**Use Case**|
|---|---|
|**`text-center`**|Centers text.|
|**`text-left`**|Left aligns (Default).|
|**`text-right`**|Right aligns.|
|**`truncate`**|**Essential.** Adds `...` if text is too long for one line.|
|**`break-words`**|Forces long words to break so they don't overflow the container.|

---

## 4. The "Designer" Touches (Spacing)

This is what separates a messy UI from a polished one.

### Line Height (`leading`)

Tailwind sets good defaults based on text size, but you often need to tweak it.

- **`leading-none`**: Line height = 1. Use for **Big Headings** so lines don't have huge gaps.
    
- **`leading-tight`**: Use for smaller headings.
    
- **`leading-relaxed`**: Use for **paragraphs** to make them easier to read.
    

### Letter Spacing (`tracking`)

- **`tracking-tight`**: Makes big headlines look tighter and more modern.
    
- **`tracking-wide`**: Use for **UPPERCASE** labels to make them legible.
    

---

## 5. Transformation & Decoration

- **`uppercase`**: TRANSFORMS TO ALL CAPS.
    
- **`capitalize`**: Capitalizes First Letter Of Each Word.
    
- **`lowercase`**: transforms to lower case.
    
- **`underline`**: Adds an underline.
    
- **`decoration-blue-500`**: Changes the color of the underline (great for links).
    

---

## ⚡ Cheat Sheet Examples

### The "Modern Heading"

Big, dark, tight spacing, with a specific font weight.

HTML

```
<h1 class="text-4xl font-bold tracking-tight text-slate-900 sm:text-6xl">
  Build faster.
</h1>
```

### The "Subtle Label"

Small, uppercase, wide spacing, muted color.

```html
<span class="text-xs font-medium uppercase tracking-wider text-slate-500">
  Category
</span>
```

### The "Truncated List Item"

Forces text to stay on one line and adds "..." if it's too long.

```html
<p class="w-full truncate text-sm text-slate-700">
  This is a very long description that will get cut off...
</p>
```

### The "Link"

Blue text, underlined only when you hover over it.

```html
<a href="#" class="text-blue-600 hover:underline hover:text-blue-800">
  Read more
</a>
```