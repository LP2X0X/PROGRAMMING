---
tags: 
 - js
 - term
 - DOM
 - fundamental
---

When you query the DOM (e.g., using `children`, `childNodes`, `getElementsByTagName`, etc.), the result is often not a plain JavaScript array but a **collection object**.

These collections are **array-like objects**:

- They have a `.length` property.
    
- Items are accessible by index (`collection[0]`, `collection[1]`, …).
    
- But they are _not real arrays_ → no `.map()`, `.filter()`, etc. (unless converted with `Array.from()` or `[...collection]`).
    

````ad-note
We can use `for..of` to iterate over it:

```javascript
for (let node of document.body.childNodes) {
  alert(node); // shows all nodes from the collection
}
```
````

---

### Common Types of DOM Collections

1. **`HTMLCollection`**
    
    - Returned by many DOM methods/properties, e.g.:
        
        - `element.children`
            
        - `document.getElementsByTagName("div")`
            
    - Contains **only element nodes** (ignores text and comment nodes).
        
    - **Live collection** → updates automatically when the DOM changes.
        
    - Example:
        
        ```js
        let divs = document.getElementsByTagName("div");
        console.log(divs.length); // updates if new <div> is added
        ```
        
2. **`NodeList`**
    
    - Returned by:
        
        - `element.childNodes`
            
        - `document.querySelectorAll("p")`
            
    - Can contain **any type of node** (elements, text, comments).
        
    - Two kinds:
        
        - **Live `NodeList`** (e.g. `childNodes`) → auto-updates with DOM changes.
            
        - **Static `NodeList`** (e.g. `querySelectorAll`) → fixed snapshot at query time.
            
    - Example:
        
        ```js
        let nodes = document.querySelectorAll("p"); // static NodeList
        console.log(nodes.length); // stays the same even if new <p> is added
        ```
        
3. **`NamedNodeMap`**
    
    - Used for an element’s **attributes** (`element.attributes`).
        
    - Contains attribute nodes.
        
    - Items can be accessed by index or by name.
        
    - Example:
        
        ```js
        let attrs = document.querySelector("img").attributes;
        console.log(attrs["src"].value);
        ```
        

---

### Comparing Them

|Collection|What it holds|Live or Static|Example source|
|---|---|---|---|
|`HTMLCollection`|Elements only|Live|`document.forms`, `element.children`|
|`NodeList`|All node types|Live or Static|`childNodes` (live), `querySelectorAll` (static)|
|`NamedNodeMap`|Attributes of an element|Live|`element.attributes`|

---

### Converting to Array

Since collections aren’t real arrays:

```js
let divs = document.getElementsByTagName("div");
let arr = Array.from(divs);  // now it's a true array
arr.forEach(div => console.log(div));
```

---

### Notes

- DOM collections are read-only.
- Almost all DOM collections with minor exceptions are _live_. In other words, they reflect the current state of DOM.
	- If we keep a reference to `elem.childNodes`, and add/remove nodes into DOM, then they appear in the collection automatically.

---

👉 In short: **DOM collections are special array-like objects that represent groups of nodes (elements, children, attributes). Some are live (auto-update), some are static (fixed snapshot).**