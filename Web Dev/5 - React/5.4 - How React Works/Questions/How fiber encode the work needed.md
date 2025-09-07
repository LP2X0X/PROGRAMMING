---
tags:
  - react
  - question
---

A **fiber node** is not just a snapshot of a component. It _encodes the work React still needs to do_ for that part of the tree. Here’s how:

---

## **🔹 How a Fiber Encodes Work**

  

Each fiber has several fields that represent **what work needs to be done** and **how React should process it**:

1. **Props & State**
    
    - pendingProps → the new props for this update
        
    - memoizedProps → the previous props
        
    - memoizedState → the previous state
        
        ➝ If pendingProps differ from memoizedProps, React knows it has work to do for this component.
        
    

---

2. **Type of Work**
    
    - tag → describes what kind of fiber this is (HostComponent, FunctionComponent, ClassComponent, etc.)
        
    - React uses this to decide how to process the fiber (call a function, run render(), create a DOM node, etc.).
        
    

---

3. **Links (tree structure)**
    
    - child, sibling, return → determine which fiber comes next in the “work loop.”
        
    - This makes the tree traversable _without recursion_, allowing React to pause/resume work.
        
    

---

4. **Effect Tracking**
    
    - flags (used to be effectTag) → encodes what side effects need to happen:
        
        - Placement (insert a new DOM node)
            
        - Update (update an existing DOM node)
            
        - Deletion (remove a node)
            
        - Ref, Callback, etc.
            
        
    - During the **complete phase**, React accumulates these effects so that the commit phase can apply them all at once.
        
    

---

5. **Alternate Fibers (double buffering)**
    
    - alternate → points to the old version of the fiber from the previous render.
        
    - This allows React to **compare old vs. new** (diffing) and decide if any work is required at all.
        
    

---

6. **Scheduling Info**
    
    - lanes (or older expirationTime) → priority of the work.
        
    - React can skip or delay low-priority fibers if something urgent (like user input) happens.
        
    

---

## **🔹 The Key Idea**

  

A fiber = **data + work description**

- _Data_ → “What this component was and is” (props, state, type).
    
- _Work_ → “What React should do next” (effect flags, pending vs. memoized values, priority).
    

  

That’s why React can pick up a fiber, run “beginWork” on it, decide what its children should be, pause if needed, and later continue exactly where it left off.

---

✨ So in short:

The fiber encodes the work needed via its **effect flags, scheduling priority, and the difference between its new (pending) and old (memoized) props/state**.