---
tags: 
 - js
 - fetch
 - object
---

`FormData` is a **built-in browser object** that lets you easily construct a set of key/value pairs representing form fields and their values.

The constructor is:

```javascript
let formData = new FormData([form]);
```

If HTML `form` element is provided, it automatically captures its fields.

The special thing about `FormData` is that network methods, such as `fetch`, can accept a `FormData` object as a body. It’s encoded and sent out with `Content-Type: multipart/form-data`.

From the server point of view, that looks like a usual form submission.

---

## 🔹 How to use

```js
<form id="formElem">
  <input type="text" name="name" value="John">
  <input type="text" name="surname" value="Smith">
  <input type="submit">
</form>

<script>
  formElem.onsubmit = async (e) => {
    e.preventDefault();

    let response = await fetch('/article/formdata/post/user', {
      method: 'POST',
      body: new FormData(formElem)
    });

    let result = await response.json();

    alert(result.message);
  };
</script>
```

---

## 🔹 Creating FormData

You can create an empty one:

```js
const formData = new FormData();
formData.append("username", "longpham");
formData.append("age", 25);
```

Or directly from a `<form>` element:

```html
<form id="myForm">
  <input type="text" name="username" value="longpham" />
  <input type="file" name="avatar" />
</form>
```

```js
const formElement = document.getElementById("myForm");
const formData = new FormData(formElement);
```

This will automatically capture all form inputs (including files).

---

## 🔹 Common Methods

- `append(name, value)` → Add a new field.
    
- `set(name, value)` → Replace existing or add new.
    
- `get(name)` → Get the first value by name.
    
- `getAll(name)` → Get all values with that name.
    
- `has(name)` → Check if a key exists.
    
- `delete(name)` → Remove a field.
    
- `forEach(callback)` → Iterate over entries.
    
- `formData.append(name, blob, fileName)` → add a field as if it were `<input type="file">`, the third argument `fileName` sets file name (not form field name), as it were a name of the file in user’s filesystem,
    


Example:

```js
formData.append("hobby", "coding");
formData.append("hobby", "reading");

console.log(formData.get("hobby"));    // "coding"
console.log(formData.getAll("hobby")); // ["coding", "reading"]
```

---

## 🔹 Sending with `fetch`

```js
fetch("/submit", {
  method: "POST",
  body: formData
});
```

⚠️ Important:

- You **don’t need** to set `Content-Type: multipart/form-data` yourself — the browser does it automatically with the right boundary string.
    
- If you manually set it, you’ll break the request.
    

---

## 🔹 When to use `FormData`

- Submitting HTML forms via JavaScript.
    
- Uploading files (`<input type="file">`).
    
- Sending mixed text + binary data in one request.
    