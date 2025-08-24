---
tags: 
 - css
 - layout
 - property
---


## **📖 Definition**

The **float property** in CSS positions an element to the left or right of its container, allowing text and inline elements to wrap around it. It was originally designed for placing images in text but is also used for layouts (though modern CSS favors Flexbox and Grid instead).

```ad-note
Floated elements are removed from [[Normal Flow|normal flow]].
```

---

## **🖋️ Syntax**

```css
selector {
  float: left | right | none | inline-start | inline-end;
}
```

- left → Floats the element to the left side of its container.
    
- right → Floats the element to the right side of its container.
    
- none → Default; element stays in normal document flow.
    
- inline-start / inline-end → Floats relative to writing mode (useful for RTL text).
    

---

## **💡 Example**

HTML:

```html
<img src="photo.jpg" alt="example" class="float-img">
<p>This paragraph wraps around the floated image.</p>
```

CSS:

```css
.float-img {
  float: left;
  margin: 0 15px 15px 0;
  width: 150px;
}
```

This makes the image float to the left and text flows neatly around it.

---

## **🛠️ Tips & Best Practices**

- ✅ **Use for wrapping text**: Best when you want inline text to flow around an image or block.
    
- ❌ **Avoid for layout**: Use Flexbox or Grid for modern page layouts (floats were a hacky workaround before).
    
- 🔄 **Clear floats**: Use the clear property (e.g., clear: both;) or clearfix techniques to prevent floated elements from overlapping with following content.
    
- ⚠️ **Floats are removed from normal flow**: They can cause parent containers to collapse in height. Fix with overflow: auto; or clearfix.
    
- 🌍 **Legacy support**: Floats are well supported everywhere but considered “old-school” for layout purposes.
    