---
tags: 
 - react
 - router
 - term
---

# 🎯 **What is Revalidation?**

> **Revalidation = re-running a route’s loader to refresh its data.**

React Router’s data system is _cache-based_:

- Loader data is cached
    
- It only reloads when necessary
    
- Mutations (actions) or navigation can trigger a refresh
    

Revalidation is how React Router keeps UI **in sync with server state** without manually refetching.

---

# 📌 **1. When Does Revalidation Happen Automatically?**

## ✔ A) After an `action()` finishes

When any action on a route completes, React Router:

1. Runs the relevant `action`
    
2. Then **re-runs loaders** for:
    
    - the current route, and
        
    - all parent routes
        

This ensures the UI reflects updated data.

Example:

- You add a new todo
    
- The action adds it on the server
    
- React Router automatically re-runs the loader for the todo list
    

---

## ✔ B) When navigating to a new route

Normal navigation (link click) re-runs loaders for the new route tree.

---

## ✔ C) When calling `fetcher.load()`

Manually reloading a specific loader:

```js
fetcher.load("/posts");
```

---

## ✔ D) When calling `revalidate()`

You can force a refresh programmatically.

---

# 📌 **2. When Does React Router _Not_ Revalidate Automatically?**

- When submitting a fetcher action on a _different_ route
    
- When you manually use `fetch()` instead of the router
    
- When you want to control revalidation yourself
    

---

# 🚀 **3. Manual Revalidation**

React Router exposes two revalidation triggers:

---

## ⭐ A) `useRevalidator()`

```js
const { revalidate, state } = useRevalidator();
```

Then:

```js
revalidate();
```

This forces **all active loaders** to run again.

State is:

```
idle → loading → idle
```

Useful for:

- “Refresh Data” buttons
    
- Polling
    
- Websocket-triggered reloads
    

---

## ⭐ B) `fetcher.load()` (revalidate a specific route)

```js
fetcher.load("/dashboard");
```

Only reloads the loader for that URL, not the whole tree.

Ideal for:

- Live search
    
- Background updates
    
- Partial refreshes
    

---

# 📦 **4. Fine-Grained Control: ShouldRevalidate**

You can customize when a loader re-runs using:

```js
export function loader({ request, params, context, shouldRevalidate }) {
  ...
}
```

Or define it on the route:

```js
{
  path: "/posts",
  loader: postsLoader,
  shouldRevalidate: ({ currentParams, nextParams, formMethod, formData }) => {
    return true; // or conditional logic
  }
}
```

This gives full control, e.g.:

- skip reloading on same query
    
- reload only when ID changes
    
- prevent revalidation while submitting
    

---

# 🔄 **5. Revalidation Flow (Visual)**

```
User submits form
       ↓
Action runs
       ↓
If action succeeds:
    React Router re-runs loaders
       ↓
UI updates with fresh data
```

---

# 🧠 **6. Why Revalidation Matters**

Revalidation gives React Router a **query library built-in**.

You get benefits similar to React Query:

- Cache
    
- Auto-refetch after mutations
    
- Background refresh
    
- Consistent data flow
    
- Automatic loading states
    

But with route-aware context.

---

# ⭐ Summary (Most Important Points)

- **Revalidation = re-running loaders**
    
- Automatically happens:
    
    - After actions
        
    - After navigation
        
- Manual options:
    
    - `useRevalidator()`
        
    - `fetcher.load()`
        
- Advanced control via `shouldRevalidate`
    