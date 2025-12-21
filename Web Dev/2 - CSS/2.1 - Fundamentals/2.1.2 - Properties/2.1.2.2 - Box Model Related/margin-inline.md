---
tags: 
 - css
 - property
 - box
---

The **`margin-inline`** property is a **logical CSS shorthand** that controls horizontal margins **relative to writing direction**, not physical left/right.

---

## What it does

```css
margin-inline: 16px;
```

Equivalent to:

```css
margin-inline-start: 16px;
margin-inline-end: 16px;
```

It applies margins on the **inline axis**:

- Left / right in LTR languages
    
- Right / left in RTL languages
    

---

## Why it exists (the point)

It makes layouts **direction-aware** automatically.

Instead of:

```css
margin-left: 16px;
margin-right: 16px;
```

You write:

```css
margin-inline: 16px;
```

Now the layout works correctly for:

- English (LTR)
    
- Arabic / Hebrew (RTL)
    
- Vertical writing modes
    

---

## Common use cases

### 1. Direction-agnostic spacing

```css
.card {
  margin-inline: auto;
}
```

→ centers the element in any writing direction

---

### 2. Responsive UI / internationalization

```css
.sidebar {
  margin-inline-start: 1rem;
}
```

Automatically flips in RTL layouts.

---

### 3. Cleaner shorthand

```css
margin-inline: 8px 24px;
```

Equivalent to:

```css
margin-inline-start: 8px;
margin-inline-end: 24px;
```

---

## When to use vs not use

✔ Use `margin-inline` for **layout spacing**  
✘ Avoid for **physical positioning** (icons, sprites)

---

## Summary

- `margin-inline` = logical left/right margin
    
- Respects LTR / RTL automatically
    
- Replaces `margin-left` / `margin-right`
    
- Best for modern, international-ready layouts
    

That’s the essence.

----

### Side notes

We have `margin-left` and `margin-right` because with the `margin` shorthand, you need to override all values. Sometimes you don't want to override them all.

For why we got `margin-inline`: that respects the direction of the language of the page.

So, `margin-left: 10px`is always on the left side, even if the language is Right-to-Left. Whereas `margin-inline: 10px auto;` is _at the start of the line_. Which is on the right in Right-to-Left languages.

This might make it clear: [https://jsfiddle.net/bfmehvnu/14/](https://jsfiddle.net/bfmehvnu/14/)