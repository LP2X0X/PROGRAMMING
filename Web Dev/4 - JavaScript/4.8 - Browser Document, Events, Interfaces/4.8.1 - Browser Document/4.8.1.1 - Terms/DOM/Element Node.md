---
tags: 
 - js
 - term
 - fundamental
---

In JavaScript, an **Element** is a specific type of **DOM node** that represents an actual HTML tag from the page. When the browser parses your HTML, each tag (like `<div>`, `<button>`, `<p>`) becomes an **Element object** in the DOM tree.

### ✅ **Element vs Node**

- A **Node** is the most generic DOM unit. It can be:
    
    - Element
        
    - Text
        
    - Comment
        
    - Document
        
    - DocumentFragment
        
- An **Element** is a _specialized node_ that specifically represents HTML elements.
    

➡️ **Every element is a node, but not every node is an element.**

---

# 🧩 **Characteristics of a DOM Element**

A DOM Element:

- Has a **tag name** (`DIV`, `SPAN`, `P`…)
    
- Can have **attributes** (id, class, etc.)
    
- Can contain **child nodes**: elements, text, comments
    
- Supports querying and selection (`querySelector`, etc.)
    
- Supports manipulation (`appendChild`, `setAttribute`, etc.)
    

Example:

```js
const el = document.querySelector("div");
console.log(el.tagName); // "DIV"
```

---

# 🔍 **Important Element APIs**

Useful properties/methods:

### **Reading structure**

- `element.children` → element _children only_
    
- `element.childNodes` → all nodes (including text)
    

### **Searching**

- `element.querySelector(selector)`
    
- `element.getElementById(id)`
    
- `element.getElementsByClassName(name)`
    

### **Manipulating**

- `element.setAttribute(name, value)`
    
- `element.classList.add("active")`
    
- `element.appendChild(node)`
    
- `element.remove()`
    

---

# 🆚 **Difference from Text Nodes**

Example HTML:

```html
<div>Hello</div>
```

DOM representation:

- `<div>` → **Element node**
    
- `Hello` → **Text node**
    

So looping with `childNodes` gives both:

```js
div.childNodes // [Element?, Text?]
```

But:

```js
div.children // [Element? only]
```

---

# 🎯 **Key Takeaways**

- “Element” in JS is the **object representation** of an HTML tag.
    
- Elements = subset of DOM nodes.
    
- Use `children` for element-only traversal.
    
- Use `childNodes` when you need to see text/comments too.
    
- Elements have tag names, attributes, and DOM manipulation APIs.
    

---

If you want, I can also give:

- A visual DOM tree diagram
    
- A comparison between `Node`, `Element`, and `HTMLElement`
    
- Common interview questions related to DOM nodes and elements