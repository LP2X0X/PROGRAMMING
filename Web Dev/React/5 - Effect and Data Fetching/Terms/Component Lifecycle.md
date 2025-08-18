---
tags: react, note, advance
---

![[Pasted image 20250818183604.png|center]]

In **React**, a _component lifecycle_ describes the series of phases a component goes through from its creation (mounting), updating (when state or props change), to its removal (unmounting).


## **🔹 Function Component Lifecycle (with Hooks)**

Function components don’t have lifecycle methods, but **React Hooks** let you achieve the same.

- **Mounting & Updating:** useEffect(() => { ... }) → runs after every render by default. You can control with dependency array:
	- useEffect(() => { ... }, []) → runs once (mount).
	- useEffect(() => { ... }, \[someVar]) → runs when someVar changes.
- **Unmounting (cleanup):**
    Return a cleanup function from useEffect:

```jsx
useEffect(() => {
  const id = setInterval(() => console.log("tick"), 1000);

  return () => clearInterval(id); // cleanup on unmount
}, []);
```

---

## **🔹 Visual Summary**

```jsx
Mounting → Updating → Unmounting
  useEffect(() => {...}, [])         // mount
  useEffect(() => {...}, [deps])     // update when deps change
  useEffect(() => {... return ...})  // cleanup on unmount
```