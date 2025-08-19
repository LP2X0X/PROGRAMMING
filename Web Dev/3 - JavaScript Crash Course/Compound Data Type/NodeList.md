---
tags:
  - js
  - datatype
  - fundamental
---

In the DOM, a NodeList is a collection (or list) of nodes — typically returned by DOM querying methods like document.querySelectorAll().

It’s similar to an array, but not exactly the same.

---

## **📦 What is a NodeList?**

A NodeList is an array-like object that represents a collection of [[Web Dev/1 - HTML/Terms/DOM|DOM]] nodes (elements, text, comments, etc.).

### **Common ways you get a NodeList:**

```js
const allParas = document.querySelectorAll("p");  // NodeList of all <p> elements
const childNodes = document.body.childNodes;      // NodeList of all child nodes of <body>
```

---

## **✅ Features of NodeList**

|**Feature**|**Description**|
|---|---|
|Indexed|You can access items using index: nodelist[0], nodelist[1], etc.|
|length|Like arrays, NodeLists have a length property|
|forEach() method (modern)|NodeLists support forEach in modern browsers|
|Not always live|Most NodeLists are static (they don’t update if the DOM changes)|

---

## **❗ NodeList vs HTMLCollection**

|**Feature**|**NodeList**|**HTMLCollection**|
|---|---|---|
|Type of content|Any node type|Only elements|
|Live or Static|Usually static|Live (updates with DOM changes)|
|Methods available|forEach, item|item, namedItem|

Example:

```js
document.querySelectorAll("p");       // → NodeList (static)
document.getElementsByTagName("p");   // → HTMLCollection (live)
```

---

## **🧪 Example**

HTML:

```html
<p>One</p>
<p>Two</p>
<p>Three</p>
```

JS:

```js
const paras = document.querySelectorAll("p");  // NodeList

paras.forEach((p, index) => {
  console.log(`Paragraph ${index}: ${p.textContent}`);
});
```

---

## **📍 Converting NodeList to Array**

To use full array methods (like map, filter, etc.):

```js
const nodeList = document.querySelectorAll("li");
const array = Array.from(nodeList);    // Modern way
// or:
const array2 = [...nodeList];          // Spread syntax
```

---

## **Summary**

- NodeList is an array-like collection of DOM nodes.
- It’s read-only and often static.
- Useful for working with query results.
- You can iterate it with forEach or convert it to a real array for more powerful operations.