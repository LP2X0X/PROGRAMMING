---
tags: 
 - css
 - tailwind
 - technique
---

## **What “divide” actually does**

The `divide-*` utilities apply **borders _between_ child elements** of a container **without adding borders to the outside edges**.

Tailwind does this using a clever selector:

```css
> :not([hidden]) ~ :not([hidden]) {
  border-top-width: 1px;
}
```

This means:

- Only **the elements that follow another element** get a border
    
- The **first child gets no border**
    
- No border around the container itself
    
- No need to manually add borders inside each child
    

---

# **Why use divide instead of borders?**

### ✔️ Cleaner & fewer classes

Instead of doing:

```html
<li class="border-b last:border-none">
<li class="border-b last:border-none">
<li class="border-b last:border-none">
```

You just do:

```html
<ul class="divide-y">
```

No need for extra utilities on each item.

---

### ✔️ Prevents "double borders"

If you add `border-b` to each item, you sometimes get _2px borders_ between items (one bottom border + one top border).

`divide-y` guarantees **only one border** per gap.

---

### ✔️ Automatically ignores hidden elements

`divide-*` uses `:not([hidden])`, so hidden items don’t leave weird double borders or blank spaces.

---

### ✔️ Works with vertical or horizontal layouts

- `divide-y` → put borders **between rows**
    
- `divide-x` → put borders **between columns**
    

Example:

```html
<div class="flex divide-x divide-gray-300">
  <button>Left</button>
  <button>Right</button>
</div>
```

---

# **How to use divide utilities**

## **1. Basic example**

```html
<ul class="divide-y divide-gray-200">
  <li>Item 1</li>
  <li>Item 2</li>
  <li>Item 3</li>
</ul>
```

This creates:

```
Item 1
──────
Item 2
──────
Item 3
```

But **no border on the top or bottom** of the list.

---

# **2. Thickness utilities**

```html
<ul class="divide-y divide-y-2 divide-gray-300">
```

- `divide-y-2` → 2px thick borders
    

---

# **3. Dashed, dotted, double**

```html
<ul class="divide-y divide-dashed">
<ul class="divide-y divide-dotted">
<ul class="divide-y divide-double">
```

---

# **4. Colors**

```html
<ul class="divide-y divide-gray-300">
```

Any color from your theme works.

---

# **5. Horizontal dividing**

```html
<div class="flex divide-x divide-gray-400">
  <div>A</div>
  <div>B</div>
  <div>C</div>
</div>
```

Adds vertical borders _between_ flex items.

---

# **6. Combining with spacing**

```html
<ul class="space-y-4 divide-y divide-gray-300">
```

- `space-y-4` → vertical gaps
    
- `divide-y` → borders in gaps
    

Everything stays clean and consistent.

---

# **7. Using divide with grid layouts**

Yes, it works with grids too:

```html
<div class="grid grid-cols-3 divide-x divide-gray-200">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>
```

---

# **How Tailwind implements divide (under the hood)**

Just for understanding:

```css
.divide-y > :not([hidden]) ~ :not([hidden]) {
  border-top-width: 1px;
}
```

Meaning:

- The 2nd, 3rd, 4th… child get borders
    
- The 1st one never does
    
- Cleaner than manually applying border rules
    

---

# **Summary**

### ✔️ **Divide utilities** let you add borders **between items only**, with:

- No outer borders
    
- No double borders
    
- No extra classes on each child
    
- Automatic handling of hidden elements
    
- Support for direction (`divide-x`, `divide-y`)
    
- Support for thickness, style, color
    

### ✔️ **Best use cases**

- Lists
    
- Menus
    
- Stacked cards
    
- Flex rows/columns
    
- Grid items
    
- Button groups
    