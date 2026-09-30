---
tags: 
 - react
 - hook
 - memoization
---

## ⚙️ 1. What `useCallback` Really Does

At its core, 👉 **`useCallback(fn, deps)` is just a shorthand for `useMemo(() => fn, deps)`**

So instead of memoizing a **computed value**, you’re memoizing a **function reference**.

React stores:

- your callback function
    
- its dependency array  
    and on every render, it checks if the dependencies have changed.
    

If they haven’t, React gives you back the **same function instance** (same reference in memory).

---

## 🧩 2. Why This Matters

In React, **functions are recreated on every render**:

```js
function Component() {
  const handleClick = () => console.log("clicked!");
}
```

Even if the logic inside `handleClick` doesn’t change, React re-creates a new function object each time.

If that function is passed as a prop to a child component,  
➡️ the child sees a “new” prop on every render,  
➡️ and will re-render unless it’s wrapped in `React.memo`.

`useCallback` prevents that by giving you the **same function reference** between renders — _as long as dependencies don’t change_.

---

## 🧭 3. Internal Mechanism (Just Like useMemo)

When React renders your component:

1. It finds the `useCallback` hook entry for that component instance.
    
2. Compares the new dependency array with the previous one (shallow equality).
    
3. - If dependencies are **the same** → returns the **cached function**.
        
    - If dependencies **changed** → stores and returns the **new function**.
        

You can think of it like this pseudo-cache:

```
useCallback table
-------------------------------------
Index | Dependencies | Cached Function
-------------------------------------
  0   | [a, b]       | handleSubmit()
-------------------------------------
```

---

## ⚡ 4. Analogy with useMemo

| Concept            | `useMemo`                          | `useCallback`                     |
| ------------------ | ---------------------------------- | --------------------------------- |
| Purpose            | Cache a computed **value**         | Cache a **function**              |
| Returns            | The result of a function           | The function itself               |
| Prevents           | Recomputing expensive values       | Re-creating function references   |
| Syntax             | `useMemo(() => compute(), [deps])` | `useCallback(() => fn(), [deps])` |
| Internally same as | ✅                                 | `useMemo(() => fn, [deps])`       |

---

## 🔬 5. When It’s Useful

1. **When passing callbacks to memoized children** (`React.memo`):
    
    ```jsx
    const handleClick = useCallback(() => {
      setCount(c => c + 1);
    }, []);
    
    <Child onClick={handleClick} />  // Child won't re-render unnecessarily
    ```
    
2. **When using dependencies in hooks** (`useEffect`, `useLayoutEffect`, etc.)  
    If you define a function inline, it will always trigger re-runs unless memoized:
    
    ```js
    useEffect(() => {
      fetchData();
    }, [fetchData]); // ❌ re-runs if fetchData re-created each render
    ```
    
    Use `useCallback`:
    
    ```js
    const fetchData = useCallback(() => {...}, []);
    ```
    

---

## 🚫 6. Common Misunderstandings

| Misconception                                       | Reality                                                                                      |
| --------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| “`useCallback` improves performance automatically.” | ❌ It prevents re-renders _only if_ the callback is passed to a memoized component.          |
| “It freezes the function.”                          | ❌ It only reuses the same reference when deps don’t change.                                 |
| “It’s like `bind`.”                                 | ⚠️ No, `bind` permanently ties a function’s `this`. `useCallback` just memoizes a reference. | 
| “It caches the _result_ of the function.”           | ❌ It caches the _function itself_. Use `useMemo` for results.                               |

---

## 🧠 7. Under-the-Hood Model

You can think of React’s internal “callback cache” like this:

```js
const memoizedCallback = depsChanged
  ? newFunction
  : previousFunction;
```

Each render checks the **dependency key**, just like a cache lookup.

---

## 🧩 8. Real-World Example

### Without `useCallback`

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = () => setCount(c => c + 1);

  return <Child onClick={handleClick} />;
}

const Child = React.memo(({ onClick }) => {
  console.log("Child rendered!");
  return <button onClick={onClick}>Click</button>;
});
```

✅ Every time `Parent` re-renders,  
❌ `handleClick` is recreated → new reference → `Child` re-renders.

---

### With `useCallback`

```jsx
const handleClick = useCallback(() => setCount(c => c + 1), []);
```

✅ React reuses the same reference between renders  
✅ `Child` won’t re-render unless it actually needs to

---

## 🧠 9. Summary Mental Model

> **`useCallback` = "remember this function between renders unless dependencies change."**

It works as a **per-component local cache**,  
keyed by the **dependency array**,  
and prevents **new function reference creation**.

---

## ⚡ TL;DR Summary Table

| Concept                  | Description                                         |
| ------------------------ | --------------------------------------------------- |
| Hook type                | Referential memoization                             |
| Stores                   | Function reference                                  |
| Dependency check         | Shallow equality                                    |
| Trigger for new function | Any dependency change                               |
| Prevents                 | Useless child re-renders, effect re-triggers        | 
| Implementation shortcut  | `useCallback(fn, deps)` ≡ `useMemo(() => fn, deps)` |