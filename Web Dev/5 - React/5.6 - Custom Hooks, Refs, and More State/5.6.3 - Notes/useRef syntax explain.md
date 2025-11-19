---
tags: 
 - react
 - use
 - ref
 - note
---

Here is **exactly how the syntax works**, purely from JavaScript + JSX transformation — no magic.

---

# ✅ 1. JSX is just a function call

This JSX:

```jsx
<input ref={inputRef} />
```

gets compiled into pure JS:

```js
React.createElement("input", { ref: inputRef });
```

So the `ref` is not special syntax —  
it’s **just a property on a props object**.

---

# ✅ 2. What React does with `{ ref: inputRef }`

React receives:

```js
{ ref: inputRef }
```

Now React says:

- If this element is a **DOM element** (“input”), `ref` means:  
    → _“When this DOM node is created, assign it to `ref.current`.”_
    
- If this element is a **custom component**, `ref` normally does not work unless forwarded.
    

### The important part:

`ref` works because React **reserved the prop name `"ref"`** and gives it special treatment.

Just like `"key"` and `"children"`.

So internally React does:

```js
if (props.ref) {
    // Attach DOM node → set ref.current = node
}
```

---

# ✅ 3. How the DOM element attaches

When React creates the actual DOM node:

```js
const el = document.createElement("input");
```

React checks if there is a `ref` prop:

```js
if (typeof inputRef === "object") {
    inputRef.current = el;
}
```

This is the core.

`inputRef` is an object:  
`{ current: null }`

React mutates it:

```js
inputRef.current = el; // DOM node
```

---

# 🧠 Summary in JS-only terms

1. `<input ref={inputRef} />` → becomes  
    `React.createElement("input", { ref: inputRef })`
    
2. The `ref` is just a property on the props object.
    
3. React, internally, looks for `props.ref`.
    
4. When the element mounts, React runs something like:
    

```js
props.ref.current = createdDomNode;
```

5. Now your JS object `inputRef.current` holds the DOM node.
    

---

# 📌 Why the syntax works

Because **React intercepts the `ref` prop**, and interprets it according to its own rules.

JSX does NOT understand refs.  
JavaScript does NOT understand refs.  
DOM does NOT understand refs.

**React does.**