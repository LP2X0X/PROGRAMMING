---
tags: html, term, technique
---

A **fragment identifier** is the part of a URL that starts with `#`, used to **jump to a specific element** on the page by its [[id]].

---

### ✅ Example:

#### HTML:

```html
<h2 id="contact">Contact Us</h2>
```

#### Link:

```html
<a href="#contact">Go to Contact Section</a>
```

Clicking this link scrolls the page to the element with `id="contact"`.

---

### 🧠 Key Notes:

- Only works if the target element has an `id`.
    
- Works **within the same page**, or **between pages**:
    

```html
<!-- Linking from another page -->
<a href="about.html#team">Meet the Team</a>
```

- The fragment (`#team`) scrolls to `id="team"` on `about.html`.
```ad-note
The browser gives us a free "top of page" link. Setting `href="#top"`, case-insensitive, or simply `href="#"`, will scroll the user to the top of the page.
```

---

Use fragment identifiers for **in-page navigation**, **table of contents**, or linking to **specific sections** of content.