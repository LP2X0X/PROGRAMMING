---
tags: 
 - js
 - term
 - DOM
 - fundamental
---

- In the **DOM (Document Object Model)**, everything in an HTML or XML document is represented as a **node**.
    
- A **node** is the basic building block of the DOM tree. Each piece of the document (elements, text, attributes, comments, etc.) is represented as a specific type of node.
    

---

### Example Visualize Nodes
![[Pasted image 20251201102829.png|center|700]]

---

### Common Node Types

#### 🧱 1. **Element Nodes**

Represent **HTML elements** — the building blocks of the webpage.  
Each tag like `<div>`, `<p>`, `<img>`, or `<a>` is an **element node**.

**Example:**

```html
<div>Hello</div>
```

- The `<div>` tag is an **element node**.
    
- It can contain **attributes**, **text nodes**, or even other **element nodes**.
    

🧩 **In JavaScript:**

```js
const div = document.querySelector("div");
console.log(div.nodeType); // 1  → ELEMENT_NODE
console.log(div.nodeName); // "DIV"
```

#### 💬 2. **Text Nodes**

Contain the **actual text content** inside an element.  
Text nodes always appear **as children** of element nodes.

**Example:**

```html
<div>Hello</div>
```

- `"Hello"` is a **text node**, nested inside the `<div>` element.
    

🧩 **In JavaScript:**

```js
console.log(div.firstChild.nodeType); // 3 → TEXT_NODE
console.log(div.firstChild.nodeValue); // "Hello"
```

> ⚠️ Even spaces or line breaks between tags can count as text nodes!

#### 🏷️ 3. **Attribute Nodes**

Represent **HTML attributes** such as `src`, `href`, `alt`, etc.  
While these used to be considered separate nodes, in modern DOM APIs, they are usually accessed via **element properties** instead.

**Example:**

```html
<img src="cat.png" alt="A cat">
```

- `src="cat.png"` and `alt="A cat"` are **attribute nodes** belonging to the `<img>` element.
    

🧩 **In JavaScript:**

```js
const img = document.querySelector("img");
console.log(img.getAttribute("src")); // "cat.png"
```

> ⚙️ You can still create or inspect attributes using `document.createAttribute()` — but this is rare in practice.

#### 💭 4. **Comment Nodes**

Represent comments inside the HTML document.  
They are not displayed in the browser but are part of the DOM tree.

**Example:**

```html
<!-- This is a comment -->
```

🧩 **In JavaScript:**

```js
const comment = document.createComment("Hello!");
console.log(comment.nodeType); // 8 → COMMENT_NODE
```

#### 📜 5. **Document Node**

Represents the **entire HTML document** itself.  
This is the **entry point** to access all other nodes.

🧩 **In JavaScript:**

```js
console.log(document.nodeType); // 9 → DOCUMENT_NODE
console.log(document.documentElement.nodeName); // "HTML"
```

> It’s what you interact with when you write `document.querySelector()` or `document.body`.

#### ⚙️ 6. **DocumentType Node**

Represents the `<!DOCTYPE html>` declaration that tells the browser which HTML version to use.

**Example:**

```html
<!DOCTYPE html>
```

🧩 **In JavaScript:**

```js
console.log(document.doctype.name); // "html"
```

> Usually there’s only one per document — at the very top.

---

### Visual Example (tree form)

HTML:

```html
<html>
  <body>
    <p>Hello, world!</p>
    <!-- A comment -->
  </body>
</html>
```

DOM Tree:

```
Document
 └── <html> (Element node)
      └── <body> (Element node)
           ├── <p> (Element node)
           │     └── "Hello, world!" (Text node)
           └── <!-- A comment --> (Comment node)
```

---

👉 So in short: **A node is a single point in the DOM tree that represents some part of the document — an element, text, attribute, comment, or even the whole document itself.**

---

### References
https://dom.spec.whatwg.org/#node

