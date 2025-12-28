---
tags: 
 - web dev
---

### 🎯 What they are

**Tagged template literals** allow a function (the _tag_) to process a template literal **before it becomes a final string**.

```js
tag`Hello ${name}!`
```

Instead of returning a string directly, JavaScript calls `tag(...)`.

---

### 🧠 How they work

A tagged template literal is translated into a function call:

```js
tag(strings, ...values)
```

- `strings` → array of literal string segments
    
- `values` → evaluated expressions inside `${}`
    

```js
function tag(strings, ...values) {
  console.log(strings); // ["Hello ", "!"]
  console.log(values);  // ["Alex"]
}
```

```ad-note
So the function receives:
1. **An array of string segments** (not one string)
2. **Each interpolated value as a separate argument**
```

---

### 🧩 Example

```js
function highlight(strings, ...values) {
  return strings.reduce(
    (result, str, i) =>
      result + str + (values[i] ? `<strong>${values[i]}</strong>` : ""),
    ""
  );
}

const name = "Alex";
highlight`Hello ${name}!`;
```

---

### 🛠 Common use cases

- **Escaping / sanitizing values**
    
- **Formatting strings** (i18n, dates, currencies)
    
- **Building DSLs** (domain-specific languages)
    
- **CSS-in-JS** libraries (`styled-components`, `emotion`)
    
- **Logging and debugging**
    

---

### ⚠️ Important characteristics

- Expressions are **evaluated first**
    
- The tag function controls **how interpolation happens**
    
- No automatic string concatenation occurs
    
- Can return **any type**, not just strings
    

---

### 🧪 Example: safe HTML escaping

```js
function safeHTML(strings, ...values) {
  return strings.reduce(
    (acc, str, i) =>
      acc + str + (values[i]?.replace(/[&<>]/g, "") ?? ""),
    ""
  );
}
```

---

### 🧠 Mental model

> A tagged template literal is a **function call disguised as syntax sugar**, giving you full control over interpolation.

---

### ✅ Key takeaway

Use tagged template literals when you need **custom processing of template strings**, not just interpolation.
It _feels_ like calling a function with a string, but it’s actually calling a function with **the template’s structure**, which is what makes tagged templates powerful.