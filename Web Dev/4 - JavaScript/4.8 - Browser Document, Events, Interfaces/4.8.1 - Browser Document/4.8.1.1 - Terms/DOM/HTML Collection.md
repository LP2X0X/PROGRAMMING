---
tags: 
 - js
 - html
 - DOM
 - collection
---

# **HTMLCollection — Overview**

### ✅ **What it is**

`HTMLCollection` is a **live**, array-like collection of DOM elements.

It is returned by DOM methods such as:

- `document.getElementsByTagName()`
    
- `document.getElementsByClassName()`
    
- `element.children`
    
- `document.forms`, `document.images`, `document.links`, etc.
    

---

# **Key Characteristics**

### **1. Live collection**

If the DOM changes, the collection updates **automatically**.

```js
const items = document.getElementsByTagName("li");
console.log(items.length); // 3

ul.appendChild(document.createElement("li"));

console.log(items.length); // 4 (updated!)
```

---

### **2. Array-like, NOT a real array**

You can access items like:

```js
items[0]
items.length
```

…but you **cannot** use array methods directly:

❌ `items.map(...)`  
❌ `items.forEach(...)` (in older browsers)

To convert it:

```js
const arr = [...items];
```

---

### **3. Contains only elements**

No text nodes, no comment nodes.

This makes `HTMLCollection` different from:

### **→ NodeList**

- `querySelectorAll()` returns a **static** NodeList.
    
- NodeList can include text nodes (depending on method).
    
- NodeList (modern) supports `forEach`.
    

---

# **Common sources**

|API|Returns|Live?|
|---|---|---|
|`getElementsByClassName`|HTMLCollection|✔️ Live|
|`getElementsByTagName`|HTMLCollection|✔️ Live|
|`element.children`|HTMLCollection|✔️ Live|
|`document.querySelectorAll`|NodeList|❌ Static|
|`childNodes`|NodeList|❌ Static|

---

# **Why does it exist?**

Before `querySelector` era, browsers needed a fast way to keep dynamic references to elements without re-querying the DOM every time.

---

# 🧠 **Why HTMLCollection Is Live**

Because it's tied to the DOM tree directly.  
When the DOM structure changes, HTMLCollection reflects new state immediately.

Useful in some cases, but **dangerous if you don’t expect it**, like looping while removing children:

```js
const items = document.getElementsByTagName("li");

for (let i = 0; i < items.length; i++) {
  items[i].remove(); // length shrinks WHILE looping
}
```

This can skip elements.

---

# 🟩 **Safe approach**

Convert HTMLCollection to array first:

```js
const arr = [...document.getElementsByClassName("box")];
arr.forEach(el => el.remove());
```