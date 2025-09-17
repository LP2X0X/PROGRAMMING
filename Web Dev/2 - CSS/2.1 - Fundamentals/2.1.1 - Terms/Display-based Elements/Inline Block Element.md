---
tags: 
 - css
 - term
 - element
 - type
---

# 🔹 What is an Inline-Block Element?

An **inline-block element** is an element that:

- Flows **inline with text** (like inline).
    
- But you can set **width, height, margin, and padding on all sides** (like block).
    
- Does **not force a new line** by default.
    
- Allows other inline(-block) elements to sit beside it.
    

---

# 🔹 Examples of Inline-Block Elements

By default:

- `<img>`
    
- `<button>`
    
- `<input>`
    
- `<textarea>`
    
- `<select>`
    
- `<label>`
    

Any element can be made inline-block with:

```css
div {
  display: inline-block;
}
```

---

# 🔹 Behavior of Inline-Block Elements

### 1. Flow

- Inline-blocks **stay in the same line** until there’s no space, then wrap to the next line.
    

```html
<span style="display:inline-block;width:100px;height:50px;background:lightblue;">A</span>
<span style="display:inline-block;width:100px;height:50px;background:lightgreen;">B</span>
```

✅ They sit side by side.

---

### 2. Width & Height

- Unlike pure inline elements, you **can explicitly set width and height**.
    

```css
span {
  display: inline-block;
  width: 120px;
  height: 60px;
  background: coral;
}
```

---

### 3. Margins & Padding

- **All margins and padding work** (top, bottom, left, right).
    
- This makes inline-blocks useful for **button-like elements** or **grid-like layouts**.
    

---

### 4. Alignment

- Inline-blocks align like text. Their baseline aligns with surrounding inline text.
    
- You can adjust with `vertical-align`:
    
    - `baseline` (default)
        
    - `top`
        
    - `middle`
        
    - `bottom`
        

Example:

```css
span {
  display: inline-block;
  vertical-align: middle;
}
```

---

### 5. Whitespace Issue

When writing inline-blocks in HTML, **spaces between them count as actual spaces** (like text).

```html
<span class="box"></span> <span class="box"></span>
```

→ Adds a ~4px gap.

Fixes:

- Remove the whitespace in HTML.
    
- Comment trick:
    
    ```html
    <span class="box"></span><!--
    --><span class="box"></span>
    ```
    
- Or set `font-size: 0` on the parent container.
    

---

# 🔹 Differences vs Inline & Block

|Feature|Inline|Block|Inline-Block|
|---|---|---|---|
|Starts new line?|❌ No|✅ Yes|❌ No|
|Default width|Content only|100% parent|Content (unless set)|
|Can set width/height?|❌ No|✅ Yes|✅ Yes|
|Margin top/bottom work?|❌ Ignored|✅ Works|✅ Works|
|Flows inline with text?|✅ Yes|❌ No|✅ Yes|

---

# 🔹 Use Cases

✅ Buttons styled as `<a>` or `<span>`  
✅ Menu items (navigation bars)  
✅ Laying out grid-like structures before `flexbox`/`grid` existed  
✅ Form inputs  
✅ Image galleries

---

# 🔹 Quick Example

```html
<style>
  .btn {
    display: inline-block;
    padding: 10px 20px;
    margin: 5px;
    background: #3498db;
    color: white;
    border-radius: 6px;
  }
</style>

<a href="#" class="btn">Button 1</a>
<a href="#" class="btn">Button 2</a>
```

✅ Buttons sit inline, can have width/height/padding, and behave like blocks in sizing.

---

⚡ **Summary**:

- **Inline = text-like, no box control.**
    
- **Block = big boxes, full width.**
    
- **Inline-block = inline flow + block box sizing.**
    