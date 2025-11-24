---
tags: 
 - css
 - unit
 - relative
---

## **1. What is `dvh`?**

`dvh` stands for **Dynamic Viewport Height**. It is a relative length unit in CSS that represents **1% of the viewport height**, but with a **dynamic behavior on mobile devices**.

### **Key idea**:

- Unlike the classic `vh` unit, `dvh` **adapts to changes in the viewport caused by browser UI**, like the address bar or on-screen keyboard appearing/disappearing.
    

---

## **2. Syntax**

```css
.element {
  height: 50dvh; /* 50% of the dynamic viewport height */
}
```

- **1dvh** = 1% of the viewport height **excluding dynamic UI overlays**
    
- You can use `dvh` anywhere `length` is accepted (height, min-height, max-height, etc.)
    

---

## **3. Why `dvh` is needed**

On mobile browsers, using `100vh` often causes issues:

- **Problem with `vh`**:  
    On mobile devices, the **browser chrome** (address bar, toolbar) can appear/disappear when scrolling.  
    Example:
    

```css
.element {
  height: 100vh;
}
```

- This can make a full-screen element **overflow or underflow** when the address bar hides.
    
- Users may see unwanted white space or scroll issues.
    
- **Solution: `dvh`** adjusts **in real time**, giving a more reliable “full screen” behavior.
    

---

## **4. Comparison Table**

|Unit|Behavior on Mobile|Updates on UI change|
|---|---|---|
|`vh`|Fixed at initial viewport height|**Does not adjust dynamically**|
|`lvh` (large viewport height)|Fixed at largest viewport height|Does not shrink|
|`svh` (small viewport height)|Fixed at smallest viewport height|Does not expand|
|`dvh`|Dynamic|**Updates automatically** when UI changes|

> `dvh` is preferred for **full-screen layouts** on mobile because it reacts to browser UI changes.

---

## **5. Browser Support**

- Most modern browsers support `dvh` (Chrome, Edge, Firefox, Safari 16+).
    
- On older browsers, you may still need fallback using `vh`:
    

```css
.element {
  height: 100vh; /* fallback */
  height: 100dvh; /* dynamic height */
}
```

---

## **6. Use Cases**

1. **Full-screen hero sections**:
    

```css
.hero {
  height: 100dvh;
  display: flex;
  justify-content: center;
  align-items: center;
}
```

2. **Modal windows / overlays** that need to cover the visible screen without being affected by the address bar:
    

```css
.modal {
  max-height: 90dvh;
  overflow-y: auto;
}
```

3. **Responsive layouts** on mobile where the viewport height may change due to on-screen keyboard:
    

```css
.form-container {
  height: 80dvh; /* adapts when keyboard opens */
}
```

---

## **7. Notes / Tips**

- `dvh` is **dynamic**—it recalculates when the browser UI shows/hides.
    
- Use it instead of `100vh` for **true full-screen layouts** on mobile.
    
- Combine with `min-height` or `max-height` for better control:
    

```css
.container {
  min-height: 50dvh;
  max-height: 90dvh;
}
```

---

✅ **Summary:**  
`dvh` is the **modern replacement for `vh` on mobile**, solving scroll and full-screen issues caused by dynamic browser UI. Always consider it when designing **mobile-first layouts**.