---
tags: 
 - react
 - tools
---

# 🧭 React Developer Tools – Complete Overview

React DevTools has **two major panels** inside your browser’s DevTools:

1. **🧩 Components tab** → inspect the structure, props, and hooks of your React tree.
    
2. **⚡ Profiler tab** → measure render performance and detect wasted renders.
    

Let’s go through both in deep detail 👇

---

## 🧩 1. Components Tab – Inspect and Understand Your App

### 🧠 What It Shows

The Components tab visualizes the **React component tree** of your running application.  
It’s React’s internal virtual DOM — **not the raw DOM**, but React’s structure of components.

---

### 📂 The Left Panel — The Component Tree

- You’ll see a **tree view** showing your entire React app hierarchy:
    
    ```
    <App>
      <Header />
      <Sidebar />
      <Main>
        <UserList />
        <UserDetails />
      </Main>
    ```
    
- You can **expand/collapse** nodes to navigate parent-child relationships.
    

---

### 🔍 The Right Panel — Component Inspector

When you select a component on the left:

- You’ll see **all the details React knows** about it:
    

#### 🔸 Props

- Lists the current props and their values.
    
- Updates live as props change.
    
- If a prop is a function, it shows `[ƒ]` — hover to see the function definition.
    

#### 🔸 State

- Shows internal `useState` values.
    
- Each state update re-renders the component; you can watch it live.
    

#### 🔸 Hooks

- Shows all hooks used inside the component (useState, useEffect, useMemo, useContext, etc.).
    
- You can inspect hook values, dependencies, and memoized values.
    
- Even nested custom hooks are expanded (e.g., `useAuth` → reveals its inner states).
    

#### 🔸 Context

- Displays context values consumed by this component (if using `useContext`).
    

---

### 🛠 Tools and Actions

#### 1. **Highlight Updates**

Toggle the “Highlight updates when components render” option —  
→ React will **visually flash** (with a colored outline) each component that re-renders.  
Perfect for detecting wasted renders.

#### 2. **Search Bar**

You can search for component names or props (e.g., typing “user” highlights `<UserCard>`).

#### 3. **Right-Click Options**

- **Show in Elements panel** → jumps to the corresponding DOM node in Chrome DevTools.
    
- **Copy component path** → gives you the full hierarchy.
    
- **View source** → opens your component in VS Code (if source maps are available).
    

#### 4. **Render Counter (React 18+)**

Hover over a component to see how many times it re-rendered since mounting.

---

### 🧭 Common Use Cases

|Goal|What to Do|
|---|---|
|Debug why props/state not updating|Inspect component’s state and props live|
|Watch how context values flow|Inspect the Context section|
|Find re-renders visually|Turn on highlight updates|
|Explore large app structure|Expand and collapse nested trees|

---

## ⚡ 2. Profiler Tab – Measure and Optimize Performance

The **Profiler** helps you **measure render times**, **detect wasted renders**, and **see what triggered re-renders**.

```ad-tip
You should turn on the **Record why each component rendered while profilling** option in setting.
```

---

### 🧠 What It Does

The Profiler **records every render and commit** of your React app.  
It shows:

- How long each component took to render.
    
- Which components rendered unnecessarily.
    
- What caused the render (state update, prop change, context change, etc.).
    

---

### 🎬 How to Use It

1. Go to the **Profiler** tab.
    
2. Click **“Start profiling”** (red dot icon).
    
3. Interact with your app (click buttons, navigate, etc.).
    
4. Click **“Stop profiling”**.
    
5. You’ll now see a detailed visualization of render performance.
    

---

### 📊 The Profiler UI Breakdown

#### 1. **Flamegraph View (default)**

- Each bar represents a React component.
    
- The **width** = render duration (wider = slower).
    
- The **color** = relative cost (red = expensive, yellow = medium, green = light, gray = none).
    

Hover over a component to see:

- Actual render time (ms)
    
- Total render time (including children)
    
- Render count
    

You can click a bar to inspect more details in the sidebar.

---

#### 2. **Ranked View**

- Sorts components by render cost (slowest at the top).
    
- Lets you find **performance bottlenecks** easily.
    

---

#### 3. **Sidebar Info Panel**

When you click a component:

- **Why did this render?**
    
    - Shows whether it re-rendered due to:
        
        - Prop change
            
        - State change
            
        - Context change
            
        - Parent re-render
            
- **Render duration**
    
    - Time spent rendering just this component.
        
- **Commit duration**
    
    - Time React took to apply all changes to the DOM in that commit.
        

---

### ⚙️ Advanced Options

#### ⚡ “Record why each component rendered”

If enabled, React tracks _exact reasons_ for each render — great for diagnosing wasted renders.

#### 🧩 “Hide commits below X ms”

Filters out small renders to focus on heavy ones.

#### 🔄 Commit Navigation

You can step through each render commit like a timeline — useful for debugging changes between renders.

---

### 🧮 Typical Performance Diagnosis Flow

1. **Start Profiling**
    
2. Interact with app
    
3. Stop and open the flamegraph
    
4. Look for wide or red bars → those took the longest
    
5. Click to see **“Why did this render?”**
    
6. Apply memoization (`React.memo`, `useMemo`, `useCallback`)
    
7. Re-profile → confirm improvement
    

---

### 🧠 Example: Diagnosing a Wasted Render

If you see a component re-rendering even when its props/state are the same:

- Check “Why did this render?” → shows “Parent re-rendered”.
    
- You can fix it with:
    
    ```jsx
    export default React.memo(MyComponent);
    ```
    
- Or stabilize prop references with `useMemo` / `useCallback`.
    

---

## 🔍 Components vs Profiler Summary

|Feature|Components Tab|Profiler Tab|
|---|---|---|
|Goal|Inspect state, props, hooks|Measure render performance|
|Focus|Static view of current tree|Dynamic view of rendering timeline|
|Shows|Data structure|Time and cause of updates|
|Use When|Debugging logic / data flow|Debugging slowness or wasted renders|
|Highlight tool|“Highlight updates” toggle|Flamegraph colors|
|Best for|Understanding behavior|Optimizing performance|

---

## 🚀 Final Tips

- **Use Components first** to confirm your data flow and prop logic.
    
- Then **use Profiler** to confirm your optimization effectiveness.
    
- Remember: A re-render isn’t always bad — only wasted re-renders hurt performance.
    
- For frequent renders (like animations or large lists), use memoization and virtualization.
    