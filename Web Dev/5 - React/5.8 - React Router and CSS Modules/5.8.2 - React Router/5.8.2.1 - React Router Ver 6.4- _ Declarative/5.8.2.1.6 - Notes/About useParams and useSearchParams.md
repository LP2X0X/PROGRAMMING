---
tags: 
 - react
 - router
 - note
 - useParams
 - useSearchParams
---

**When the URL changes**, **any component that calls `useParams()`** inside a `<Route>` that matches the new URL **will re-render**, because React Router updates its context.

However, **it does not trigger every `useParams` in the entire app**, only those that are **currently mounted and inside active routes**.

---

## 🧭 Detailed Explanation

When the URL changes:

1. React Router updates its internal `RouterContext`.
    
2. All components that call hooks relying on that context (`useLocation`, `useNavigate`, `useParams`, etc.) will **receive new context values**.
    
3. If the new params differ from the previous ones, **React re-renders** the component using `useParams()`.
    

---

## 🧩 Example

```jsx
function UserPage() {
  const { id } = useParams();
  console.log("Rendered with id:", id);
  return <div>User ID: {id}</div>;
}

// Routes
<Route path="/users/:id" element={<UserPage />} />
```

When the URL changes from  
`/users/1` → `/users/2`,

React Router updates the `params` from `{ id: "1" }` → `{ id: "2" }`.  
👉 `UserPage` re-renders because the context value changed.

---

## 💡 Important Notes

- If you have multiple routes each using their own `useParams()`, only **the components of the matching routes** will re-render.
    
- Unmounted routes (not currently visible) do **not** re-render.
    
- If the URL changes but the params **stay the same**, React Router may **skip** re-rendering because the value is shallowly equal.
    

---

## ✅ Pro Tip

If you want to respond to param changes _without re-rendering everything_, you can memoize or isolate logic like this:

```jsx
const { id } = useParams();

useEffect(() => {
  fetchUser(id);
}, [id]); // runs only when id changes
```

This way, only your data fetching or related effects respond — the rest of the component can stay stable.