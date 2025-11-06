---
tags: 
 - react
 - term
 - render
---

## ⚡️ What Is a Wasted Render?

A **wasted render** (also called an **unnecessary re-render**) happens when a component **renders again even though its visible output (UI)** doesn’t change.

In other words — React spends CPU time reconciling and building the virtual DOM tree **without any visible benefit**.

---

## 🧠 Why Wasted Renders Happen

React re-renders components when:

1. **Parent re-renders** → child re-renders (even if props are the same).
    
2. **State updates** → component re-renders (even if state value didn’t actually change).
    
3. **Context updates** → all consumers re-render (unless optimized).
    
4. **Anonymous functions or new object/array references** are passed as props each render.
    
5. **Inline component definitions** (e.g. defining a component inside another component) cause new references every render.
    

---

## 🔍 Example

```jsx
function Child({ user }) {
  console.log("Child render");
  return <div>{user.name}</div>;
}

function Parent() {
  const [count, setCount] = useState(0);
  const user = { name: "Alice" }; // 👈 new object every render!

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <Child user={user} />
    </>
  );
}
```

💥 Every time you click the button:

- Parent re-renders
    
- A **new `user` object** is created
    
- Child re-renders even though the UI didn’t change
    

This is a **wasted render**.

---

## 🧩 How to Avoid Wasted Renders

### 1. ✅ Use `React.memo`

```jsx
const Child = React.memo(function Child({ user }) {
  console.log("Child render");
  return <div>{user.name}</div>;
});
```

→ Skips re-render if props are the same (shallow compare).

---

### 2. ✅ Memoize values and functions

Use `useMemo` and `useCallback` to **preserve references**:

```jsx
const user = useMemo(() => ({ name: "Alice" }), []);
const handleClick = useCallback(() => console.log("Click"), []);
```

---

### 3. ✅ Avoid updating state with same value

```jsx
setCount(prev => prev === 5 ? prev : prev + 1);
```

Prevents React from scheduling redundant renders.

---

### 4. ✅ Split Context or use selectors

Large contexts cause all consumers to re-render.  
Use context splitting or libraries like **Zustand** / **Jotai** that support granular updates.

---

### 5. ✅ Avoid inline functions or objects in JSX

Bad 👇

```jsx
<Child onClick={() => setCount(c => c + 1)} />
```

Better 👇

```jsx
const handleClick = useCallback(() => setCount(c => c + 1), []);
<Child onClick={handleClick} />
```

---

## 🧮 Summary

|Cause|Fix|
|---|---|
|Parent re-renders|Use `React.memo` on children|
|New object/array/function props|Use `useMemo` / `useCallback`|
|Context re-renders|Split context or use selectors|
|State updates with same value|Guard state updates|
|Inline definitions|Define outside render|

---

## 🚀 Final Note

Wasted renders **don’t always mean a performance issue** — React is fast and handles small re-renders well.  
But understanding and reducing them is crucial in **large apps**, **lists**, or **frequently updated UIs** to keep performance smooth.