---
tags: 
  - html
  - element
  - fundamental
---

## 🔗 `<link>` Element — Overview

### 📌 Purpose

- Declares a relationship between the current document and an external resource
    
- Commonly used for **stylesheets, icons, manifests, and performance hints**
    
- A **void element** (no closing tag)
    
- Must be placed inside the `<head>`
    

```html
<link rel="stylesheet" href="/styles.css">
```

---

### 🧱 Core Characteristics

- Not visible in the UI
    
- Not interactive
    
- Used for **metadata and resource loading**
    
- Interpreted entirely by the browser
    

---

### 🔁 Common `rel` Values

- **stylesheet** — load and apply CSS
    
- **icon** — define favicon
    
- **preload** — fetch critical resources early
    
- **prefetch** — fetch possible future resources
    
- **preconnect** — establish early connection to another origin
    
- **dns-prefetch** — resolve DNS early
    
- **manifest** — link to web app manifest
    
- **alternate** — alternate representations (RSS, language versions)
    

---

### ⚙️ Key Attributes

- **href**  
    URL of the linked resource
    
- **rel**  
    Relationship type (required for behavior)
    
- **as**  
    Resource type for `preload` (`style`, `script`, `font`, `image`)
    
- **type**  
    MIME type (mostly optional now)
    
- **media**  
    Conditional loading (e.g. responsive CSS)
    
- **crossorigin**  
    Required for cross-origin fonts and some preloads
    
- **integrity**  
    Enables Subresource Integrity (SRI)
    

---

### 🎨 Stylesheet Usage

- Stylesheets loaded via `<link>` are **render-blocking** by default
    
- Media queries can delay application
    

```html
<link rel="stylesheet" href="print.css" media="print">
```

- Preferred over `<style>` for shared or large CSS files
    

---

### 🚀 Performance Use Cases

- Preload critical resources
    

```html
<link rel="preload" href="/fonts/app.woff2" as="font" crossorigin>
```

- Preconnect to third-party origins
    

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
```

---

### 🖼️ Favicon Example

```html
<link rel="icon" href="/favicon.ico" type="image/x-icon">
```

---

### 🔄 `<link>` vs `<a>`

- `<link>`  
    Metadata only, browser-level behavior, non-interactive
    
- `<a>`  
    User navigation, clickable, part of document content
    

---

### ⚠️ Common Pitfalls

- Missing `rel` → browser ignores the link
    
- Using `<link>` outside `<head>`
    
- Forgetting `crossorigin` for preloaded fonts
    
- Expecting `<link>` to be clickable
    

---

### 🧠 Mental Model

Think of `<link>` as **declaring dependencies** of the document:

> “This page depends on or is related to this external resource.”
