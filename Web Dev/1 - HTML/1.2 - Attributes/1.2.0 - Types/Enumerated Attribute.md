---
tags: 
 - html
 - attribute
 - type
---

### Enumerated Attributes

An **enumerated attribute** is an HTML attribute that accepts a **limited set of predefined values** (called an **enumeration**).

---

### ✅ Examples:

#### 🔘 `contenteditable`

```html
<div contenteditable="true">You can edit this text</div>
<div contenteditable="false">This is not editable</div>
```

* Allowed values: `"true"`, `"false"`
* Missing or invalid value → **browser default**

---

#### 🗣️ `translate`

```html
<p translate="yes">This should be translated.</p>
<p translate="no">Do not translate this.</p>
```

* Allowed values: `"yes"`, `"no"`

---

#### 🎨 `draggable`

```html
<div draggable="true">Drag me</div>
```

* Allowed values: `"true"`, `"false"`, `"auto"`

---

### 🧠 Notes:

* Values are often **strings**, not booleans.
* **Invalid values** may be ignored or fallback to default.

---

### 📌 Common Enumerated Attributes:

| Attribute         | Common Values                         |
| ----------------- | ------------------------------------- |
| `contenteditable` | `"true"`, `"false"`                   |
| `draggable`       | `"true"`, `"false"`, `"auto"`         |
| `translate`       | `"yes"`, `"no"`                       |
| `dir`             | `"ltr"`, `"rtl"`, `"auto"`            |
| `input type`      | `"text"`, `"email"`, `"number"`, etc. |

---

Use enumerated attributes when the attribute expects one **specific value from a fixed list**.
