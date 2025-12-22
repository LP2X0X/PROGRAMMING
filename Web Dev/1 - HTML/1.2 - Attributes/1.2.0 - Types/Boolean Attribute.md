---
tags: 
 - html
 - attribute
 - type
---


### Boolean Attributes

A **boolean attribute** is an attribute that is either **true by being present**, or **false by being absent**.

---

### 📌 Examples:

```html
<input type="checkbox" checked />
<input type="text" disabled />
<option selected>Option 1</option>
```

In these cases:

* `checked`, `disabled`, and `selected` are **boolean attributes**.
* You don't need to write `checked="checked"` — just having it is enough.

---

### 🧠 How It Works:

| Attribute Present | Value is... |
| ----------------- | ----------- |
| ✅ Present         | `true`      |
| ❌ Absent          | `false`     |

- When toggling between true and false, add and remove the attribute altogether with JavaScript rather than toggling the value.
	```js
	const myMedia = document.getElementById("mediaFile");
	myMedia.removeAttribute("muted");
	myMedia.setAttribute("muted");
	```

---

### 🔍 Common Boolean Attributes:

* `checked`
* `disabled`
* `readonly`
* `required`
* `autofocus`
* `multiple`
* `hidden`
* `selected`
* `controls`

---

### ✅ Tip:

For boolean attributes, **just include the attribute name** — no value needed.

```html
<input type="text" readonly> <!-- Correct -->
<input type="text" readonly="readonly"> <!-- Also valid -->
```
