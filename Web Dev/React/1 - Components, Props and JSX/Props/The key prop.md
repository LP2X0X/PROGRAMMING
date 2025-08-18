---
tags: react, prop, fundamental
---

## **🔹 What is the key prop?**

- A **special attribute** (not accessible as props.key) that React uses **only internally**. In other words, it is a special prop that we use to tell the diffing algorithm that an element is **unique**.
    
- Its purpose: **help React identify which items in a list have changed, been added, or removed**.
    

It’s essentially an _identity_ for a React element across renders.

---

## **🔹 Why keys matter in diffing**

  
React’s diffing rules (from the context you gave me) rely heavily on **element type + key**:

1. **When a key **stays the same** across renders, the element will be kept in the DOM (_even if the position in the tree changes_)**
    - React assumes it’s the _same element_ → reuses the existing fiber & DOM node.
    - Only updates props/state if needed.
    - This is what lets React **preserve component state** even if the element’s position in the list changes.

```jsx
// Old render
[<Todo key="1" text="A" />, <Todo key="2" text="B" />]

// New render (items swapped)
[<Todo key="2" text="B" />, <Todo key="1" text="A" />]

// React reuses fibers by key → no remount, just reorder
```

  => For performance reason.

---

2. **When a key **changes** between renders, the element will be destroyed and a new one will be created (_even if the position in the tree is the same as before_)**
    - React treats it as a _different element_, destroys the old one, and mounts a new one.
    - This is why a changed key **resets component state**.

```jsx
function Counter({ id }) {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{id}: {count}</button>;
}

// Parent render
<Counter key="A" id="A" />   // count preserved

// Next render
<Counter key="B" id="A" />   // React remounts → count reset to 0
```

  => For the need of change.

---

3. **Keys must be unique among siblings**
    
    - Within the same list, each child should have a unique key.
        
    - Otherwise, React can’t correctly match old vs. new elements.
        
    - Keys don’t need to be globally unique — only among siblings.
        
    

---

## **🔹 Practical Usage**

- **In lists:** Always use stable, unique identifiers (like database IDs).
    

```jsx
{todos.map(todo => (
  <Todo key={todo.id} {...todo} />
))}
```

- **Avoid using array indexes** as keys unless the list is truly static (no reordering, inserting, or deleting). Index-based keys cause incorrect reuse and lead to weird UI bugs.
    

---

## **🔹 Why this ties into Fiber**

  
- The key prop directly affects **how React creates or reuses fiber nodes**:
	- Same key + same type → reuse fiber → update work only.
	- Different key (or missing key) → create new fiber → old one marked for deletion.

So the **key is React’s way of telling the diffing algorithm what counts as “the same work unit”**.

---

✅ **In short:**

The key prop doesn’t show up in props. It’s metadata for React’s diffing. It ensures stability of elements across renders, controls whether state is preserved or reset, and helps React do efficient DOM updates.