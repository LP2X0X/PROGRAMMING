---
tags: 
 - css
 - unit
 - relative
---

# **CSS `vmin` & `vmax` Overview**

## **1. What They Are**

`vmin` and `vmax` are **viewport-relative units**.  
They scale based on the **smaller or larger** dimension of the viewport.

- The viewport = the visible browser window.
    

---

## **2. Definitions**

### **`1vmin`**

= **1% of the smaller side** of the viewport  
(smaller of width or height)

If viewport is 1200px × 800px →  
`1vmin = 1% of 800px = 8px`

Useful for:

- Ensuring text/elements shrink with smaller screen side
    
- Perfect squares/circles that stay responsive
    

---

### **`1vmax`**

= **1% of the larger side** of the viewport  
(larger of width or height)

If viewport is 1200px × 800px →  
`1vmax = 1% of 1200px = 12px`

Useful for:

- Ensuring elements remain large on tall or wide screens
    
- Hero text that always stays big
    

---

## **3. When to Use**

### **Use `vmin` when you want:**

- Element scales based on the limiting dimension  
    (e.g., square that fits on any device)
    

```css
.box {
  width: 50vmin;
  height: 50vmin;
}
```

### **Use `vmax` when you want:**

- Element scales with the largest dimension  
    (e.g., headings that stay visually large regardless of orientation)
    

```css
.title {
  font-size: 5vmax;
}
```

---

## **4. Differences from `vw` / `vh`**

|Unit|Based On|Behavior|
|---|---|---|
|`vw`|viewport width|Changes with width only|
|`vh`|viewport height|Changes with height only|
|`vmin`|smaller of width/height|Good for responsive shapes|
|`vmax`|larger of width/height|Good for big visuals|

---

## **5. Example: Responsive Circle**

```css
.circle {
  width: 30vmin;
  height: 30vmin;
  border-radius: 50%;
}
```

Stays a perfect circle on phones, tablets, desktops.