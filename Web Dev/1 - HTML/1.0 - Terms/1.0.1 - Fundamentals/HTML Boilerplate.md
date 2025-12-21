---
tags: 
 - html
 - term
---

## 🧱 HTML Boilerplate Overview

### `<!DOCTYPE html>`

- Declares the document type
    
- Tells the browser to use **standards mode**
    
- Required to avoid quirks mode
    

---

### `<html lang="en">`

- Root element of the document
    
- `lang`:
    
    - Improves **accessibility** (screen readers)
        
    - Helps **SEO** and browser language handling
        

---

### `<head>`

Metadata container (not rendered)

Common responsibilities:

- Character encoding
    
- Page title
    
- Viewport configuration
    
- SEO and social metadata
    
- Stylesheets and icons
    

---

### `<meta charset="UTF-8">`

- Sets character encoding
    
- Prevents broken characters (especially non-English text)
    
- Should be the **first meta tag**
    

---

### `<meta name="viewport" content="width=device-width, initial-scale=1.0">`

- Controls layout on mobile devices
    
- Prevents desktop scaling on small screens
    
- Mandatory for responsive design
    

---

### `<title>`

- Browser tab name
    
- Used by search engines
    
- Appears in bookmarks and history
    

---

### `<link rel="stylesheet" href="style.css">`

- Loads external CSS
    
- Does not block HTML parsing
    
- Render-blocking until CSS is loaded
    

---

### `<script>`

Used to load JavaScript

Common patterns:

- `defer`: executes after HTML is parsed
    
- `async`: executes as soon as downloaded
    

Recommended:

```html
<script src="app.js" defer></script>
```

---

### `<body>`

- Contains all visible content
    
- Text, images, forms, navigation, components
    

---

## 🧩 Minimal Modern Boilerplate

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <link rel="stylesheet" href="style.css" />
    <script src="app.js" defer></script>
  </head>
  <body>
    <main>
      <h1>Hello World</h1>
    </main>
  </body>
</html>
```

---

## 🧠 Key Takeaways

- Boilerplate is about **predictable browser behavior**
    
- Everything in `<head>` is about **how the page behaves**
    
- Everything in `<body>` is about **what the user sees**
    
- `defer` is almost always the correct choice for scripts
    