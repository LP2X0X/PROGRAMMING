---
tags: 
 - react
 - hook
 - memoization
---

This explanation will cover **concept → mechanism → internal behavior → pitfalls → best practices**.

---

# 🧠 useMemo — Complete Deep Dive

---

## 🚀 1. What Is `useMemo`?

> `useMemo` is a **React Hook** that lets you **memoize (cache) the result** of an expensive computation between re-renders. 
> It helps **reduce wasted renders indirectly** by ensuring that values passed to child components (or used inside the component) are **referentially stable**, avoiding unnecessary updates when nothing actually changed.

### ✅ Syntax:

```jsx
const memoizedValue = useMemo(() => computeSomething(a, b), [a, b]);
```

- `computeSomething(a, b)` → your expensive calculation function
    
- `[a, b]` → dependency array
    
- React will **only re-run** the computation when **dependencies change**.  
    Otherwise, it **returns the cached value** from last render.
    

---

## ⚙️ 2. What Problem Does It Solve?

Without `useMemo`, every render recalculates the value, even when inputs didn’t change.

Example:

```jsx
const expensiveValue = computeExpensiveValue(a, b);
```

👉 On every render, `computeExpensiveValue` runs again, even if `a` and `b` are unchanged.  
This can slow things down.

✅ With `useMemo`, React remembers the previous result:

```jsx
const expensiveValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

Now, React:

- **Caches** the previous result
    
- **Skips recomputation** until `a` or `b` changes
    

---

## ⚙️ 3. Under the Hood — How React Stores It

Internally, React keeps a **memoized value table** (per component instance):

```
useMemo index → { dependencies, cachedResult }
```

Every render, React:

1. Checks the new dependency array vs the previous one (shallow compare).
    
2. If **dependencies are same**, returns the cached result.
    
3. If **different**, executes the function again and updates the cache.
    

> This mechanism is identical to `useEffect`’s dependency tracking.

---

## 🧮 4. Real Example

```jsx
function App({ list }) {
  const [filter, setFilter] = useState("");

  // 🔥 Expensive computation
  const filteredList = useMemo(() => {
    console.log("Filtering...");
    return list.filter((item) => item.includes(filter));
  }, [list, filter]);

  return (
    <>
      <input value={filter} onChange={(e) => setFilter(e.target.value)} />
      <List items={filteredList} />
    </>
  );
}
```

🧩 What happens:

- On every keystroke → component re-renders
    
- Without `useMemo`, the `.filter()` runs every time
    
- With `useMemo`, React re-filters **only when `list` or `filter` changes**
    

---

## 🧩 5. `useMemo` vs `useCallback`

|Hook|Returns|Use case|
|---|---|---|
|`useMemo`|**Memoized value**|Cache computed result|
|`useCallback`|**Memoized function**|Cache function reference|

```jsx
const cachedValue = useMemo(() => compute(a, b), [a, b]);
const cachedFn = useCallback(() => compute(a, b), [a, b]);
```

You often use `useCallback` with `React.memo` to prevent child re-render  
(because function identity is stable).

---

## 💥 6. Common Pitfalls

|Pitfall|Why it’s a problem|
|---|---|
|❌ Omitting dependencies|Value never updates correctly|
|❌ Using too many `useMemo`s|Adds complexity + minimal performance gain|
|❌ Using for simple values|Waste of memory and CPU to track dependencies|
|❌ Expecting it to “freeze” object|It only memoizes per render, not globally|
|❌ Using inside conditional|Violates hook rules|

---

## 📦 7. Correct Dependency Behavior

### Example 1 — Works

```jsx
const result = useMemo(() => a + b, [a, b]);
```

### Example 2 — Missing dependency ❌

```jsx
const result = useMemo(() => a + b, [a]); // b missing
```

👉 React will skip recomputation even when `b` changes  
➡ leads to stale (incorrect) result

✅ Always include every variable used inside the function.

---

## 🧱 8. Returning Stable Object or Array References

Sometimes, `useMemo` is used **not for heavy computation**,  
but to **preserve object identity** (to prevent re-renders).

```jsx
const options = useMemo(() => ({ theme: "dark" }), []);
return <Child options={options} />;
```

Without `useMemo`, this would re-create a new object on each render  
→ cause child (using `React.memo`) to re-render unnecessarily.

---

## ⚙️ 9. `useMemo` Inside Component Lifecycle

1. On **first render** → function runs, value stored.
    
2. On **subsequent renders** → dependencies compared:
    
    - If unchanged → cached result reused.
        
    - If changed → recomputed and stored again.
        
3. On **unmount** → memory cleared (React GC cleans it up).
    

---

## 🧩 10. Example — Stabilizing Child Props

```jsx
function Parent({ theme }) {
  const config = useMemo(() => ({ theme }), [theme]);
  return <Child config={config} />;
}

const Child = React.memo(function ({ config }) {
  console.log("Render Child");
  return <div>{config.theme}</div>;
});
```

🧩 Without `useMemo`, `config` object changes reference on every render →  
Child re-renders unnecessarily.

✅ With `useMemo`, same reference reused → child skips re-render.

---

## 🧠 11. When NOT to Use It

You **don’t need** `useMemo` when:

- The computation is _cheap_ (e.g. string concatenation)
    
- Component rarely re-renders
    
- Dependencies change frequently (caching useless)
    
- You’re just trying to “optimize everything”
    

> Overuse can **hurt** performance because dependency comparison has its own cost.

---

## 🔬 12. Example — Real Performance Case

```jsx
const sortedData = useMemo(() => {
  console.log("Sorting large data...");
  return data.sort((a, b) => a.value - b.value);
}, [data]);
```

✅ If `data` rarely changes → huge performance gain  
❌ If `data` changes every render → waste of memoization overhead

---

## 🧭 13. useMemo vs Computation in Render

|Case|useMemo needed?|
|---|---|
|Large list filter/sort|✅ Yes|
|Derived UI label (`count * 2`)|❌ No|
|Stable object for child props|✅ Yes|
|Simple condition check|❌ No|

---

## 🧮 14. Summary Table

|Concept|Description|
|---|---|
|Hook type|Value memoization|
|Returns|Cached result of function|
|Input|Function + dependency array|
|Comparison type|Shallow|
|When recomputes|When any dependency changes|
|When useful|Heavy computations or stable references|
|Related hook|`useCallback` (for memoizing functions)|

---

## 🧩 15. Example Summary Code

```jsx
function Example({ items, theme }) {
  const sorted = useMemo(() => {
    console.log("Sorting...");
    return [...items].sort();
  }, [items]);

  const style = useMemo(() => ({ color: theme === "dark" ? "white" : "black" }), [theme]);

  return <Child sorted={sorted} style={style} />;
}
```

✅ Optimized:

- Only sorts when `items` change
    
- Only creates new `style` object when `theme` changes
    
- Prevents unnecessary re-render of `Child`
    