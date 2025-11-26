---
tags: 
 - html
 - attribute
 - important
---

# 🔶 **What is `href`?**

`href` stands for **Hypertext Reference**.  
It specifies the **destination** that a link or resource points to.

You use it on elements that _link to something_.

---

# 🔶 **Where is `href` used?**

|Element|What `href` points to|
|---|---|
|`<a>`|A page, section, file, or protocol link|
|`<link>`|External resources (CSS, icons, preloads)|
|`<area>`|Image map target|
|`<base>`|Sets default URL base for relative links|

---

# 🔶 **1. `<a href="...">` — hyperlinks**

Most common usage.

```html
<a href="https://example.com">Visit</a>
```

**Types of values:**

- URL
    
- fragment (`#section1`)
    
- protocol (`mailto:`, `tel:`)
    
- relative path (`./page`, `/about`)
    
- file download
    

### Example types:

```html
<a href="#top">Jump to top</a>
<a href="mailto:hello@example.com">Email us</a>
<a href="tel:+12345678">Call</a>
<a href="/docs/manual.pdf">Download PDF</a>
```

---

# 🔶 **2. Absolute vs relative URLs**

### **Absolute**

```html
<a href="https://site.com/page">
```

### **Root-relative**

```html
<a href="/page">
```

### **Relative to current folder**

```html
<a href="page.html">
<a href="../parent/page.html">
```

---

# 🔶 **3. `<link href="...">` — external resource**

Used in the `<head>`.

```html
<link rel="stylesheet" href="style.css">
<link rel="icon" href="/favicon.ico">
<link rel="preload" href="/font.woff2" as="font">
```

---

# 🔶 **4. `href` with `<base>`**

Sets the **base URL** for all relative `href` values.

```html
<base href="https://cdn.example.com/">
```

Now all relative links use this base.

---

# 🔶 **5. Special protocols**

You can use custom or browser protocols.

### `mailto:` (email)

```html
<a href="mailto:test@example.com">Email</a>
```

### `tel:` (phone)

```html
<a href="tel:+1800123456">Call</a>
```

### `sms:`

```html
<a href="sms:12345">Send SMS</a>
```

---

# 🔶 **6. Empty or missing `href`**

### Empty:

```html
<a href="">...</a>
```

Reloads the page.

### Missing:

```html
<a>...</a>
```

Not keyboard-focusable.  
Not link behavior.  
Should use `<button>` instead if interactive.

---

# 🔶 **7. Security notes**

🟠 **Avoid `href="javascript:..."`**  
It's unsafe, deprecated, and blocks CSP.

```html
<a href="javascript:alert(1)">  <!-- bad -->
```

🟢 Use event listeners instead.

---

# 🔶 **8. Using `target` with `href`**

```html
<a href="/docs" target="_blank">Open in new tab</a>
```

Add security:

```html
<a href="/docs" target="_blank" rel="noopener noreferrer">
```

---

# 🔶 **9. SEO considerations**

- Links with `href` can be crawled by search engines.
    
- Missing `href` → not a real link.
    
- Prefer semantic `<a>` for navigation.
    

---

# 🔶 Summary (cheat sheet)

- **`href` defines where a link or resource goes.**
    
- Used by: `<a>`, `<link>`, `<area>`, `<base>`.
    
- Supports absolute URLs, relative paths, fragments, and protocols.
    
- Empty → reloads page.
    
- Missing → not a link.
    
- Avoid `javascript:` URLs.
    
- For navigation: always use `<a href="...">`.
    