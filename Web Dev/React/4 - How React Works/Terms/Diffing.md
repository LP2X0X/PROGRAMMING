---
tags: react, term, fundamental
---

```ad-important
- Diffing uses 2 fundamental assumptions (rules):
	- Two elements of different types will produce different trees.
	- Elements with a stable key prop stay the same across renders.
```

---

## **🔹 What is Diffing?**

Diffing = comparing the **previous React element tree** (from the last render) with the **new React element tree** (from the current render) to figure out the **minimum set of changes** needed for the DOM.


This is a core part of the **reconciliation** process.

---

## **🔹 Why Diffing?**

- Updating the whole DOM is too slow.
    
- Instead, React calculates _what changed_ and applies only those changes.
    

---

## **🔹 React’s Diffing Algorithm (Simplified Rules)**

1. **Different element, same position → different subtrees**
    - If the type changes (e.g. \<div> → \<span>, or ComponentA → ComponentB), React will **tear down** the old tree and build a new one since React assumes the entire sub-tree is no longer valid.
    - Old components are destroyed and removed from DOM, **including state**.

```jsx
<div /> → <span />   // React deletes <div>, creates <span>
```

![[Pasted image 20250817154520.png|center|600]]

---

2. **Same position, same element → reuse the node, update props**
    - If the element type is the same (\<div> → \<div>), React **reuses the DOM node** and only applies prop changes.
    - Element will be kept (as well as child elements), **including state**.
    - New props / attributes are passed if they changed between renders.

```jsx
<button className="red" /> 
→ <button className="blue" />
// React keeps <button>, just updates className
```

![[Pasted image 20250817154440.png|center|600]]

---

3. **Lists need keys (for child diffing)**
    - When children are arrays, React matches them by **key** (if provided) or by index (fallback).

- With keys:
```jsx
// Old
[<li key="A" />, <li key="B" />]

// New
[<li key="B" />, <li key="A" />]

// React sees same keys "A" and "B" → reuses nodes, just reorders
```

- Without keys:
```jsx
[<li>A</li>, <li>B</li>] → [<li>B</li>, <li>A</li>]
// React assumes <li>A</li> was replaced by <li>B</li>, etc.
// Leads to unnecessary deletes + inserts
```

---

## **🔹 Diffing Phases in Fiber**

  

In Fiber, diffing happens during the **render (reconciliation) phase**:

1. **Begin work** → Compare current fiber’s child with the new React element(s).
    
    - Decide which children need to be created, updated, or deleted.
        
    - Create new fibers for new elements.
        
    
2. **Complete work** → Collect side effects (e.g. “this DOM node needs to be updated”).
    
3. **Commit phase** → Apply the changes to the real DOM all at once.
    

---

## **🔹 Efficiency**

  

Naive diffing of two arbitrary trees is **O(n³)** (very slow).

React’s heuristics (assumptions above) reduce this to **O(n)** in practice.

---

✅ **In short:**

Diffing in React = comparing old and new trees using type + key to decide whether to reuse, update, reorder, or replace nodes. This makes DOM updates efficient.