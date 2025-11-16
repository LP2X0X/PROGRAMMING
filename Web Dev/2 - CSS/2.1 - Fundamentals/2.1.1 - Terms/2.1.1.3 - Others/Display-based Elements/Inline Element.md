---
tags: 
 - css
 - term
 - element
 - type
---

# 🔹 What is an Inline Element?

An **inline element** is an element that:

- Flows **inside a line of text**, like a word or character.
    
- **Does not start on a new line**.
    
- Takes up **only as much width as its content**.
    
- **Does not respect top/bottom margin** (only left/right margin works).
    
- **Can’t have width/height** set directly (unless you change its `display`).
    

---

# 🔹 Examples of Inline Elements

Common inline elements:

- `<a>` → hyperlinks
    
- `<span>` → generic inline container
    
- `<strong>`, `<em>` → emphasis/strong text
    
- `<b>`, `<i>` → bold/italic
    
- `<img>` → inline-replaced element (special rules)
    
- `<label>`, `<abbr>`, `<cite>`, `<code>`
    

---

# 🔹 Behavior of Inline Elements

### 1. Flow

- Inline elements **sit inside a line box**, just like words in a sentence.
    
- They don’t break the line unless `white-space` or `line-break` rules apply.
    

Example:

```html
<p>This is <span>inline</span> text.</p>
```

Output: `This is inline text.` (no line break around `<span>`).

---

### 2. Width & Height

- **Cannot set `width` or `height` directly.**
    
- Their box size is determined by **content** and **font metrics**.
    

```css
span {
  width: 200px;   /* ❌ ignored */
  height: 50px;   /* ❌ ignored */
  background: yellow;
}
```

✅ To allow width/height, change to `inline-block` or `block`.

---

### 3. Margin & Padding

- **Left/right margin works**.
    
- **Top/bottom margin is ignored** (but padding does apply and affects line height).
    
- Padding increases clickable area (useful for links).
    

```css
a {
  margin: 10px 20px; /* only 20px left/right works */
  padding: 10px;     /* applies, increases space inside link */
}
```

---

### 4. Line Height

- Inline elements align based on **line-height** and **vertical-align**.
    
- They create a "line box" that stretches vertically to fit tallest inline element.
    

```css
span {
  line-height: 2;        /* doubles spacing between lines */
  vertical-align: middle; /* adjust alignment relative to text baseline */
}
```

---

### 5. Formatting Context

- Inline elements form an **inline formatting context (IFC)**.
    
- Inside, text and inline boxes are laid out horizontally.
    
- If there isn’t enough horizontal space → line breaks happen.
    

---

### 6. Special Inline Element: `<img>`

- `<img>` is inline by default but is a **replaced element**:
    
    - You can set its width/height.
        
    - It aligns like text (baseline, middle, top, bottom).
        
    - Often causes “extra gap” under images (because of baseline alignment).
        

Fix:

```css
img {
  display: block; /* or inline-block */
  vertical-align: middle;
}
```

---

# 🔹 Ways to Change Inline Behavior

1. **inline** → default, behaves like text.
    
2. **inline-block** → inline flow, but allows width/height, margin top/bottom.
    
    ```css
    a {
      display: inline-block;
      width: 150px;
      height: 50px;
    }
    ```
    
3. **block** → starts on a new line, takes full width.
    
4. **inline-flex** or **inline-grid** → advanced, inline container but flex/grid layout inside.
    

---

# 🔹 Use Cases

✅ When to use inline:

- Styling a **word or phrase** inside text (`<span>highlight</span>`).
    
- Links in a sentence (`Click <a href="#">here</a> now.`).
    
- Icons inside text (`<img src="icon.png" alt="">`).
    

✅ When not to use inline:

- For layout/boxes — switch to `inline-block` or `block`.
    
- If you need vertical spacing (margins top/bottom).
    

---

# 🔹 Quick Cheatsheet

|Feature|Inline|Inline-block|Block|
|---|---|---|---|
|Starts new line?|❌ No|❌ No|✅ Yes|
|Width/height possible?|❌ No|✅ Yes|✅ Yes|
|Margin top/bottom work?|❌ Ignored|✅ Works|✅ Works|
|Fits content only?|✅ Yes|✅ Yes|❌ No (fills parent)|