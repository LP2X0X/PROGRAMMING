---
tags:
  - html
  - attribute
  - global
---

The `contenteditable` attribute makes **any HTML element editable** directly by the user — like a mini text editor in the browser.

---

### ✅ Example:

```html
<div contenteditable="true">
  You can click here and edit this text!
</div>
```

---

### 🔢 Allowed Values:

- `"true"` — element **is editable**
    
- `"false"` — element **is not editable**
    
- _(If omitted, treated as `false`)_
    

---

### 🧠 Key Notes:

- Works on almost any element: `<div>`, `<p>`, `<td>`, etc.
    
- Changes made by the user **do not persist** unless you save them via JavaScript.
    
- Common in:
    
    - Admin dashboards
        
    - CMS previews
        
    - Rich text editors
        

---

### 🧪 Example with JavaScript:

```html
<div contenteditable="true" id="editor">Edit me!</div>
<button onclick="alert(document.getElementById('editor').innerText)">Save</button>
```

---

### 🚫 Not for Forms:

If you need user input to submit in a form, use `<input>` or `<textarea>`, or grab the content via JS.
