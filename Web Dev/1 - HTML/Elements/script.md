---
tags:
  - html
  - element
  - fundamental
---
The **script** element in HTML is used to embed or reference executable JavaScript code. It plays a key role in making your web pages interactive, dynamic, and responsive to user input.

---

## **📦 Basic Syntax**

```html
<script>
  // Your JavaScript code goes here
  console.log("Hello from a script!");
</script>
```

Or to include external JavaScript:

```html
<script src="script.js"></script>
```

---

## **📚 Attributes** 

##  **Tag**

|**Attribute**|**Description**|
|---|---|
|src|Path to an external JS file. If present, the contents inside the  block are ignored.|
|type|Specifies the script type. Usually type=“text/javascript”, but it’s optional in modern browsers.|
|async|Loads the script asynchronously (doesn’t block parsing the rest of HTML).|
|defer|Defers execution until after the document has been parsed.|
|nomodule|Prevents modern browsers from executing script if they support JavaScript modules.|
|crossorigin|Handles CORS settings for external scripts.|

---

## **📌 Inline vs External**

🔹 Inline script:

```html
<script>
  alert("This runs immediately!");
</script>
```

🔹 External script:

```html
<script src="app.js"></script>
```

---

## **⏱️ Placement & Loading**

Traditionally,  is placed:

🔽 Just before  — to make sure the HTML is fully loaded before the script runs.

```html
<body>
  ...
  <script src="main.js"></script>
</body>
```

Or:

🚀 In the  with defer or async

```html
<head>
  <script src="main.js" defer></script>
</head>
```

🧠 Difference:

|**Attribute**|**When it runs**|**Blocks HTML parsing?**|
|---|---|---|
|None|As soon as fetched|Yes|
|defer|After DOM is parsed|No|
|async|As soon as fetched|Yes, but unpredictable|

---

## **🧪 Example**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Script Example</title>
  <script defer src="hello.js"></script>
</head>
<body>
  <h1>DOM Example</h1>
</body>
</html>
```

📁 hello.js:

```js
document.addEventListener("DOMContentLoaded", () => {
  console.log("DOM fully loaded");
});
```

---

## **🧠 Extra: Module Scripts**

Modern JavaScript supports modules:

```js
<script type="module" src="main.js"></script>
```

This allows import/export between files and is loaded in strict mode.