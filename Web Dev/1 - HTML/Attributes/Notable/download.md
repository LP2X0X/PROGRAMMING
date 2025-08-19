---
tags:
  - html
  - attribute
---

The `download` attribute on an `<a>` tag tells the browser to **download** the linked file instead of navigating to it.

---

### ✅ Basic Syntax:

```html
<a href="files/report.pdf" download>Download Report</a>
```

- Clicking this link will **download** `report.pdf` instead of opening it in the browser.
    

---

### 📝 Rename the File:

```html
<a href="files/report.pdf" download="MyFile.pdf">Download Report</a>
```

- The file will be saved as `MyFile.pdf`.
    

---

### 🔐 Notes:

- Only works for **same-origin URLs** (same domain).
    
- For **cross-origin** links, the server must send the correct `Content-Disposition: attachment` header.
    
- Does **not** work for:
    
    - Blobs or data URLs in some browsers without JavaScript
        
    - Pages meant to be opened, like `.html`
        

---

### 🧠 Use Cases:

- Downloadable PDFs, images, or CSVs
    
- Triggering downloads without server logic
    
- Letting users save a pre-generated file
    
