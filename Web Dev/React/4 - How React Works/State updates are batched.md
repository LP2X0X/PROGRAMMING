---
tags: react, note, advance
---

- When you call multiple setState (or useState setters) **inside the same function execution context**, React **groups them into one render** instead of re-rendering for each call.
![[Pasted image 20250817183148.png|center]]

---

## **🔹 1. Batching is per render pass, not per component**

- When React batches, it doesn’t re-render _each component separately_. Instead, React:
	1. Collects **all state updates** that happen in the same event/task.
	2. Figures out the **root fiber tree(s)** that are affected.
	3. Runs **one render (reconciliation) pass per root**.
	4. During that render, **all affected components** under that root are re-rendered (only the ones whose state/props changed).


So batching → _one render pass for the whole root tree_, not separate renders per component.

---

## **🔹 2. Example**

```jsx
function A() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>A {count}</button>;
}

function B() {
  const [flag, setFlag] = useState(false);
  return <button onClick={() => setFlag(f => !f)}>B {String(flag)}</button>;
}

function App() {
  return (
    <>
      <A />
      <B />
    </>
  );
}
```

Case:

```jsx
// Inside the same React event handler:
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
}
```

- React batches → schedules **one render for the App root**.
    
- During that render:
    
    - Fiber for \<A> re-renders (because count changed).
        
    - Fiber for \<B> re-renders (because flag changed).
        
    
- Other components not affected won’t re-render.
    

---

## **🔹 3. Async Case (React 18+)**

```jsx
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
}, 0);
```

- React 18 → still batched, so only **one render of the root**.
    
- Both \<A> and \<B> get re-rendered in that pass.
    

---

## **🔹 4. Important Note**

- React does **not** re-render the whole tree blindly.
- Only components whose state/props/context changed get re-rendered.
- But those re-renders happen within **the same render pass per root** when batched.
- State update is asynchronous.

```ad-important
If we need to update state based on previous update, we use setState with callback.
```

