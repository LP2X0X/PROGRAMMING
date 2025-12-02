---
tags: 
 - js
 - html
 - modify
 - document
 - DOM
---

## ✅ Ways to **Create** New Content (Nodes)

You can create new DOM nodes (elements or text nodes) with:

- `document.createElement(tag)` — creates a new element node. 
    
- `document.createTextNode(text)` — creates a text node with given text. 
    

Example:

```js
let div = document.createElement('div');
let text = document.createTextNode('Hello world');
div.append(text);
```


Often you create an element (e.g. `<div>`) and then fill it with content/text or sub-elements. 

---

## 👉 Inserting / Inserting + Removing / Replacing Nodes

Once you have a node (element or text), you need to insert it into the document (or remove / move / replace). The modern APIs you should know:

- `node.append(...nodes or strings)` — add nodes/text at the _end_ of `node`. 
    
- `node.prepend(...nodes or strings)` — add at the _beginning_. 
    
- `node.before(...nodes or strings)` — insert right _before_ `node`. 
    
- `node.after(...nodes or strings)` — insert right _after_ `node`. 
    
- `node.replaceWith(...nodes or strings)` — replace `node` with given nodes/text. 
    
- `node.remove()` — remove `node` from DOM.
    

![[Pasted image 20251202150526.png|center|500]]

```ad-note
These methods accept actual DOM nodes or strings. If strings are provided, they get inserted as **text nodes** (i.e. special characters like `<`, `>` are escaped — they won’t be interpreted as HTML). 
```

---

## 🔄 Inserting HTML as HTML (not just text)

If you want to insert an HTML fragment (string that contains HTML tags) and have it parsed as real HTML — not just literal text — there is:

- `elem.insertAdjacentHTML(where, htmlString)` — this lets you insert HTML at positions relative to `elem`. 
    

Possible values for `where`:

- `"beforebegin"` — before the element itself.
    
- `"afterbegin"` — as first child inside element.
    
- `"beforeend"` — as last child inside element.
    
- `"afterend"` — right after the element. 
    

![[Pasted image 20251202150800.png|center|500]]

There are also analogous methods for text (`insertAdjacentText`) and for inserting a single element/node (`insertAdjacentElement`), though they are used less often because the `append` / `prepend` / `before` / `after` methods already cover most use cases. 

---

## Node removal

- To remove a node, there’s a method node.remove(). Let’s make our message disappear after a second:

```html
<style>
.alert {
  padding: 15px;
  border: 1px solid #d6e9c6;
  border-radius: 4px;
  color: #3c763d;
  background-color: #dff0d8;
}
</style>

<script>
  let div = document.createElement('div');
  div.className = "alert";
  div.innerHTML = "<strong>Hi there!</strong> You've read an important message.";

  document.body.append(div);
  setTimeout(() => div.remove(), 1000);
</script>
```

````ad-note
Please note: if we want to move an element to another place – there’s no need to remove it from the old one. All insertion methods automatically remove the node from the old place. For instance, let’s swap elements:
```html
<div id="first">First</div>
<div id="second">Second</div>
<script>
  // no need to call remove
  second.after(first); // take #second and after it insert #first
</script>
```
````

---

## 📦 Cloning and Reusing Nodes

- `elem.cloneNode(true)` — creates a **deep** clone (clone element + all its subelements). 
    
- `elem.cloneNode(false)` — creates a **shallow** clone (just the element itself, no children). 
    

Cloning is useful when you need duplicates of a complex element, perhaps to reuse templates or create multiple similar items. 

---

## 🧪 Efficient Bulk Insertion: DocumentFragment

- `DocumentFragment` is a lightweight DOM node that doesn’t belong to the main document tree. You can build a batch of nodes inside it, then insert the fragment — all child nodes are appended, but the fragment itself disappears. 
    
- This reduces overhead compared to multiple individual insertions (better performance). 
    

Example:

```js
let frag = new DocumentFragment();
for (let i = 1; i <= 3; i++) {
  let li = document.createElement('li');
  li.textContent = i;
  frag.append(li);
}
ul.append(frag); // all LIs get inserted at once
```

---

## 🕰️ “Old-school” Methods (Still Worth Knowing, but Less Used)

These are older DOM manipulation methods. They work, but modern methods above are more flexible and cleaner.

- `parent.appendChild(node)` — adds node as last child. 
    
- `parent.insertBefore(newNode, referenceNode)` — inserts before referenceNode. 
    
- `parent.removeChild(node)` — remove a child node. 
    
- `parent.replaceChild(newNode, oldNode)` — replace oldNode with newNode. 
    

All these return the node inserted or removed.

These are mostly of historical interest (or useful when you need compatibility with older browsers). 

---

## ⚠️ Ancient Method: document.write

- `document.write(htmlString)` writes HTML directly to the page at parse-time. 
    
- **Big limitation:** If you call it after the page has loaded (i.e. dynamically, after DOM ready) — it will **erase the entire document** and write new contents. 
    
- Therefore: rarely used nowadays, only seen in legacy scripts. 
    

---

## 📋 Additional Notes / Comparison: `createTextNode` vs `innerHTML` vs `textContent`

- If you have plain text (possibly containing characters like `<`, `>`), and you want to insert it safely (without treating it as HTML), you can use either:
    
    - `elem.append(document.createTextNode(text))`, or
        
    - `elem.textContent = text`.
        
- These two are effectively equivalent: both insert text nodes safely (no HTML parsing / execution). 
    
- Using `elem.innerHTML = text` however will treat `text` as HTML — risky if the text comes from users. 
    

---

## 🚀 When to Use What — Practical Guidance

- Use **DOM methods** (`createElement`, `append`, `prepend`, `before`, `after`, etc.) when you build structure piece by piece. This is more explicit, safer, and easier to maintain.
    
- Use **cloning** (via `cloneNode`) when you need multiple similar subtrees.
    
- Use **`DocumentFragment`** when inserting many nodes at once (better performance).
    
- Use **`insertAdjacentHTML`** if you have a big HTML chunk ready as a string and you trust or sanitize it (because it inserts as actual HTML).
    
- Avoid `document.write` in modern development.
    
- For text insertion — prefer `textContent` or `createTextNode`, not `innerHTML`, if the content may include arbitrary user input / needs to be safe.
    