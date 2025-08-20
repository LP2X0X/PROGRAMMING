---
tags:
  - css
  - term
  - fundamental
---

## **1. What is the viewport?**

- **Definition:**  
    The **viewport** is the portion of the web page that is **currently visible to the user** in their browser window or device screen.  
    Think of it as the “camera view” through which you’re looking at a larger canvas (the full document).
    </br>
- **Different from the document:**  
    The document (web page) can be larger or smaller than the viewport, depending on content size. If it’s larger, you can scroll to reveal the rest.
    

---

## **2. Viewport in Desktop Browsers**

- On desktop, the viewport is usually:
    
    - **Browser window size** minus scrollbars, tabs, and other browser UI.
        
    - Can change when the user **resizes** the window.
        
- Example:  
    If your monitor is 1920×1080 and the browser window is maximized but the UI takes up some space, your viewport might be ~1903×947 pixels.
    

---

## **3. Viewport in Mobile Browsers**

- Mobile devices have **physical pixels** and **CSS pixels** (they’re not always the same because of pixel density & zoom).
    
- On mobile, the viewport can:
    
    - Change when rotating from portrait to landscape.
        
    - Be controlled with the `<meta name="viewport">` tag to adjust zoom & scale.
        
- Without a proper meta viewport tag, mobile browsers often zoom out to fit the whole page as if it were on a desktop.
    

Example of a meta viewport tag:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

- **width=device-width** → match the device's width in CSS pixels.
    
- **initial-scale=1.0** → don’t zoom in/out by default.
    

---

## **4. Measuring the viewport**

In JavaScript:

```js
window.innerWidth  // Width of viewport in pixels
window.innerHeight // Height of viewport in pixels
```

In CSS:

```css
/* Viewport-relative units */
width: 100vw; /* 100% of viewport width */
height: 100vh; /* 100% of viewport height */
```

---

## **5. Why the viewport matters**

- **Responsive design:** Decides how your layout adjusts for different screen sizes.
    
- **Fixed elements:** Elements fixed relative to the viewport stay in place when scrolling.
    
- **Media queries:** CSS breakpoints use viewport width to trigger different styles.
    
    ```css
    @media (max-width: 600px) {
      body { font-size: 14px; }
    }
    ```
    

---

## **6. Special gotchas**

- **Mobile browser UI (address bar) can shrink/expand the viewport** as you scroll.
    
- **Scrollbars** reduce effective viewport width on desktop unless using overlay scrollbars.
    
- **Zooming** changes CSS pixel size, so viewport width in CSS units changes without the physical screen changing.
    