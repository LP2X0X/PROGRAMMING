---
tags: 
 - js
 - term
 - DOM
---

# 🌿 **NodeList Overview**

A **NodeList** is a **collection of DOM nodes** returned by DOM-selection methods like:

- `document.querySelectorAll()`
    
- `element.childNodes`
    
- `document.getElementsByName()` (live NodeList)
    

It looks like an array, but **it is NOT a real array**.

---

# 🧩 **1. What a NodeList Contains**

Depending on the method, it might include:

- Element nodes
    
- Text nodes
    
- Comment nodes
    

Example:

```js
const nodes = document.querySelectorAll("p");
console.log(nodes); // NodeList of <p> elements
```

---

# 🧠 **2. NodeList Is “Array-like” but Not an Array**

This means:

### It has:

- `.length`
    
- Index access: `nodes[0]`
    
- `.forEach()`
    

### It does NOT have:

- `.map()`
    
- `.filter()`
    
- `.reduce()`
    

Unless converted.

---

# 🔄 **3. Convert NodeList → Array**

```js
const arr = Array.from(nodes);
```

or

```js
const arr = [...nodes];
```

Then you can do:

```js
arr.map(el => el.textContent);
```

---

# 📌 **4. Two Types of NodeList**

## **1️⃣ Static NodeList**

Returned by: `querySelectorAll()`

- Does NOT auto-update when DOM changes
    
- Stable, predictable
    
- Most commonly used
    

Example:

```js
const items = document.querySelectorAll("li");
// Adding more <li> later will NOT update items.
```

## **2️⃣ Live NodeList**

Returned by: `childNodes`, `getElementsByName`, `getElementsByClassName`

- Automatically updates when DOM changes
    
- Can cause unexpected behavior
    

Example:

```js
const nodes = document.body.childNodes;
document.body.appendChild(document.createElement("div"));
console.log(nodes.length); // updates automatically
```

---

# 🧭 **5. Common Operations**

### Loop over NodeList

```js
nodes.forEach(node => console.log(node));
```

### Access a specific node

```js
const first = nodes[0];
```

### Filter nodes (after converting):

```js
[...nodes]
  .filter(node => node.nodeType === Node.ELEMENT_NODE);
```

---

# 🔥 **6. NodeList vs HTMLCollection**

| Feature              | NodeList                         | HTMLCollection                                   |
| -------------------- | -------------------------------- | ------------------------------------------------ |
| Returned from        | `querySelectorAll`, `childNodes` | `getElementsByClassName`, `getElementsByTagName` |
| Static or Live       | Both                             | Always Live                                      |
| Supports forEach     | Yes                              | No (in many browsers)                            |
| Includes text nodes? | Sometimes                        | No, only elements                                |

---

# 🎯 Why React Developers Should Care

React uses the **Virtual DOM**, but sometimes you still work with real nodes:

- `useRef()` gives you an actual DOM node
    
- Animations / measuring size
    
- Portals
    
- Third-party libraries
    

Understanding NodeLists helps avoid surprises when DOM changes.