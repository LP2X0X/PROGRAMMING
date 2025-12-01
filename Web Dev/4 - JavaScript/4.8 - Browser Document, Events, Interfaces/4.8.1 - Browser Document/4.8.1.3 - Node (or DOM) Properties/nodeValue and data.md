---
tags: 
 - js
 - DOM
 - node
 - property
---

The `innerHTML` property is only valid for element nodes.

Other node types, such as text nodes, have their counterpart: `nodeValue` and `data` properties. These two are almost the same for practical use, there are only minor specification differences. So we’ll use `data`, because it’s shorter.

### 1. **`nodeValue`**

- **Definition:** `nodeValue` is a property of a **Node** object. It represents the **text content** of that node, but it only makes sense for certain types of nodes:
    
    - `Text` nodes
        
    - `Comment` nodes
        
    - `Attribute` nodes
        
- For other node types (like `Element` nodes), `nodeValue` is `null`.
    
- **Example:**
    

```javascript
let textNode = document.createTextNode("Hello");
console.log(textNode.nodeValue); // "Hello"

let commentNode = document.createComment("This is a comment");
console.log(commentNode.nodeValue); // "This is a comment"
```

---

### 2. **`data`**

- **Definition:** `data` is **essentially the same as `nodeValue`**, but it exists specifically on **Text**, **Comment**, and **CDATASection** nodes. It is more descriptive because you’re accessing the **actual content** of a text-like node.
    
- **Example:**
    

```javascript
let textNode = document.createTextNode("Hello");
console.log(textNode.data); // "Hello"

// You can also change it
textNode.data = "Hi there";
console.log(textNode.nodeValue); // "Hi there"
```

---

### ✅ Key Differences

|Property|Available on|Can be set?|Notes|
|---|---|---|---|
|`nodeValue`|Node (Text, Comment, Attr, etc.)|Yes|`null` for Element nodes|
|`data`|Text, Comment, CDATASection|Yes|Preferred for text nodes; same as `nodeValue`|

**Summary:**

- `nodeValue` is the general property of a Node that holds the node’s value.
    
- `data` is a more specific, convenient property for text-like nodes. Changing one updates the other.
    