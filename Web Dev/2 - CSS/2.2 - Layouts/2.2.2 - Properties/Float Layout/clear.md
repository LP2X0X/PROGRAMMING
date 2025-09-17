---
tags: 
 - css
 - layout
 - property
---

## **📖 Definition**

The clear property tells an element **on which sides it should not allow floating elements to be placed next to it**.

In other words, it forces the element to move **down below floated elements**.

---

## **🖋️ Syntax**

```css
element {
  clear: none | left | right | both | inline-start | inline-end;
}
```

---

## **📝 Values**

- **none** (default)
    
    No clearing; element may sit next to floated elements.
    
- **left**
    
    The element is moved below any **left-floated** elements.
    
- **right**
    
    The element is moved below any **right-floated** elements.
    
- **both**
    
    The element is moved below **all floated elements** (left and right).
    
- **inline-start**
    
    Clears floats on the **start side** (left in LTR, right in RTL).
    
- **inline-end**
    
    Clears floats on the **end side** (right in LTR, left in RTL).
    

---

## **💡 Example**

  

HTML:

```html
<div class="box left">Float Left</div>
<div class="box right">Float Right</div>
<p class="text">This text might wrap around the floats.</p>
<p class="clear-both">This text will appear below both floats.</p>
```

CSS:

```css
.box.left { float: left; width: 100px; height: 50px; background: lightblue; }
.box.right { float: right; width: 100px; height: 50px; background: lightgreen; }

.text { background: #eee; } 

.clear-both { clear: both; background: yellow; }
```

➡️ Without clear, the paragraph text may wrap around the floats.

➡️ With clear: both, the element is pushed below the floats.

---

## **🛠️ Tips & Best Practices**

- Use clear to prevent unwanted text/image wrapping around floated elements.
    
- Common in old layouts (before flexbox/grid), especially for **clearfix hacks**.
    
- Nowadays, prefer **flexbox** or **grid** for layout, but clear is still useful for controlling floats in legacy or text-flow contexts.
    
- clear: both is the most commonly used value in practice.
    