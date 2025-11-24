---
tags: 
 - css
 - unit
 - relative
---

These are **new viewport height units** introduced to fix the long-standing mobile browser issues where the browser’s address bar expanding/collapsing changes the viewport height unexpectedly.

---

# 📌 **Motivation**

On mobile browsers (especially Safari):

- `100vh` changes when the browser bars appear/disappear.
    
- This causes layouts to “jump” or content to move.
    

So CSS introduced new dynamic viewport units in 2022–2023.

---

# 🔹 **1. `svh` — Small Viewport Height**

### **Meaning**

`1svh` = 1% of the **smallest possible** viewport height  
(i.e., when the browser UI bars take up the MOST space)

### In practice:

- Represents the **worst-case height**
    
- Stable and predictable
    
- Does **not** expand when the UI disappears
    

### Use When:

- You want **zero layout jump**
    
- You want to ensure the content always fits
    
- Safe areas, bottom sheets, fixed headers/footers
    

### Example:

```css
.panel {
  height: 100svh;
}
```

A panel that **never grows** when the bars collapse.

---

# 🔹 **2. `lvh` — Large Viewport Height**

### **Meaning**

`1lvh` = 1% of the **largest possible** viewport height  
(i.e., when the browser UI bars are fully hidden)

### In practice:

- Represents the **best-case height**
    
- Expands fully when the user scrolls down and UI hides
    

### Use When:

- You want maximum available space
    
- Hero sections
    
- Full-page layouts optimized for immersive display
    

### Example:

```css
.hero {
  height: 100lvh;
}
```

The hero expands to the **full height when UI disappears**.

---

# 🔹 **Comparison Table**

|Unit|Meaning|Changes when browser bars hide?|Best use|
|---|---|---|---|
|`vh`|current viewport height|YES (everything jumps)|Legacy / desktop|
|`svh`|smallest viewport height|NO|Stable UI elements|
|`lvh`|largest viewport height|YES|Fullscreen areas|
|`dvh`|dynamic viewport height|YES (updates smoothly)|Most apps|

---

# 🔹 **Visual Mental Model**

### Mobile Browser

- Browser UI expanded → shorter viewport → `svh`
    
- Browser UI collapsed → taller viewport → `lvh`
    

```
svh  <  dvh <  vh (varies)  <  lvh
```

`dvh` = live, dynamic  
`s vh` = stiff (never grows)  
`l vh` = full height (max possible)

---

# 🧪 **Which one should you use?**

### Use `dvh` most of the time:

```css
min-height: 100dvh;
```

### Use `svh` for **fixed, stable UI** like:

- navbars
    
- bottom bars
    
- overlays that must never exceed the smallest screen
    

### Use `lvh` for **fullscreen experiences** like:

- hero sections
    
- image backgrounds
    
- card stacks
    
- games
    

---

## References

https://www.youtube.com/watch?v=7judyqwqmKo&t=254s