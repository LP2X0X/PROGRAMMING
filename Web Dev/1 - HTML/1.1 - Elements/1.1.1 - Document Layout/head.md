---
tags:
  - html
  - element
  - layout
  - fundamental
---

- `<head>` is a **container for metadata** about the HTML document.
    
- Metadata = "data about data" → it describes the page but isn’t directly shown to users in the main browser window.
    
- Browsers, search engines, and assistive tech use `<head>` to understand the page.
    

---

## 🔑 Common Elements Inside `<head>`

### 1. **`<title>`**

```html
<title>My Website</title>
```

- Sets the page title (shown in browser tab, bookmarks, search results).
    

---

### 2. **`<meta>`**

Provides metadata in name–content pairs.

- **Character encoding**:
    
    ```html
    <meta charset="UTF-8">
    ```
    
- **Responsive design**:
    
    ```html
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    ```
    
- **Description (SEO)**:
    
    ```html
    <meta name="description" content="A short summary of my website.">
    ```
    
- **Keywords (less relevant today)**:
    
    ```html
    <meta name="keywords" content="HTML, CSS, tutorial">
    ```
    

---

### 3. **`<link>`**

- Links external resources (CSS, icons, etc.).
    
    ```html
    <link rel="stylesheet" href="styles.css">
    <link rel="icon" href="/favicon.ico" type="image/x-icon">
    ```
    

---

### 4. **`<style>`**

- Embeds CSS directly.
    
    ```html
    <style>
      body { font-family: Arial; }
    </style>
    ```
    

---

### 5. **`<script>`**

- Adds JavaScript (usually in `<body>`, but sometimes in `<head>`).
    
    ```html
    <script src="app.js" defer></script>
    ```
    
    - `defer`: waits until HTML is parsed before running.
        
    - `async`: runs as soon as possible (not guaranteed in order).
        

---

### 6. **Other tags you may see**

- **`<base>`**: sets a base URL for all relative links.
    
    ```html
    <base href="https://example.com/">
    ```
    
- **`<meta property="og:...">`**: Open Graph tags (social media previews).
    
- **`<meta name="twitter:...">`**: Twitter Card metadata.
    

---

## 📋 Example of a Typical `<head>`

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Cool Site</title>
  <meta name="description" content="Learn HTML, CSS, and JavaScript easily.">
  
  <!-- CSS -->
  <link rel="stylesheet" href="styles.css">

  <!-- Favicon -->
  <link rel="icon" href="/favicon.ico">

  <!-- Open Graph (for social sharing) -->
  <meta property="og:title" content="My Cool Site">
  <meta property="og:description" content="Learn HTML, CSS, and JavaScript easily.">
  <meta property="og:image" content="/images/preview.png">

  <!-- JS -->
  <script src="app.js" defer></script>
</head>
```

---

## ✅ Summary

- `<head>` contains metadata, links to resources, and instructions for the browser/search engines.
    
- Key items: `<title>`, `<meta>`, `<link>`, `<style>`, `<script>`.
    
- It **does not render visible content** (that goes in `<body>`).
    