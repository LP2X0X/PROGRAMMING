---
tags: 
 - react
 - note
---

```ad-note
The function which we talked about in this case mean the prop function (which get passed from parent component), not the set function inside the component which used useEffect itself.
```

### 1. What happens inside your effect

You wrote:

```js
useEffect(() => {
  function callback(e) {
    if (e.code === "Escape") {
      onCloseMovie(); // ⬅️ uses onCloseMovie from props/state/closure
    }
  }

  document.addEventListener("keydown", callback);

  return () => {
    document.removeEventListener("keydown", callback);
  };
}, [onCloseMovie]);
```

Inside the effect, the `callback` function **closes over** whatever `onCloseMovie` was _at the time the effect ran_.

---

### 2. Why dependency is needed

- If `onCloseMovie` ever **changes identity** (for example, parent component re-creates it with new state inside), your callback would still hold on to the _old_ function if you don’t include it in dependencies.
- It is a prop so it can be changed.
- By listing `[onCloseMovie]`, you tell React:
    

> “If `onCloseMovie` changes, tear down the old event listener and set up a new one with the updated callback.”

---

### 3. What happens if you omit it?

- ESLint (React hooks lint rule) will warn you about **missing dependency**.
    
- The effect will keep calling the **stale `onCloseMovie`** (old function reference). That can cause bugs if the function depends on state/props that have since updated.
    

Example:

```js
function Parent() {
  const [count, setCount] = useState(0);

  function onCloseMovie() {
    console.log("Closing with count =", count);
  }

  return <Child onCloseMovie={onCloseMovie} />;
}
```

If you omit `[onCloseMovie]`, then when `count` changes, your `onCloseMovie` in the child’s effect is still pointing to the **old `count`** value.

---

### 4. Summary

- ✅ You must pass `onCloseMovie` as a dependency because your effect _uses it_.
    
- This ensures your event listener always calls the **latest version** of the function.
    
- Without it, you risk bugs due to **stale closures**.
    

---

👉 If you want to avoid re-subscribing every time `onCloseMovie` changes, you can wrap it in `useCallback` in the parent, or use a `ref` trick.

---

# Examples

### 🔴 Case 1: **Without `[onCloseMovie]` dependency**

```jsx
function Parent() {
  const [count, setCount] = React.useState(0);

  function onCloseMovie() {
    console.log("Closing with count =", count);
  }

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Increase</button>
      <Child onCloseMovie={onCloseMovie} />
    </div>
  );
}

function Child({ onCloseMovie }) {
  React.useEffect(() => {
    function callback(e) {
      if (e.code === "Escape") {
        onCloseMovie(); // uses whatever was captured!
      }
    }

    document.addEventListener("keydown", callback);

    return () => document.removeEventListener("keydown", callback);
  }, []); // ❌ Forgot dependency!

  return <p>Press Escape to close</p>;
}
```

#### What happens:

1. Parent renders first time → `count = 0`.  
    `onCloseMovie` logs `0`.
    
2. You click **Increase** → `count = 1`.  
    Parent re-renders, creates a _new_ `onCloseMovie` function that logs `1`.
    
3. But Child’s effect **never re-ran**, because dependency list was empty `[]`.  
    → The event listener is still pointing to the _old_ `onCloseMovie`.  
    → Press Escape → it still logs:
    
    ```
    Closing with count = 0
    ```
    
    ❌ Stale value bug!
    

---

### 🟢 Case 2: **With `[onCloseMovie]` dependency**

```jsx
function Child({ onCloseMovie }) {
  React.useEffect(() => {
    function callback(e) {
      if (e.code === "Escape") {
        onCloseMovie();
      }
    }

    document.addEventListener("keydown", callback);
    return () => document.removeEventListener("keydown", callback);
  }, [onCloseMovie]); // ✅ included dependency
}
```

#### What happens:

1. Parent renders with `count = 0` → Child sets up listener with `onCloseMovie(0)`.
    
2. You click **Increase** → Parent re-renders, new `onCloseMovie(1)`.  
    → Because `[onCloseMovie]` changed, Child tears down old listener and sets up new one.
    
3. Press Escape → now logs:
    
    ```
    Closing with count = 1
    ```
    
    ✅ Always correct, no stale value.
    

---

So the **whole point**:

- React doesn’t magically update closures for you.
    
- The effect has to be re-run with the new `onCloseMovie`.
    
- The dependency array is how React knows when to re-run.
    