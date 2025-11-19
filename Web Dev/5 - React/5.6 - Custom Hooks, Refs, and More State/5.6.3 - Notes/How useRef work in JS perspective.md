---
tags: 
 - react
 - js
 - ref
 - perspective
---

Here is the explanation **purely from JavaScript syntax and behavior**, not React:

---

# ✅ **How React refs work — explained _only using JS syntax + mechanics_**

A React ref is basically **an object whose `.current` property React will mutate for you**.

The primitive looks like this:

```js
const ref = { current: null };
```

When you call:

```js
const inputRef = useRef();
```

React returns an object:

```js
{
  current: undefined   // or null initially
}
```

Then React does this behind the scenes:

- When the component renders and the `<input>` attaches,
    
- React sets:
    

```js
inputRef.current = DOMElementInstance;
```

So syntactically, `ref.current` is **just a normal object property**.

There is nothing special in JavaScript.  
React just fills it for you.

---

# 🧠 **Why does it work like that? (From JS perspective)**

### 1️⃣ **Objects keep identity across re-renders**

JS objects are _reference types_.

So:

```js
const ref = useRef();
```

On every re-render:

- `ref` is **the same object**
    
- only `ref.current` changes
    

This is why refs can “store” values without triggering renders.

### 2️⃣ **React mutates `.current` directly**

JS allows mutation of object properties anytime:

```js
obj.key = value;
```

React uses this to set the DOM element:

```js
inputRef.current = inputNode;
```

This mutation **does not cause the component to re-render**, because:

- setting `.current` is just a normal JS mutation
    
- React does not track it like state
    

This is _exactly_ why refs are used for values you don’t want to cause renders.

---

# 📌 **Ref = A “container object” that React fills in**

JS perspective:

```js
// Think of refs like a box:
const ref = { current: null };

// React puts something inside the box later
ref.current = domNode;
```

Nothing magical.  
Just an object containing a property.

---

# 🎯 **When to use refs (JS-based explanation)**

Use a ref when you need a **stable object across renders** to hold something.

### Use refs when:

1. **You want to store mutable data without causing re-renders**
    
    ```js
    ref.current = someValue;
    ```
    
2. **You want to access a DOM element directly**
    
    ```js
    inputRef.current.focus();
    ```
    
3. **You need instance-like variables (like in OOP classes)**  
    Refs work like:
    

```js
this.somethingInAClass
```

Because the ref object stays the same through renders.

---

# 🚫 **When NOT to use refs**

- Not for UI state
    
- Not for anything that affects rendering
    
- Not for derived values
    

If your value should update the UI → use state.  
If your value is just stored for later use → use ref.

---

# 🧩 **Why not just use a variable?**

You might ask:

```js
let value = 0;
```

Why does this reset on every render?

Because your function re-runs:

```js
function Component() {
  let value = 0; // recreated every render
}
```

But objects returned from hooks:

```js
const ref = useRef();
```

are stored outside the function body, so they survive re-renders.

---

# ⚡ Final summary — JS syntax view

- `useRef()` returns **an object**: `{ current: <initial> }`
    
- This object **never changes identity** across renders
    
- React mutates the `.current` property
    
- Mutating `.current` does **not** trigger re-renders (normal JS mutation)
    
- Use it like a "persistent box" that survives renders
    