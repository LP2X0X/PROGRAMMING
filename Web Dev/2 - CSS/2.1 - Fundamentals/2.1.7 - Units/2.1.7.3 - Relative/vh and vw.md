---
tags: 
 - css
 - unit
 - relative
---

`vh` and `vw` are **viewport-relative units**, meaning they scale based on the current size of the browser window.

---

## **2. Definitions**

### **`1vh`**

= **1% of the viewport height**

If viewport height = 900px →  
`1vh = 9px`

### **`1vw`**

= **1% of the viewport width**

If viewport width = 1200px →  
`1vw = 12px`

---

## **3. When To Use**

### **Use `vh` when:**

- You want something to be based on **height**
    
- Full-screen sections (`100vh`)
    
- Vertically centered layouts
    
- Hero sections that match device height
    

```css
.hero {
  height: 100vh;
}
```

### **Use `vw` when:**

- You want something to scale with **width**
    
- Responsive text
    
- Images or divs that fill width naturally
    

```css
.title {
  font-size: 5vw;
}
```

---

## **4. Relationship to `vmin` / `vmax`**

|Unit|Based on|Good for|
|---|---|---|
|`vh`|viewport height|Full-height layouts|
|`vw`|viewport width|Horizontal scaling|
|`vmin`|smaller of width/height|Responsive shapes|
|`vmax`|larger of width/height|Large headings|

---

## **5. Common Issues**

On mobile devices, the viewport size can change dynamically. In an attempt to maximize available screen size, mobile browsers have a feature where some UX controls—the address bar at the top of the screen and the navigation buttons at the bottom—slide out of view as you scroll down the page. And they slide back into view if you scroll back up.
These dynamic changes cause a resize of the viewport, which in turn causes elements on the page using vh units to change size and content beneath them to jump around on the screen.

Solution: Use newer units (`dvh`, `svh`, `lvh`).

---

## **6. Notes / Tips**

- On **desktop**, `vh` works as expected.
    
- On **mobile**, it may cause **scrolling or spacing issues** due to dynamic browser UI.
    
- Consider using **`dvh`** if targeting modern mobile browsers for better responsiveness.
    