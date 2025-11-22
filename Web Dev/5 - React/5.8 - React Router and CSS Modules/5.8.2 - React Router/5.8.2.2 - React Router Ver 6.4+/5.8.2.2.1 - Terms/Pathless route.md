---
tags: 
 - react
 - router
 - term
---

If you **don’t specify a `path`** in a React Router route, the route becomes a **“pathless route”**.  
This has very specific behavior and is often misunderstood.

---

# ✅ 1. **The route ALWAYS matches**

A route without a `path` matches **whenever its parent route matches**.

Example:

```jsx
{
  element: <Layout />,
  children: [
    {
      element: <Dashboard />  // ← no path
    }
  ]
}
```

`Dashboard` will render **every time the parent route renders**, because nothing is restricting it.

It’s basically a route that is _always active_ under its parent.

---

# ✅ 2. **Common uses: layout routes**

Pathless routes are usually used to group UI structure, such as:

- Shared layout
    
- Shared loaders
    
- Shared error handling
    
- Shared authentication checks
    

Example:

```jsx
{
  element: <ProtectedRoute />,   // guard logic
  children: [
    {
      path: "dashboard",
      element: <Dashboard />
    }
  ]
}
```

Here, `<ProtectedRoute>` is a wrapper.  
It isn’t itself a navigable path — it just wraps children.

---

# ✅ 3. **It is _NOT_ a page you can navigate to**

You cannot go to a pathless route with a URL.

This will NOT work:

```
/(no path)
```

It has no path, so React Router can’t navigate to it directly.

It only affects rendering of its children.

---

# ✅ 4. **It inherits params and context**

Since it always matches, it can:

- Set up loaders/actions that children share
    
- Provide context to children
    
- Provide `errorElement` for children
    
- Render UI wrappers
    

---

# ⚠️ 5. If multiple pathless routes are nested → they all render

Example:

```jsx
{
  element: <A />,
  children: [
    {
      element: <B />,
      children: [
        {
          path: "inbox",
          element: <Inbox />,
        }
      ]
    }
  ]
}
```

When visiting `/inbox`:

**A → B → Inbox**  
All three render because none blocked by a path.

---

# Summary

### **When a route has no `path`:**

✔ It always matches  
✔ It is not navigable by URL  
✔ It acts like a wrapper around its children  
✔ It’s used for layouts, guards, loaders, and error boundaries  
✔ All nested pathless routes render together