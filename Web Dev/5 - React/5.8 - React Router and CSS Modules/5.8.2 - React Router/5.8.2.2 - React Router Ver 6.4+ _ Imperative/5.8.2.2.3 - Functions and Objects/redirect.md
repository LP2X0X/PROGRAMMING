---
tags: 
 - react
 - router
 - function
---

# 🔀 `redirect()` in React Router 6.4+

### **Used inside loaders and actions**

`redirect()` lets you navigate **before rendering**—during data loading or form submission.

It returns a special Response object that React Router understands and converts into a client-side redirect.

---

# 📌 1. Basic Example

```js
import { redirect } from "react-router-dom";

export async function loader() {
  const user = await getCurrentUser();

  if (!user) {
    return redirect("/login");
  }

  return null;
}
```

If `user` is null → the route **never renders** → the router immediately redirects.

---

# 📌 2. Redirect in `action()` (form handling)

```js
export async function action() {
  await savePost();
  return redirect("/posts");
}
```

This is the normal way to redirect after a form submission.

---

# 📌 3. Redirect with status code (optional)

Default status = **302**.

You can customize:

```js
return redirect("/login", { status: 301 }); // permanent
```

---

# 📌 4. Redirecting with query params

```js
return redirect(`/search?q=${keyword}`);
```

---

# 📌 5. Redirecting with dynamic params

```js
return redirect(`/users/${params.id}`);
```

---

# 📌 6. Redirect inside route definition (loader only)

```jsx
{
  path: "admin",
  loader: requireAdmin,
  element: <AdminPage />
}
```

```js
export function requireAdmin() {
  if (!isAdmin()) {
    return redirect("/not-authorized");
  }
}
```

---

# 📌 7. Redirect vs Navigate (important!)

### `redirect()`

- Used in **loaders** and **actions**
    
- Runs before rendering
    
- No component needed
    
- Works even on first load (server-like behavior)
    

### `<Navigate />`

- Used in **components**
    
- Requires rendering
    
- Cannot be used inside loaders/actions
    

---

# 📌 8. Redirect + Error Boundary interaction

`redirect()` **does not** trigger `errorElement`.  
It’s not an error — it's a controlled navigation.

Example:

```js
throw new Response("Not found", { status: 404 }); // goes to errorElement
redirect("/login");                               // navigates away
```

---

# 📌 9. Real-world pattern: Auth redirect

```js
export async function protectedLoader({ request }) {
  const isLogged = await checkAuth();

  if (!isLogged) {
    const url = new URL(request.url).pathname;
    return redirect(`/login?redirectTo=${url}`);
  }

  return null;
}
```

Used for routes that require login.

---

# 📌 10. Redirect after deleting a resource

```js
export async function action({ params }) {
  await deleteProduct(params.id);
  return redirect("/products");
}
```

---

# 📝 Quick Summary

- `redirect()` returns a special Response object that triggers navigation.
    
- Works only in **loaders** and **actions**.
    
- Used for auth checks, after form submissions, or conditional navigation.
    
- Does **not** involve rendering.
    
- Different from `<Navigate />`.
    
