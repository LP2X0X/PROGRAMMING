---
tags: 
 - react
 - router
 - redux
 - toolkit
 - note
---

Both **`createAsyncThunk` (Redux Toolkit)** and **React Router `action` functions**:

- Handle **side effects** (API calls, mutations)
    
- Are **async-first**
    
- Represent **intent** rather than UI logic
    
- Decouple **data mutation** from components
    
- Return structured results that update state elsewhere
    

At a high level, both answer:

> “What should happen when the user submits or triggers this intent?”

---

## Key Differences (Very Important)

### 1. Ownership of State

**createAsyncThunk**

- Owns **global application state** (Redux store)
    
- Result is dispatched as actions (`pending / fulfilled / rejected`)
    
- State lives independently of routes
    

**React Router action**

- Owns **route-scoped data**
    
- Result affects:
    
    - navigation
        
    - loaders revalidation
        
    - redirects
        
- State is tied to the routing tree
    

---

### 2. Output Semantics

**createAsyncThunk**

```ts
return data; // becomes action.payload
```

- Updates Redux state via reducers
    
- No navigation control
    

**React Router action**

```ts
return redirect("/orders");
return json({ error: "Invalid" }, { status: 400 });
```

- Controls **navigation flow**
    
- Controls **revalidation**
    
- Can short-circuit rendering
    

---

### 3. Execution Model

**createAsyncThunk**

- Triggered manually via `dispatch`
    
- UI-driven
    

**React Router action**

- Triggered by **form submission or navigation**
    
- URL-driven
    
- Part of routing lifecycle
    

---

### 4. Error Handling

**createAsyncThunk**

- Errors flow into Redux state
    
- UI decides how to render errors
    

**React Router action**

- Errors bubble to **error boundaries**
    
- Integrated with route hierarchy
    

---

## Mental Model Comparison

|Concept|createAsyncThunk|React Router action|
|---|---|---|
|Trigger|dispatch()|navigation / form|
|Scope|Global store|Route|
|State ownership|Redux|Router|
|Navigation control|❌|✅|
|Revalidation|❌|✅|
|Error boundaries|❌|✅|

---

## When They Feel the Same

They feel very similar when:

- Submitting forms
    
- Mutating server data
    
- Coordinating async work outside components
    

But they live in **different layers**:

- Redux → **state management**
    
- React Router → **navigation + data lifecycle**
    

---

## Bottom Line

> **createAsyncThunk is an async state mutation tool.**  
> **React Router actions are async navigation-side effects.**

They overlap in _what they do_, but not in _what they control_.