---
tags: 
 - css
 - technique
 - layout
---



- You’ll want to constrain the width of the page’s main column. Because block-level elements fill the width of their container by default, you will often want to reduce their width from this default.
- In your page, \<body> serves as the outer container. By default, this is already 100% of the page width, so you won’t have to apply any new styles to it. Inside that, you’ve wrapped the main contents of the page in a \<div class="container">, which serves as the inner container. You will restrict the width of this container and apply margins to center it.

```html
<div class="outer-container">
  <div class="inner-container">
    <!-- content -->
  </div>
</div>
```

---

### **Why it's used**

1. **Max-width + full-bleed elements**
    
    - Outer container handles full-width background or spacing.
        
    - Inner container keeps the text/content centered and limited.
        
2. **Consistent page alignment**
    
    - Inner container ensures alignment across sections.
        
3. **Easy to add section backgrounds**
    
    - Apply background to outer.
        
    - Keep readable content inside inner.
        

---

### **Common pattern**

```html
<section class="px-4">
  <div class="max-w-4xl mx-auto">
    <!-- content -->
  </div>
</section>
```

- Outer: padding + full width
    
- Inner: max-width + centered
    

```ad-tip
Reference [[Magic Centering]].
```

---

### **Notes**

- You can also use the same pattern a second time in the header to keep it aligned with the content beneath. The \<header> will serve as the outer container, and the \<h1> will be the inner container. Because this needs to be the same width, you can use a custom property to control the size of both in one place.