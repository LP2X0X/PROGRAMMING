---
tags: 
 - react
 - router
 - hook
---

- `useParams` is a **hook** supplied by React Router.
    
- It lets you access the **dynamic parameters** (path variables) from the current URL inside a component.
    
- When you define a route with parameters (e.g. `/users/:userId/posts/:postId`) — the parts prefixed by `:` become parameters.
    

Example:

```jsx
// in your router definition
<Route path="/users/:userId/posts/:postId" element={<Post />} />

// in Post component
import { useParams } from "react-router-dom";

function Post() {
  const params = useParams();
  // params is an object: { userId: "...", postId: "..." }
  return (
    <div>
      <h1>Post {params.postId} by user {params.userId}</h1>
    </div>
  );
}
```

So `useParams` returns an object where keys = parameter names, values = the current string value from URL.

---

# 🔧 When / Why to Use It

- When you have **dynamic routes** (IDs, slugs, categories, etc.) and you need to know which specific resource to load.
    
- It helps components react (fetch data, render proper UI) based on URL parameters.
    
- It keeps UI state in sync with URL — important for bookmarkable links, back/forward navigation.
    

---

# 🧠 What to Remember / Limitations

- The values returned are always **strings** (even if route defines `:id`). If you need a number, you must convert explicitly:
    
    ```js
    const { id } = useParams();
    const idNum = Number(id);
    ```
    
- If a parameter is optional (depending on how you configured your routes), it might be `undefined`. Be careful.
    
- `useParams` only reads parameters from the **current matched route**. If a higher-level route defines `:lang` and a nested route defines `:id`, both show up in `useParams`. But if that nested route is not matched, its params don’t exist.
    
- Changes to the URL (via navigation, link click, programmatic navigation) cause component rerender, and `useParams` updates accordingly.
    

---

# 🧑‍💻 Common Patterns / Best Practices

- **Fetch data**: In component load, call API with the `id` from params.
    
- **Type conversion/validation**: For numeric IDs or dates — parse & validate before use.
    
- **Optional params**: Check for undefined / fallback values.
    
- **Nested routes**: Use params from parent & child routes properly.
    
- **Destructure**:
    
    ```js
    const { userId, postId } = useParams();
    ```
    
- **React to param changes**: If your component stays mounted when URL changes (e.g. same route but different param), you can use `useEffect` to listen param change and re-fetch.
    

---

## 🔄 What Happens Under the Hood — and Key Behaviors

- `useParams` reads the **“matched”** route for the current location and returns **only the dynamic segments** defined in the path. :contentReference[oaicite:8]{index=8}  
- It works with nested routes: child components rendered under a parent route will also get params from both parent and child paths (if defined). :contentReference[oaicite:9]{index=9}  
- Since it's a **hook**, it must be called inside a **functional component** (or another hook), not in class components. :contentReference[oaicite:10]{index=10}  
- The returned object’s values are strings (the URL segments). If you expect numeric IDs, you may need to convert (e.g. using `parseInt`) manually.

---

## ⚠️ Limitations and Gotchas

- If the route pattern doesn’t include a param (or the URL doesn’t match it), the corresponding param key will be `undefined`. :contentReference[oaicite:11]{index=11}  
- Because the params come from the URL string, you typically need to **validate or sanitize** them before use — for example, ensure an ID is numeric, a slug has correct format, etc. :contentReference[oaicite:12]{index=12}  
- It only provides **path parameters**, not query-string parameters. For query parameters (e.g. `?page=2`), you’d use a different hook (e.g. `useSearchParams`). :contentReference[oaicite:13]{index=13}


---

# ✅ Summary

- `useParams` is the standard way in React Router to access URL path parameters.
    
- It returns an object of strings keyed by parameter names.
    
- Very useful for dynamic routing (IDs, slugs, filters).
    
- Always handle conversion/validation when needed.
    