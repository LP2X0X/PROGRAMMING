---
tags:
 - html
 - element
 - fundamental
---

### Meta

The `<meta>` tag in HTML gives **metadata** about the page — important for browsers, search engines, and social media platforms.

---

### 🧩 **Where it goes**

- Inside the `<head>` tag.
    
- Not visible to users, but used by tools and platforms.
    

---

### 🔧 **Common Types of `<meta>` Tags**

| Purpose                   | Example                                                                    |
| ------------------------- | -------------------------------------------------------------------------- |
| **Character encoding**    | `<meta charset="UTF-8">`                                                   |
| **Viewport (responsive)** | `<meta name="viewport" content="width=device-width, initial-scale=1.0">`   |
| **SEO: Description**      | `<meta name="description" content="Your site summary for search engines">` |
| **Author Info**           | `<meta name="author" content="Your Name">`                                 |
| **Redirect / Refresh**    | `<meta http-equiv="refresh" content="5;url=https://example.com">`          |

---

### 🧠 **Attributes**

- `name`: Type of metadata (e.g. "viewport", "description", "author")
- `content`: The value associated with `name`
- `charset`: Character encoding (usually "UTF-8")
- `http-equiv`: Used to simulate HTTP headers (e.g., `refresh`, `content-type`)

---

### 🔗 **Open Graph Meta Tags**

Open Graph tags are used by **social media** (like Facebook, LinkedIn) to show rich link previews.

| Property         | Description                              | Example                                                              |
| ---------------- | ---------------------------------------- | -------------------------------------------------------------------- |
| `og:title`       | Title of the page                        | `<meta property="og:title" content="My Awesome Page">`               |
| `og:description` | Description shown in the preview         | `<meta property="og:description" content="This page is about...">`   |
| `og:image`       | Image shown in the preview               | `<meta property="og:image" content="https://example.com/image.jpg">` |
| `og:url`         | Canonical URL of the page                | `<meta property="og:url" content="https://example.com/page">`        |
| `og:type`        | Type of content (e.g., website, article) | `<meta property="og:type" content="website">`                        |
| `og:site_name`   | Name of the website                      | `<meta property="og:site_name" content="My Site">`                   |

---

### 📌 **Full Example**

```html
<head>
  <!-- Standard Meta Tags -->
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Learn HTML, CSS, and JS the fun way.">
  <meta name="author" content="John Doe">

  <!-- Open Graph Meta Tags -->
  <meta property="og:title" content="Learn Web Development">
  <meta property="og:description" content="Start learning HTML, CSS, and JS with easy tutorials.">
  <meta property="og:image" content="https://example.com/preview.jpg">
  <meta property="og:url" content="https://example.com/learn">
  <meta property="og:type" content="website">
  <meta property="og:site_name" content="DevLearn">
</head>
```

---

### ✅ **Why Use Open Graph?**

- Enhances **link previews** when shared on platforms like Facebook, X, LinkedIn.
    
- Improves **click-through rates** and professional appearance.
    