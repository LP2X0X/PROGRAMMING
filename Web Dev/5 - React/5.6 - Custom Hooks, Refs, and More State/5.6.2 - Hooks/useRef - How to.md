---
tags: 
 - react
 - ref
 - how
---

# 🚀 **1. What is a ref?**

A **ref** is an object created by React that holds a **mutable `.current` property**.

```js
const ref = useRef(initialValue);
```

It returns a stable object:

```js
{ current: initialValue }
```

React **does NOT track changes** to `ref.current` the way it tracks state.

### Key properties of refs:

- `.current` is mutable
    
- changing `.current` **does not** cause re-render
    
- the same ref object is preserved across renders
    

---

# 🧠 **2. Why refs work (under the hood)**

Refs work because React **keeps the same object between renders**.

When React re-renders a component, it creates fresh variables, fresh closures, **but it does NOT recreate the ref object**.

Ref stability is guaranteed.

Example:

```js
const inputRef = useRef();
```

React ensures:

|Render #1|Render #2|Render #3|
|---|---|---|
|`{ current: … }`|same object|same object|

This persistent reference allows you to:

- store DOM nodes
    
- store values that survive rerenders
    
- store mutable variables that don’t trigger re-render
    

This is why refs are perfect for:

- DOM manipulation
    
- keeping internal state that shouldn’t cause UI changes
    
- storing timers, previous values, etc.
    

---

# 🪜 **3. Steps to Use Refs**

## **Step 1 — Create the ref**

```js
const inputRef = useRef(null);
```

## **Step 2 — Attach it to a DOM element**

```jsx
<input ref={inputRef} />
```

React now sets:

```js
inputRef.current = <actual DOM input element>
```

## **Step 3 — Access `.current`**

```js
inputRef.current.focus();
```

That’s it.

---

# 💡 **4. Why `.current` exists**

React gives you a stable object so you can mutate the value:

```js
ref.current = newValue;
```

But WHY not just give you the raw DOM element?

❌ Because React must preserve consistency across renders  
❌ React must avoid re-renders when refs change  
❌ React needs a container object to mutate safely

Thus:

```js
{ current: value }
```

is the safest pattern.

---

# 🔧 **5. Practical Examples**

## **A. Focusing an input**

```js
function LoginForm() {
  const emailRef = useRef();

  useEffect(() => {
    emailRef.current.focus();
  }, []);

  return <input ref={emailRef} />;
}
```

---

## **B. Storing a mutable value without re-rendering**

```js
const renderCount = useRef(0);
renderCount.current++;
console.log(renderCount.current);
```

State would re-render. Refs do **not**.

---

## **C. Storing a timeout ID**

```js
const timer = useRef();

function start() {
  timer.current = setTimeout(() => {
    console.log("done");
  }, 1000);
}

function cancel() {
  clearTimeout(timer.current);
}
```

---

## **D. Keeping previous props/state**

```js
const prevCount = useRef(count);

useEffect(() => {
  prevCount.current = count;
});
```

---

# 🎯 **6. When to use refs (the correct cases)**

## ✅ **Case 1 — Accessing DOM nodes**

- focus()
    
- scrollIntoView()
    
- measure size (offsetWidth)
    
- play/pause videos
    

## ✅ **Case 2 — Storing mutable values without re-rendering**

Useful for:

- timers
    
- intervals
    
- scroll positions
    
- WebSocket instances
    
- previous values
    
- counters that shouldn’t cause UI updates
    

## ✅ **Case 3 — Avoid triggering effects by storing stable references**

Example: storing a stable event handler or callback.

---

# ❌ **7. When NOT to use refs**

Refs are **NOT** for:

- storing app state
    
- replacing state
    
- reacting to changes (they do NOT trigger re-renders)
    
- deriving UI values
    

Examples of bad usage:

```js
// ❌ trying to create reactive state with refs
ref.current = value;  // UI will NOT update
```

Ref is for _imperative things_, not UI logic.

---

# 🔄 **8. Refs do NOT cause re-renders**

```js
ref.current = 123;
```

does **NOT** re-render the component.

### Why?

Because `.current` is mutable and React ignores its mutation.

React state → triggers re-render  
React ref → does NOT re-render

This design allows refs to behave like:

- “boxes”
    
- “containers”
    
- “escape hatches”
    
- “instance fields in a class”
    

---

# 🧩 **9. Why refs must be stable across renders**

Because:

- effects run after render
    
- event handlers use closures
    
- input DOM element changes
    
- animations or timeouts must persist
    

If React recreated the ref on every render:

- it would break focus
    
- destroy timers
    
- break event subscriptions
    
- cause memory leaks
    

That’s why React preserves the ref object.

---

# 📝 **10. Summary**

### ✔ **Ref is a stable object `{ current: ... }` kept between renders**

### ✔ **Changing `.current` does NOT cause re-render**

### ✔ **Use refs for DOM operations, timers, instance-like variables**

### ✔ **Do NOT use refs for UI state or rendering logic**

### ✔ **Refs are your escape hatch for imperative operations**