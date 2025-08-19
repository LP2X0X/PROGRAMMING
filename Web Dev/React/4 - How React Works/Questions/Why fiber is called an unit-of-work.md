---
tags:
  - react
  - question
---

In **React Fiber**, the term _“fiber”_ doesn’t just mean the tree itself, but each **fiber node** represents a **unit of work**. Let me break down why:

---

### **1. What is “work” in React?**

  

When React updates the UI (after a setState or prop change), it needs to figure out:

- Which components should re-render
    
- What elements they return
    
- What changes need to happen in the DOM
    

  

That whole process is the **work** React must do to update the UI.

---

### **2. What is a Fiber Node?**

  

A **fiber node** is a JavaScript object that represents:

- A component instance (function/class/host element)
    
- Its props & state
    
- References to its parent, child, and siblings
    
- Effect tags (what needs to be updated)
    
- Expiration times (priority of work)
    

  

So, a fiber node is not just a description of the UI—it also encodes _the work needed_ for that part of the tree.

---

### **3. Why is it considered a unit of work ?**

  

Because React schedules and processes updates **one fiber at a time**:

- Each fiber can be _begun_ → compute what the component should render (reconciliation).
    
- Each fiber can be _completed_ → collect effects (what to change in the DOM).
    
- If React needs to pause (because the browser needs to paint or user input is happening), it can stop **after finishing the current fiber** and later resume at the next fiber.
    

  

So instead of doing the entire tree update in one big blocking chunk (old React), React Fiber breaks it into small **work units** = fibers.

---

### **4. Key Insight**

  

👉 **A fiber is both:**

- The data structure describing a component in the tree
    
- The _execution unit_ React uses to schedule and perform rendering work
    

  

That’s why we say **“a fiber is a unit of work”** — React processes them one by one, and can pause/resume/abort at the fiber level.
