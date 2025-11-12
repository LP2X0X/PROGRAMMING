---
tags: 
 - react
 - note
 - memoization
---

## 🧩 Why State Setters Don’t Need `useCallback`

### 🔹 1. State setters are **already stable**

React guarantees that the **state setter function returned by `useState`** is **referentially stable** — meaning its **identity never changes** between renders.

Example:

```jsx
const [count, setCount] = useState(0);
```

Even if the component re-renders hundreds of times,  
👉 `setCount` will **always be the same function reference**.

So:

```jsx
const fn1 = setCount;
const fn2 = setCount;
console.log(fn1 === fn2); // ✅ true every render
```

That’s why wrapping it in `useCallback` gives no benefit.

---

### 🔹 2. `useCallback` is meant for **functions you define**

`useCallback` exists to prevent **function identity changes** for functions created _inside_ a component:

```jsx
const handleClick = () => {
  setCount(c => c + 1);
};
```

Each render re-creates `handleClick`, which means its **reference changes**, possibly causing **child components** to re-render unnecessarily.

That’s when you use:

```jsx
const handleClick = useCallback(() => {
  setCount(c => c + 1);
}, []);
```

But for `setCount` itself — React already ensures its identity is fixed.

---

### 🔹 3. When passing setters to children

If you pass a state setter directly to a child:

```jsx
<Child onClick={setCount} />
```

You **don’t need** `useCallback` either,  
because React will not create a new `setCount` function on every render — it’s the same reference.

If you instead pass a custom function like:

```jsx
<Child onClick={() => setCount(c => c + 1)} />
```

Then it **is** a new function each time → and might need `useCallback` if re-renders matter.

---

### ✅ Summary

|Case|Stable by default?|Needs `useCallback`?|
|---|---|---|
|React state setter (`setCount`)|✅ Yes|❌ No|
|Inline function using setter (`() => setCount(...)`)|❌ No|✅ Yes, if passed to children|
|Event handler defined in component (`function handleClick() { ... }`)|❌ No|✅ Yes, if expensive re-renders|

---

**In short:**

> State setters are already memoized by React.  
> You only need `useCallback` for _functions you create yourself_ that depend on props or state.