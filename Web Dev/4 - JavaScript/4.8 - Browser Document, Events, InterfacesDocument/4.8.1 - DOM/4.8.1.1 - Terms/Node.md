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

### Common Node Types

1. **Element nodes**  
    Represent HTML elements (tags).  
    Example:
    
    ```html
    <div>Hello</div>
    ```
    
    - The `<div>` is an **element node**.
        
2. **Text nodes**  
    Contain the text inside elements.  
    Example:
    
    - In `<div>Hello</div>`, the word `"Hello"` is stored as a **text node**, child of the `<div>` node.
        
3. **Attribute nodes** (less used directly in modern DOM APIs)  
    Represent attributes of elements.  
    Example:
    
    ```html
    <img src="cat.png" alt="A cat">
    ```
    
    - `src="cat.png"` and `alt="A cat"` are **attribute nodes** belonging to the `<img>` element.
        
4. **Comment nodes**  
    Represent comments in the document.  
    Example:
    
    ```html
    <!-- This is a comment -->
    ```
    
5. **Document node**  
    Represents the entire document (`document` object in JS).
    
6. **DocumentType node**  
    Represents the `<!DOCTYPE html>` declaration.
    

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

### References
https://dom.spec.whatwg.org/#node

---

👉 So in short: **A node is a single point in the DOM tree that represents some part of the document — an element, text, attribute, comment, or even the whole document itself.**
