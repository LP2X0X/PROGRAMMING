---
tags: 
 - react
 - memoization
---

### Prerequisite

[[Re-render]]
[[The Context API]]

---
### Overview
![[Pasted image 20251108154528.png|center|700]]

---

## ⚡ What `React.memo` Does

> `React.memo()` is a **higher-order component (HOC)** that **wraps a functional component** and tells React:
> 
> > “💡 Only re-render me if my props actually change.”

It’s similar to `PureComponent` for class components — but for **function components**.

---

## 🧩 Basic Syntax

```jsx
const MyComponent = React.memo(function MyComponent(props) {
  console.log("Render: MyComponent");
  return <div>{props.value}</div>;
});

// Or we could do this...
export default memo(MyComponent);
```

✅ React will skip re-rendering `MyComponent` **if**

- none of its props have changed **(by shallow comparison)**, and
    
- its parent re-rendered **for unrelated reasons**.
    

---

## ⚙️ How It Works (Under the Hood)

When React reconciles the virtual DOM:

1. It compares the **previous props** of a memoized component with the **new props**.
    
2. If the shallow comparison (`===` for primitive values and references) finds **no difference**,  
    React **skips calling the component function again**.
    

> ⚠️ “Shallow” means it only checks top-level values:
> 
> - For objects/arrays/functions → React compares by reference, not by value.
>     

Example:

```jsx
const obj = { a: 1 };

<MyComp data={obj} /> // reference A
<MyComp data={obj} /> // same reference → skip render

<MyComp data={{ a: 1 }} /> // new object → different reference → re-render!
```

---

## 😅 The issue

![[Pasted image 20251108203720.png|center|700]]

---

## 🧠 When Does It Re-Render Anyway?

Even with `React.memo`, React will re-render the component if:

|Cause|Explanation|
|---|---|
|Props change|Shallow comparison finds new value/reference|
|Context changes|Component consumes context that updated|
|State changes|Component manages its own state|
|Parent unmounts/remounts|Entire subtree re-renders|
|Forced re-render (e.g. key change)|React rebuilds new element|

---

## 🧮 Custom Comparison Function

You can customize how props are compared:

```jsx
const MyComponent = React.memo(
  function MyComponent({ user }) {
    console.log("Render: MyComponent");
    return <div>{user.name}</div>;
  },
  (prevProps, nextProps) => prevProps.user.id === nextProps.user.id
);
```

The second argument `(prevProps, nextProps)` should return:

- `true` → skip re-render
    
- `false` → re-render
    

Useful when:

- You want deep comparison for specific props
    
- Or you know which props matter
    

---

## ⚠️ Common Pitfalls

|Mistake|Why It Happens|
|---|---|
|Passing inline functions as props|New reference every render → always triggers re-render|
|Passing new objects or arrays|New reference each time|
|Overusing `memo`|Adds overhead and complexity|
|Forgetting `useCallback` / `useMemo`|Props change unnecessarily due to recreated references|

✅ **Solution:** use `useCallback` or `useMemo` to stabilize props.

Example:

```jsx
const handleClick = useCallback(() => {
  console.log("clicked");
}, []); // stable reference

<Child onClick={handleClick} />
```

---

## 🧩 React.memo + Context

`React.memo` does **not** stop re-renders triggered by context changes.  
If a memoized component **consumes** context that updates → it **must re-render**.

To optimize context-heavy apps, split your context or memoize the provider value.

---

## 🧠 Practical Use Cases

✅ Good places to use `React.memo`:

- Leaf components that render large DOM but don’t often change
    
- Repeated list items (e.g. table rows, todo items)
    
- Presentational (pure) components
    

🚫 Avoid using it:

- On small components with cheap render cost
    
- When props change often (you’ll get little benefit)
    
- As a premature optimization
    

---

## 🧾 Example — See It in Action

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("Alice");

  console.log("Render: Parent");

  return (
    <>
      <button onClick={() => setCount(count + 1)}>Count {count}</button>
      <MemoizedChild name={name} />
    </>
  );
}

const MemoizedChild = React.memo(function Child({ name }) {
  console.log("Render: Child");
  return <div>Child name: {name}</div>;
});
```

🧩 **Behavior:**

- Clicking the button re-renders `Parent`
    
- `Child` will **not re-render**, because its prop `name` didn’t change  
    ✅ → “Render: Child” won’t log again
    

---

## 🧭 Summary Table

|Concept|Meaning|
|---|---|
|`React.memo`|Memoizes functional component output|
|Comparison type|Shallow (default)|
|Custom comparator|`(prevProps, nextProps) => boolean`|
|Context updates|Always re-render consumer|
|Optimization|Prevents unnecessary function re-execution|
|Complementary hooks|`useCallback`, `useMemo`|