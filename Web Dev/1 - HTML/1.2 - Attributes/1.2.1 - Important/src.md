---
tags: 
 - html
 - attribute
 - important
---

# 🔶 What is `src`?

`src` stands for **source**.  
It tells the browser **where to load an external resource from**.

It is used on elements that **embed or fetch content**, such as images, scripts, videos, iframes, etc.

---

# 🔶 Elements that use `src`

|Element|What `src` points to|
|---|---|
|`<img>`|Image file|
|`<script>`|JavaScript file|
|`<iframe>`|Web page to embed|
|`<video>`|Video file|
|`<audio>`|Audio file|
|`<source>`|Media source for `<video>` / `<audio>`|
|`<embed>`|Embedded content (PDF, plugin)|
|`<track>`|Subtitle track|
|`<input type="image">`|Image used as button|

---

# 🔶 1. `<img src="...">`

Loads an image from a URL.

```html
<img src="/images/photo.jpg" alt="Photo">
```

### Notes:

- Missing `src` → broken image icon
    
- Use `alt` for accessibility
    

---

# 🔶 2. `<script src="...">`

Loads external JavaScript.

```html
<script src="/app.js"></script>
```

### Optional behaviors:

```html
<script src="app.js" defer></script>
<script src="app.js" async></script>
```

---

# 🔶 3. `<iframe src="...">`

Embeds another webpage.

```html
<iframe src="https://example.com"></iframe>
```

---

# 🔶 4. `<video src="...">` / `<audio src="...">`

Load media files.

```html
<video src="movie.mp4" controls></video>
```

Or using `<source>`:

```html
<video controls>
  <source src="movie.mp4" type="video/mp4">
  <source src="movie.webm" type="video/webm">
</video>
```

---

# 🔶 5. Absolute vs relative paths

`src` follows standard URL rules.

### Absolute:

```html
<img src="https://example.com/logo.png">
```

### Root-relative:

```html
<img src="/assets/logo.png">
```

### Relative:

```html
<img src="images/logo.png">
```

---

# 🔶 6. When `src` is missing or empty

### Empty string → reloads current URL

```html
<img src="">
```

### Missing entirely → invalid depending on element

```html
<img>  <!-- broken -->
<script> <!-- not loaded -->
```

---

# 🔶 7. `src` vs `href` (important difference!)

|`src`|`href`|
|---|---|
|Replaces the element with external resource|Creates a link to another resource|
|Browser **fetches** it|Browser **navigates** to it|
|Used in images, scripts, media|Used in anchor links, CSS links|

Example:

- `<script src="/app.js">` loads JS.
    
- `<a href="/app.js">` navigates to the file.
    

---

# 🔶 8. Security notes

- `<script src="...">` can run code → only load trusted sources.
    
- `iframe src="...">` can embed untrusted sites → use `sandbox` attribute.
    

---

# 🔶 Summary (cheat sheet)

- **`src` = where to load external content from**
    
- Used by `<img>`, `<script>`, `<video>`, `<audio>`, `<iframe>`, `<source>`, etc.
    
- Missing or invalid `src` = failed load/broken element
    
- Not the same as `href`
    
- Can be absolute or relative URLs
    
- Affects how the browser fetches and embeds resources
    