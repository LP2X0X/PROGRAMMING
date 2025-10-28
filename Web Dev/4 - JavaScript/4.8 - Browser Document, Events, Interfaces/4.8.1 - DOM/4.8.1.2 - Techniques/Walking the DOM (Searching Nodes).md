---
tags: 
 - js
 - DOM
 - manipulation
 - fundamental
---

- All operations on the DOM start with the `document` object. That’s the main “entry point” to DOM. From it we can access any node.
- Here’s a picture of links that allow for travel between DOM nodes:
![[Pasted image 20250917092658.png|center|400]]

---

### Top-Level DOM Nodes in a Document

The browser provides direct properties on the `document` object to access the main structural nodes of an HTML page:

1. **`document.documentElement` → `<html>`**
    
    - Represents the root element of the document.
        
    - Refers to the `<html>` tag itself.
        
2. **`document.body` → `<body>`**
    
    - Provides access to the main content container of the page.
        
    - Refers to the `<body>` element.
        
3. **`document.head` → `<head>`**
    
    - Provides access to metadata and resources linked in the document.
        
    - Refers to the `<head>` element.
        

---

### Children: `childNodes`, `firstChild`, `lastChild`

#### Definitions

|Term|What it means|
|---|---|
|**Child nodes (or “children”)**|Direct descendants — nodes nested immediately inside a given node.|
|**Descendants**|All nodes in the subtree of a given node — children, grandchildren, etc.|

#### What these properties do

- `childNodes`  
    Returns a collection of _all_ child nodes, **including**:
    
    - Element nodes (tags)
        
    - Text nodes (whitespace, text between tags)
        
    - Comment nodes, etc.
        

```ad-note
Whitespace between elements = text nodes

- In the HTML source, there is a **newline/indentation** between `<body>` and `<div>`.
    
- The DOM preserves that whitespace as a **text node**.
```

- `firstChild` / `lastChild`
    
    - `firstChild` gives the first node in `childNodes` (if any).
        
    - `lastChild` gives the last node in `childNodes` (if any).
        
    - They are just shortcuts:
        
        ```js
        elem.firstChild === elem.childNodes[0]
        elem.lastChild === elem.childNodes[elem.childNodes.length - 1]
        ```
        
- `hasChildNodes()`  
    Utility method to check whether `elem.childNodes` contains any nodes (i.e. whether the node has children of any kind).
    

#### Important details & examples

- **Text nodes & whitespace count**  
    Whitespace (new lines, spaces between elements) and text are counted as nodes in `childNodes`. So you might see “Text” nodes when iterating.
    
- **`childNodes` looks like an array**. 
    But actually it’s not an array, but rather a _ [[DOM Collections|DOM collection]]_ – a special array-like iterable object.
    
- **Script timing matters**  
    If you try to access children before they are parsed, you may get incomplete collections. For example, a `<script>` inside `<head>` might see `document.body` as `null` because `<body>` hasn’t been parsed yet.
    
- **Example traversal**
    
    ```html
    <html>
      <body>
        <div>Begin</div>
    
        <ul>
          <li><b>Info</b></li>
        </ul>
      </body>
    </html>
    ```
    
    ```js
    // Iterate over all child nodes of document.body
    for (let i = 0; i < document.body.childNodes.length; i++) {
      console.log(document.body.childNodes[i]);
      // Could see: Text node (whitespace), <div>, Text, <ul>, etc.
    }
    
    console.log(document.body.firstChild);
    console.log(document.body.lastChild);
    ```
    

---

### Siblings & Parent in the DOM

#### Key Terms

|Term|Meaning|
|---|---|
|**Sibling nodes**|Nodes that share the same parent. If A and B are siblings, they are both direct children of the same node.|
|**Parent node**|The node immediately above (i.e. wrapping) a given node in the DOM tree.|

#### Navigation Properties

For navigating to siblings and parent in the DOM, there are built-in properties:

|Property|Direction or relationship|Returns|
|---|---|---|
|`parentNode`|Go _up_ one level|The parent node of the current node (could be any node type)|
|`previousSibling`|Go _sideways_ — to the immediately preceding sibling (on the left)|The node immediately before the current one under the same parent; could be an element, text, comment, etc. ([JavaScript.info](https://javascript.info/dom-navigation "Walking the DOM"))|
|`nextSibling`|Go _sideways_ — to the immediately following sibling (on the right)|The node immediately after the current one under the same parent; again could be any node type. ([JavaScript.info](https://javascript.info/dom-navigation "Walking the DOM"))|

#### Examples

- The `<head>` and `<body>` elements in a standard HTML document are **siblings**, since both are children of `<html>`. 
    
- Some navigations:
    
    ```js
    // Parent:
    document.body.parentNode === document.documentElement;  // true
    ```
    
    ```js
    // Next sibling:
    document.head.nextSibling;  // likely HTMLBodyElement in usual DOM layout
    ```
    
    ```js
    // Previous sibling:
    document.body.previousSibling;  // likely HTMLHeadElement
    ```
    

#### Notes & Caveats

- **Node types matter**: `nextSibling` / `previousSibling` can return nodes of any type — text, comment, etc., not just elements. This means you might get whitespace or comment nodes. 
    
- If there’s no sibling in that direction, `nextSibling` or `previousSibling` is `null`.
    
- If there’s no parent (for some nodes), `parentNode` might be `null`. For example, for the root element (`<html>`), its `parentNode` is `document`, but `parentElement` would be `null` (see element-only navigation). 
    

</br>

----

</br>

## Element-only Navigation in the DOM

When navigating the DOM, sometimes we only care about **element nodes**—the tags—not text nodes, whitespace, or comments. The DOM provides special properties for that.

![[Pasted image 20250918144430.png|center|400]]

---

### Why use element-only navigation

- Standard navigation (`childNodes`, `firstChild`, `nextSibling`, etc.) includes **all node types** (elements, text nodes, comments, whitespace).
    
- Often you only want to traverse HTML elements (tags), so element-only properties make code cleaner and less error-prone.
    

---

### Element-only navigation properties

|Property|Relation / Direction|What it returns|
|---|---|---|
|`children`|Child elements only|All _element-node_ children of an element.|
|`firstElementChild`|First child element|The first child that is an element (ignores text, comments).|
|`lastElementChild`|Last child element|The last child that is an element.|
|`nextElementSibling`|Next sibling element|The next sibling that is an element (ignores non-element nodes).|
|`previousElementSibling`|Previous sibling element|The previous sibling that is an element.|
|`parentElement`|Parent as element|The parent, but _only if it’s an element_. Otherwise `null`.|

---

### Differences vs “all-node” navigation

- **`parentNode` vs `parentElement`**  
    `parentNode` returns the parent node of any type. `parentElement` returns the parent _only if it's an element node_.  
    Example: the `<html>` element’s `parentNode` is the `document`, but its `parentElement` is `null`, because `document` is not an element.
    
- **First/Last/Next/Previous with “Element”** ignore non-element nodes (text, comments, whitespace). This prevents accidental navigation into unwanted nodes.
    

---

### Examples

```html
<html>
  <body>
    <div>Begin</div>

    <ul>
      <li>Information</li>
    </ul>

    <div>End</div>
  </body>
</html>
```

- `document.body.children`  
    → returns only the element children: `DIV`, `UL`, `DIV`.
    
- `document.body.firstElementChild`  
    → the `<div>Begin</div>` element.
    
- `document.body.lastElementChild`  
    → the last `<div>End</div>` in this case. 
    
- `someElem.nextElementSibling` / `someElem.previousElementSibling`  
    Moves between siblings that are **elements only**. Non-element siblings are skipped.
    