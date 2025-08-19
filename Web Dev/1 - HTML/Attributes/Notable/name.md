---
tags:
  - html
  - attribute
  - fundamental
---

The **`name` attribute** is crucial in HTML—especially in **forms**—because it **defines how data is labeled and sent when a form is submitted**.

---

### ✅ Importance of the `name` Attribute

#### 🔹 1. **Forms: Data Submission**

- When a form is submitted, only inputs with a `name` attribute are included in the HTTP request.
    
- The **`name` is the key**, and the **input’s value is the value**, forming `key=value` pairs.
    

```html
<input type="text" name="username" value="longpham">
<!-- Submits as: username=longpham -->
```

#### 🔹 2. **Radio and Checkbox Grouping**

- **Radio buttons** and **checkboxes** use `name` to form logical groups.
    
- Only one **radio** input per group can be selected.
    
- Multiple **checkboxes** with the same name submit multiple values under that name.
    

#### 🔹 3. **JavaScript Access**

- Scripts can reference form data using the `name` property:
    

```js
document.forms[0].elements["username"]
```

#### 🔹 4. **Server-side Identification**

- On the backend (e.g., PHP, Node.js, ASP.NET), the `name` is used to extract submitted values:
    

```php
$_POST['username'];
```

---

### ⚠️ Notes

- Inputs **without a `name`** are **not submitted**.
    
- `id` is for **labeling and styling**, but `name` is for **data submission**.
    