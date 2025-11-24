---
tags: 
  - css
  - media
  - property
  - fundamental
---

`object-position` controls **which part of an image/video is kept visible** when the image is cropped using `object-fit: cover` or `object-fit: none`.

Think of it like **“moving the image behind the frame”**.

---

## **Default**

```css
object-position: 50% 50%;  /* center center */
```

---

## **Basic Keywords**

You can move the crop focus using keywords:

### **Horizontal**

- `left`
    
- `center`
    
- `right`
    

### **Vertical**

- `top`
    
- `center`
    
- `bottom`
    

Examples:

```css
object-position: top;
object-position: bottom right;
object-position: left center;
```

---

## **Percentage Values**

You can fine-tune positioning:

```css
object-position: 20% 80%;
```

- First value → **x-axis** (left → right)
    
- Second value → **y-axis** (top → bottom)
    

---

## **Pixel / Length Values**

You can move the image by exact amounts:

```css
object-position: 10px 40px;
```

---

## **What It Pairs With**

`object-position` mainly works with:

- **object-fit: cover** → crop but fill box
    
- **object-fit: none** → no scaling, just offset
    
- **object-fit: scale-down** → sometimes applies
    
- **object-fit: contain** → not noticeable (no cropping)
    

---

## **Useful Tailwind Classes**

Tailwind maps directly:

- `object-center`
    
- `object-top`
    
- `object-bottom`
    
- `object-left`
    
- `object-right`
    
- `object-left-top`
    
- `object-right-bottom`
    
- etc.
    

---

## **Summary**

`object-position` lets you decide **which part of the image stays visible when cropping occurs**, making it essential for controlled image focus areas.