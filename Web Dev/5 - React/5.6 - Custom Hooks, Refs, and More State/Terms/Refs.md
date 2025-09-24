---
tags: 
 - react
 - term
 - hook
 - fundamental
---

![[Pasted image 20250910200322.png|center|700]]

## 1. What is `useRef`?

- A React Hook:
    
    ```js
    const ref = useRef(initialValue);
    ```
    
- Returns a **mutable object** with a single property:
    
    ```js
    { current: initialValue }
    ```
    
- The returned object stays the **same between renders**. React does not replace it.
    

---

## 2. Key Features

- **Persistent container**: The `ref.current` value survives re-renders.
    
- **Does not trigger re-render**: Updating `ref.current` does **not** cause the component to render again.
    
- **Same object identity**: Unlike state, the `ref` object itself is stable (React never recreates it).
    

---

## 3. Common Uses

### a) Accessing DOM elements

Attach `ref` to a JSX element:

```jsx
function App() {
  const inputRef = useRef(null);

  function focusInput() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}
```

👉 Here, `inputRef.current` points to the actual DOM node.

---

### b) Storing mutable values (not for rendering)

```js
const renderCount = useRef(0);
renderCount.current += 1;
console.log("Renders:", renderCount.current);
```

- Useful for tracking values across renders without triggering updates.
    

---

### c) Storing timeout/interval IDs

```js
const timerId = useRef(null);

useEffect(() => {
  timerId.current = setInterval(() => console.log("tick"), 1000);
  return () => clearInterval(timerId.current);
}, []);
```

---

### d) Avoiding stale closures

```js
function Timer() {
  const savedCallback = useRef();

  function callback() {
    console.log("tick");
  }

  useEffect(() => {
    savedCallback.current = callback;
  });

  useEffect(() => {
    const id = setInterval(() => savedCallback.current(), 1000);
    return () => clearInterval(id);
  }, []);
}
```

👉 This pattern ensures the interval always has the latest callback without resetting the interval.

---

## 4. Difference from `useState`

|`useState`|`useRef`|
|---|---|
|Stores state that affects rendering|Stores data that doesn’t affect rendering|
|Updating triggers re-render|Updating does **not** trigger re-render|
|Best for UI data|Best for DOM refs, timers, mutable values|

![[Pasted image 20250910200557.png|center|700]]

---

## 5. Gotchas

- Don’t read/write `ref.current` during render _to affect output_. That breaks React’s pure rendering model.
    
- Use `useRef` for imperative actions, not declarative state.
    
- Changing `ref.current` won’t automatically notify React — if UI needs to update, use `useState`.
    

---

## 6. Analogy

Think of `useRef` as a **sticky note** you can attach to your component:

- It doesn’t change the component’s shape (like state would).
    
- It just keeps a piece of data hanging around as long as the component lives.
    

---

✅ In short:  
`useRef` gives you a **persistent, mutable container** that doesn’t trigger renders. Perfect for DOM access, keeping values across renders, and storing things React shouldn’t re-render for.