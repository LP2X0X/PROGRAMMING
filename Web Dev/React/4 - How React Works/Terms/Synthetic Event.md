---
tags:
  - react
  - term
  - advance
---

## **🔹 What is a Synthetic Event?**

- In React, event objects (onClick, onChange, etc.) are **not the raw browser events**.
    
- Instead, React wraps the native event in its own **SyntheticEvent** object.
    

  

This wrapper:

- Normalizes event behavior across browsers (consistent API).
    
- Adds optimizations (like event pooling in older React versions).
    
- Integrates with React’s **event delegation system**.
    

---

## **🔹 Why Synthetic Events Exist**

1. **Cross-browser compatibility**
    Different browsers used to expose events differently. React’s SyntheticEvent ensures you always get the same properties (event.target, event.preventDefault(), etc.).
    
2. **Performance via delegation**
    
    - Instead of attaching handlers directly to each DOM node, React attaches **one listener per event type at the root** (e.g., document).
        
    - When an event happens, React figures out which component(s) should receive it (using Fiber tree).
        
    - This reduces memory usage and speeds up mounting/unmounting.
        
    
3. **Controlled lifecycle**
    
    - React can decide when events are fired in relation to state updates and batching.
        
    - This helps ensure UI stays predictable during React’s async rendering.
        
    

---

## **🔹 Example**

```jsx
function App() {
  function handleClick(event) {
    console.log(event.type);       // "click"
    console.log(event.nativeEvent); // The real DOM event
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

- event → SyntheticEvent
    
- event.nativeEvent → underlying real browser event
    

---

## **🔹 Event Pooling (React < 17)**

  

In older React, SyntheticEvents were **pooled** (reused for performance).

That meant:

```jsx
function handleClick(e) {
  console.log(e.type);      // works
  setTimeout(() => console.log(e.type)); // ❌ error (event nullified)
}
```

Solution:

- Use e.persist() to opt-out of pooling.
    
- Or capture needed values synchronously.
    

  

In **React 17+**, pooling was removed, so you don’t need to worry about this anymore.

---

## **🔹 Synthetic Events vs Native Events**

|**Feature**|**SyntheticEvent**|**Native DOM Event**|
|---|---|---|
|Cross-browser normalized|✅ Yes|❌ No|
|Event pooling|✅ (<17)|❌|
|Delegated (single root handler)|✅ Yes|❌ Direct per-node|
|Works with React batching/scheduling|✅ Yes|❌ No|

---

✅ **In short:**

Synthetic events are React’s own event system that wraps native events to provide **cross-browser consistency, performance via delegation, and integration with React’s update cycle**.