---
tags: 
 - css
 - tailwind
 - image
---

## 🔥 **“80/20” Tailwind Image Classes Overview**

These are the classes you will use **almost all the time** when styling images.

---

## **1. Sizing (width & height)**

### **Make image fill container**

```
w-full
h-full
```

### **Fixed size**

```
w-20  h-20
w-32  h-32
```

### **Responsive width**

```
max-w-full
h-auto
```

→ avoids distortion when resizing.

---

## **2. Object Fit (controls how image behaves in its box)**

### **Most important 3**

```
object-cover   // fill container, crop if needed
object-contain // show full image, add space if needed
object-fill    // stretch (avoid)
```

👉 _Cover is the 80/20 winner._

---

## **3. Object Position (focus area)**

### **Common ones**

```
object-center
object-top
object-bottom
object-left
object-right
object-left-top
object-right-bottom
```

Used **with object-cover** to control where cropping happens.

---

## **4. Aspect Ratio (make images square/landscape)**

### **Keep fixed ratio**

```
aspect-square     // 1:1
aspect-video      // 16:9
aspect-[3/2]      // custom
```

Used with `object-cover` to control layout.

---

## **5. Rounding (for avatars & cards)**

### **Essential rounding**

```
rounded-md
rounded-lg
rounded-xl
rounded-2xl
rounded-full
```

👉 `rounded-full` for circles (profile pictures).

---

## **6. Overflow (crop images inside containers)**

```
overflow-hidden
```

Paired with:

```
object-cover
w-full
h-full
```

---

## **7. Shadow (card-style images)**

```
shadow
shadow-md
shadow-lg
shadow-xl
```

---

## **8. Borders**

```
border
border-gray-200
border-white
```

---

## 🎯 **80/20 Recommended Image Combo**

This is the single most useful, production-ready pattern for images:

```html
<div class="aspect-square overflow-hidden rounded-xl">
  <img src="" class="w-full h-full object-cover object-center">
</div>
```

You will use this combo constantly in:

- profile avatars
    
- product cards
    
- galleries
    
- thumbnails
    
- dashboard widgets
    
- responsive grids
    

---

## 🧠 **Quick Memory Hook**

1. `w-full h-full` → fill box

2. `object-cover` → crop

3. `object-center` → nice focus

4. `aspect-square` → clean layout

Master these, and 80% of image layouts become trivial.
