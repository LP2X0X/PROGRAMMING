---
tags: 
 - react
 - router
 - note
---

## 🧭 Declarative React Router (Traditional `<Routes>` API)

When you use React Router **declaratively** like this:

```jsx
<Routes>
  <Route path="/posts" element={<Posts />} />
</Routes>
```

### What happens

1. Route matches
    
2. Component renders
    
3. `useEffect` runs
    
4. Data is fetched
    
5. Component re-renders with data
    

This is **fetch-after-render**.

---

## ❌ Does This Support “Render as You Fetch”?

**No.**

The traditional declarative router:

- ❌ Does **not** block navigation
    
- ❌ Does **not** preload data
    
- ❌ Does **not** integrate with Suspense for data
    
- ❌ Causes loading states _inside_ components
    

So you must write patterns like:

```jsx
useEffect(() => {
  fetchData().then(setData);
}, []);
```

This is **imperative data fetching**.

---

## 🧠 What “Render as You Fetch” Actually Means

Render-as-you-fetch means:

- Start fetching **before** rendering
    
- Suspend rendering until data is ready
    
- Avoid waterfalls
    
- Share cache between navigations
    

This requires **data awareness at the routing level**.

---

## 🚀 Modern React Router (Data Router API)

React Router **does support render-as-you-fetch**, but **only via the Data Router API**:

- `createBrowserRouter`
    
- `loader`
    
- `<RouterProvider />`
    

Example:

```ts
const router = createBrowserRouter([
  {
    path: "/posts",
    loader: () => fetch("/api/posts"),
    element: <Posts />,
  },
]);
```

### Why this works

- Loader runs **before render**
    
- Navigation waits for data
    
- Router integrates with Suspense
    
- Errors handled declaratively
    

This is **React Router’s built-in solution**.

---

## 📦 Do You Still Need Another Library?

### Case 1: Using Declarative `<Routes>` only

✅ **Yes**, you need another library.

Common choices:

| Library                      | Purpose                                |
| ---------------------------- | -------------------------------------- |
| React Query (TanStack Query) | Server-state, caching, async lifecycle |
| Redux Toolkit Query          | Centralized API + cache                |
| SWR                          | Simple cache-based fetching            |

These provide:

- Cache
    
- Deduplication
    
- Background refetch
    
- Loading/error states
    

---

### Case 2: Using React Router Data Routers

❌ **Not strictly required**.

React Router loaders already give:

- Preloading
    
- Error boundaries
    
- Navigation blocking
    
- Redirects
    

But:

⚠️ They **do not replace** full server-state libraries:

- No background refetch
    
- No mutation invalidation
    
- Limited caching controls
    

---

## 🆚 Router Loader vs React Query

| Feature             | Router Loader | React Query |
| ------------------- | ------------- | ----------- |
| Fetch before render | ✅            | ✅          |
| Navigation blocking | ✅            | ❌          |
| Caching             | ⚠️ Basic      | ✅ Advanced |
| Background refetch  | ❌            | ✅          |
| Mutations           | ⚠️ Limited    | ✅          |
| Cross-route sharing | ⚠️            | ✅          |

---

## 🧠 Recommended Modern Pattern

### Best practice (2024+)

- **Router loaders** → page entry data
    
- **React Query / RTK Query** → ongoing server state
    

They **complement**, not replace, each other.

---

## 🧠 Final Verdict

- Declarative `<Routes>` **cannot** do render-as-you-fetch
    
- Data Routers **can**
    
- Without Data Routers, you **need** React Query / RTK Query
    
- Even with Data Routers, React Query is often still valuable
    