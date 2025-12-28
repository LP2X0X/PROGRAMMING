---
tags: 
 - css
 - tailwind
 - layout
 - grid
---

Here is the "80/20" overview for CSS Grid in Tailwind.

While **Flexbox** is for 1-dimensional layouts (a row of buttons, a navbar), **Grid** is for 2-dimensional layouts (galleries, entire page structures, dashboard grids).

---

## 1. The Container (The Parent)

Define the grid wrapper and how many columns it usually has.

| **Class**          | **Meaning**      | **Use Case**                                                                     |
| ------------------ | ---------------- | -------------------------------------------------------------------------------- |
| **`grid`**         | `display: grid`  | **Required.** Turns the div into a grid container.                               |
| **`grid-cols-1`**  | 1 Column         | Good mobile default. Items stack vertically.                                     |
| **`grid-cols-3`**  | 3 Equal Columns  | Creates a standard 3-column layout. Supports up to 12.                           |
| **`grid-cols-12`** | 12 Equal Columns | Use this for complex, bootstrap-style layouts where items span different widths. |
| **`gap-4`**        | `gap: 1rem`      | Creates space between _both_ rows and columns.                                   |

### The "Arbitrary Value" (Power Move)

Sometimes you don't want equal columns. Use brackets `[]` to define specific widths (like CSS `grid-template-columns`).

- **`grid-cols-[200px_1fr]`**: A fixed 200px sidebar on the left, and the rest (`1fr`) takes up the remaining space.
    
- **`grid-cols-[auto_1fr_auto]`**: Content determines width on sides, middle takes the rest.
    

---

## 2. The Items (The Children)

By default, every child takes up **1 cell**. You use these classes to make them bigger.

| **Class**           | **Meaning**        | **Use Case**                                                    |
| ------------------- | ------------------ | --------------------------------------------------------------- |
| **`col-span-2`**    | Span 2 columns     | Make an item wide (e.g., a featured blog post).                 |
| **`col-span-full`** | Span _all_ columns | Great for headers or footers inside a grid.                     |
| **`row-span-2`**    | Span 2 rows        | Make an item tall (vertical).                                   |
| **`col-start-2`**   | Start at line 2    | Force an item to skip the first column and start in the second. |

---

## 3. Alignment

Grid uses the same alignment logic as Flexbox, but defaults to "stretch."

- **`items-center`**: Vertically centers content within its grid cell.
    
- **`justify-center`**: Horizontally centers content within its grid cell.
    
- **`place-items-center`**: Centers in **both** directions at once (shorthand).
    

---

## ⚡ Cheat Sheet Examples

### 1. The "Responsive Gallery" (The #1 Use Case)

This is the most common pattern. 1 column on mobile, 2 on tablet, 3 on desktop. No media queries needed.

```html
<div class="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
  <div class="bg-blue-100">Card 1</div>
  <div class="bg-blue-100">Card 2</div>
  <div class="bg-blue-100">Card 3</div>
  </div>
```

### 2. The "Bento Box" / Dashboard Layout

A complex layout where some items are wide and some are tall.

```html
<div class="grid grid-cols-3 gap-4">
  <div class="col-span-2 h-32 bg-green-200">Stats Graph</div>
  
  <div class="h-32 bg-green-200">User Profile</div>
  
  <div class="row-span-2 bg-green-200">Sidebar / Feed</div>
  
  <div class="col-span-2 h-32 bg-green-200">Recent Activity</div>
</div>
```

### 3. The "Holy Grail" Layout (Sidebar + Main)

Using arbitrary values for a fixed sidebar.

```html
<div class="grid h-screen grid-cols-[250px_1fr]">
  <div class="bg-slate-800 text-white">Fixed Sidebar</div>
  <div class="bg-slate-100 p-8">Fluid Main Content</div>
</div>
```

### Summary: Flex vs. Grid?

- **Use Flex** for a single row or column (Navbars, centering a modal, aligning an icon next to text).
    
- **Use Grid** when you need to control both rows and columns at the same time (Image galleries, page skeletons, dashboard widgets).