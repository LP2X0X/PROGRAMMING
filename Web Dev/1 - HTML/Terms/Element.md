---
tags: html, term, fundamental
---

- An **element** is a complete structure consisting of an opening tag, content, and a closing tag. You use element to enclose, or wrap, different parts of the content to make it appear or act in a certain way.
![[Pasted image 20250804200322.png|500|center]]

---

## Types of elements
### ✅ 1. **Void Elements**

Void elements are **self-closing HTML tags** — they **don’t have closing tags** and **can’t have content inside**.

**Examples:**

```html
<br>
<img src="image.jpg">
<input type="text">
<hr>
<meta charset="UTF-8">
<link rel="stylesheet" href="style.css">
```

> These are always written without a closing tag, e.g., `<br>` not `<br></br>`.

### ✅ 2. **Replaced Elements**

Replaced elements are HTML elements **whose content is not defined by HTML**, but instead **by an external resource**, be it a graphical user interface (UI) widget in the case of most form controls, or a raster or scalable image file in the case of most images.

**Examples:**

* `<img>` → replaced by an image file.
* `<input>` → replaced by a form control.
* `<video>` / `<audio>` → replaced by media player.
* `<iframe>` → replaced by an embedded webpage.
* `<embed>` / `<object>` → replaced by external resources.

> Think of these as "placeholders" replaced by browser-rendered content.

### ✅ 3. **Non-replaced Elements**

Non-replaced elements **render exactly as defined in HTML** — their content comes from inside the element.

**Examples:**

```html
<p>Hello</p>
<div>Content here</div>
<span>Inline text</span>
<ul><li>List</li></ul>
```

> These elements are *not* replaced by external data. They contain and render their own child elements or text.

---

### 🧠 Summary Table

| Type         | Can have content? | Closed with `</>`? | Rendered from external source? |
| ------------ | ----------------- | ------------------ | ------------------------------ |
| Void         | ❌ No              | ❌ No               | ❌ No                           |
| Replaced     | ✅ (indirectly)    | ✅ Yes / ❌ No       | ✅ Yes                          |
| Non-replaced | ✅ Yes             | ✅ Yes              | ❌ No                           |