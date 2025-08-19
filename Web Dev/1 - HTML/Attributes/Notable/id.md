---
tags:
  - html
  - attribute
  - global
  - fundamental
---

### id

The `id` attribute gives an **HTML element a unique identifier** within the page.

---

### ✅ Purpose:

* Uniquely identify an element.
* Use for:

  * **CSS styling**
  * **JavaScript selection**
  * **Link targets** (with `#id`)

---

### 🧾 Example:

```html
<h1 id="welcome">Welcome</h1>
<a href="#welcome">Go to heading</a>
```

---

### ⚠️ Rules:

* Must be **unique** per page.
* Should start with a **letter** (not a number).
* Cannot contain **spaces**.

---

### 💡 Common Uses:

* CSS:

  ```css
  #welcome { color: blue; }
  ```
* JavaScript:

  ```js
  document.getElementById("welcome").textContent = "Hi!";
  ```

The `id` attribute is perfect for **targeting one specific element**.
