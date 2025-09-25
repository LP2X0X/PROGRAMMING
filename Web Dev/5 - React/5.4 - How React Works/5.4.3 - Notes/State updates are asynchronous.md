---
tags: 
 - react
 - note
 - advance
---

## 1. **Performance / Efficiency**

- If React updated the state **immediately** every time you called `setState` (or `setX` in hooks), React would need to **re-render** the component tree right away.
    
- Imagine this:
    
    ```js
    setCount(count + 1);
    setName("Alice");
    setTheme("dark");
    ```
    
    If each one triggered a full render immediately, that’s **3 renders back-to-back** → wasteful.
    

✅ Instead, React marks the state updates as _pending_, collects them, and processes them **together in a batch** before the next render.

---

## 2. **Consistency with Virtual DOM rendering**

- React works in _render cycles_. A render should be based on a **stable snapshot** of the state/props.
    
- If React allowed state to change mid-cycle, the UI could become inconsistent:
    
    - One part of the tree sees the "old" state.
        
    - Another part sees the "new" state.  
        → That would break React’s predictable rendering model.
        

So React defers applying updates until it’s safe to re-render everything consistently.

---

## 3. **Asynchronous scheduling**

- Since React 18, updates can be scheduled with **priority** (via the Concurrent Mode model).
    
- For example:
    
    - User typing = high priority → React processes it quickly.
        
    - Background data refresh = lower priority → React may delay rendering until the browser is free.
        
- This requires React to **treat state updates as async tasks**, not immediate mutations.
    

---

## 4. **Developer Expectation (Gotcha)**

Because updates are async, if you log right after `setState`, you’ll still see the **old value**:

```js
const [count, setCount] = useState(0);

function handleClick() {
  setCount(count + 1);
  console.log(count); // 😲 still the old count
}
```

👉 This doesn’t mean React didn’t “hear” the update — it just hasn’t _applied_ it yet.  
On the next render, the new state will be there.

✅ If you need to act **right after** the state changes, you either:

- Use a `useEffect` that depends on that state.
    
- Or use the functional updater form:
    
    ```js
    setCount(prev => prev + 1);
    ```
    

---

### 🔑 Summary:

State updates are asynchronous because:

1. React batches updates for **performance**.
    
2. It keeps rendering **consistent and predictable**.
    
3. It allows **scheduling & prioritization** of updates.
    