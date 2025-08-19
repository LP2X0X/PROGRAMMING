---
tags:
  - react
  - term
  - fundamental
---


## **What is the React Fiber Reconciler?**


The **React Fiber Reconciler** is the internal engine in React (introduced in React 16) that figures out:

1. **What parts of the UI need to change** after a component’s state or props update.
    
2. **How to apply those changes efficiently** to the DOM.
    

  

Think of it like a project manager for the UI:

- It reads the updated blueprint (Virtual DOM tree).
    
- Compares it to the current blueprint.
    
- Decides the minimal set of changes to actually apply.
    
- Schedules these changes in a way that keeps the UI responsive.
    

---

## **Why Fiber Was Created**

Before Fiber, React’s rendering was **synchronous** — if a component tree was huge, updates could block the main thread, causing jank and freezing the UI.

Work in Fiber can be perform asynchronously, so:

> **Rendering process can be split into chunks, tasks can be prioritized, and work can be paused, reused, or thrown away**

Fiber introduced:

- **Incremental rendering** (breaking work into small chunks).
    
- **Prioritization** (important updates first, e.g., typing in an input box).
    
- **Pausing and resuming work** (don’t block animations or user interactions).
    
- **Enable concurrent feature**.

---

## **How Fiber Works Internally**

  - On inital render, it takes the entire React element tree and create a [[Fiber Tree]]

Fiber is both:

- **A data structure**: Each Fiber node represents a component instance (with its type, props, state, and effects).
    
- **An algorithm**: Determines what work to do and in what order.
    

  

The **Fiber node** contains:

- type → Function/class of the component.
    
- key → For list reconciliation.
    
- stateNode → The actual DOM node or class instance.
    
- child, sibling, return → Tree navigation links.
    
- pendingProps, memoizedProps, memoizedState → For tracking changes.
    
- flags → Side effects to perform (e.g., placement, update, deletion).
    

---

## **Two Main Phases**

1. **Render (Reconciliation) Phase**
    
    - Creates a new Fiber tree based on changes.
        
    - This phase can be **paused**, **aborted**, or **restarted**.
        
    - Produces a list of effects to commit.
        
    
2. **Commit Phase**
    
    - Runs side effects and updates the DOM.
        
    - This phase is **synchronous** — once started, it finishes without interruption.
        
    

---

## **Scheduling & Priorities**

  

React assigns priorities to updates:

- **Immediate** → User typing or clicking.
    
- **High** → Animations.
    
- **Low** → Background data loading.
    
- **Idle** → Non-urgent stuff.
    

  

This is possible because Fiber breaks rendering into **units of work** and can yield control back to the browser between them.

---

## **Analogy**

  

Imagine React rendering as painting a huge mural:

- **Old React**: One artist paints the whole thing in one go — if you interrupt them, you just get a half-painted wall.
    
- **Fiber React**: The artist paints section by section, checking after each section if there’s something more urgent to do (like quickly fixing a spot where the paint is dripping).
    

---

## **Visual**

```
[ State/Props Change ]
        |
        v
[ Render Phase - Build new Fiber tree ]
        |
        v
[ Diff with old Fiber tree ]
        |
        v
[ Commit Phase - Apply minimal changes to DOM ]
```
