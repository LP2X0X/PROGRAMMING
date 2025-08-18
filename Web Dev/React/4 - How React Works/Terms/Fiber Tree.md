---
tags: react, term, fundamental
---

- The **Fiber tree** is React’s internal data structure that represents your UI, one [[Fiber]] for each *React component* or *DOM element*.

```ad-important
Fiber tree is never destroy. Its a mutable data structure which mutates over and over again.
```

---

## **1. What it is**

- Each **Fiber node** corresponds to:
    
    - A component (function/class)
        
    - Or a host element (like a \<div>, \<span>, etc.)
        
    
- Together, they form a tree structure — the **Fiber tree** — mirroring your component hierarchy.
    
- React keeps **two Fiber trees** in memory during updates:
    
    1. **Current Fiber tree** → represents the UI currently on screen.
        
    2. **Work-in-progress Fiber tree** → represents the new UI after state/prop changes.
        

---

## **2. Linkage**

The **linkage** in a Fiber tree is how React connects all its **Fiber nodes** so it can efficiently walk the tree during rendering and reconciliation.

Instead of recursion, React uses **linked-list–like pointers** between Fiber nodes.

### **2.1. Core pointers in a Fiber node**

Each Fiber node has these main pointers:

|**Pointer**|**Purpose**|
|---|---|
|**child**|Points to the node’s **first child** in the tree.|
|**sibling**|Points to the node’s **next sibling** at the same level.|
|**return**|Points to the node’s **parent**.|
|**alternate**|Points to the matching node in the other Fiber tree (current ↔ work-in-progress).|

---

### **2.2. How they connect in the tree**

  
Imagine this JSX:

```html
<App>
  <Header />
  <Main>
    <Sidebar />
    <Content />
  </Main>
</App>
```

The Fiber tree pointers would look like:

```
(App)
  child ──► (Header)
               sibling ──► (Main)
                              child ──► (Sidebar)
                                            sibling ──► (Content)
  return pointers point back up ↑
```

- (App).child → first child (Header)
    
- (Header).sibling → next sibling (Main)
    
- (Main).child → first child (Sidebar)
    
- (Sidebar).sibling → next sibling (Content)
    
- return from each child points back to its parent.
    

### **2.3. Alternate pointer**

React keeps two versions of the Fiber tree:

- **Current Fiber tree** (what’s on screen)
    
- **Work-in-progress Fiber tree** (being built for next render)
    

Each node’s alternate pointer links to its **counterpart** in the other tree:

```
(Current App Fiber) <──alternate──> (WIP App Fiber)
(Current Header)    <──alternate──> (WIP Header)
...
```

This lets React reuse parts of the old tree instead of creating everything from scratch.

### **2.4. Why this linkage matters**

- React can **walk down** the tree via child.
    
- **Traverse siblings** via sibling.
    
- **Go back up** via return.
    
- Switch between current & work-in-progress versions via alternate.
    
- All without deep recursive calls — making it possible to pause and resume work.
    

---

## **3. Why two trees? (Double Buffering)**

- When something changes (like state), React builds a **work-in-progress Fiber tree** without touching the DOM yet.
    
- Once it’s done, React **swaps** the work-in-progress tree to become the current tree.
    
- This ensures updates are consistent and not partially applied.
    

---

## **4. Example**

  

Imagine a simple app:

```html
<App>
  <Header />
  <Main>
    <Sidebar />
    <Content />
  </Main>
</App>
```

Your **Fiber tree** might look like this:

```
App (Fiber node)
 ├── Header (Fiber node)
 └── Main (Fiber node)
      ├── Sidebar (Fiber node)
      └── Content (Fiber node)
```

Each node contains:

- type (e.g., function, class, host component)
    
- props & state
    
- Pointers: child, sibling, return (parent)
    
- Effect list (what needs updating in the DOM)
    
- Alternate pointer (link to the same node in the other tree)
    

---

## **4. Update flow**

1. **Trigger** → State changes in Sidebar.
    
2. **Render phase** → React creates a new work-in-progress Fiber tree, starting from Sidebar and updating upwards if needed.
    
3. **Commit phase** → React swaps the tree and updates the DOM based on the differences.
    