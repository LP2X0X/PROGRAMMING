[](useRouteError.md)[](useRouteError.md)---
tags: 
 - react
 - router
 - error
 - handling
---

React Router 6.4+ introduces **built-in error handling** for:

- `loader()` errors
    
- `action()` errors
    
- render errors
    
- thrown responses (`throw new Response()`)
    
- `redirect()` handling
    

Everything is handled via an **`errorElement`**.

```ad-note
Error happens in child route will bubble up to parent route unless it is handled in the route itself.
```

---

# 🎯 1. `errorElement` — the core

Each route can define an `errorElement`, which renders when:

- Loader throws
    
- Action throws
    
- There is an unhandled JavaScript error
    
- A “thrown response” occurs (404, 500, etc.)
    

```jsx
{
  path: "/products/:id",
  element: <ProductPage />,
  loader: productLoader,
  errorElement: <ErrorPage />
}
```

---

# 🎯 2. Throwing errors inside `loader` / `action`

### Throw normal JS errors

```js
export async function loader() {
  const res = await fetch("/api/data");

  if (!res.ok) {
    throw new Error("Failed to load");
  }

  return res.json();
}
```

### Throwing a Response (recommended)

Useful for 404, 500, unauthorized, etc.

```js
throw new Response("Not found", { status: 404 });
```

---

# 🎯 3. Accessing the error in `ErrorPage`

Use:

```js
import { useRouteError, isRouteErrorResponse } from "react-router-dom";
```

### Pattern:

```jsx
export default function ErrorPage() {
  const error = useRouteError();

  if (isRouteErrorResponse(error)) {
    return (
      <div>
        <h1>{error.status}</h1>
        <p>{error.data}</p>
      </div>
    );
  }

  return (
    <div>
      <h1>Something went wrong</h1>
      <p>{error.message}</p>
    </div>
  );
}
```

---

# 🎯 4. Route hierarchy error bubbling

If a child route fails and has **no** `errorElement`, React Router automatically **bubbles** the error up to the nearest parent that has one.

Example:

```
Root Route (has errorElement)
 └── Dashboard (no errorElement)
      └── Products (no errorElement)
```

If `Products.loader` throws → `Root.errorElement` renders.

---

# 🎯 5. Handling errors during form submissions (action)

```js
export async function action({ request }) {
  const formData = await request.formData();
  const email = formData.get("email");

  if (!email.includes("@")) {
    throw new Error("Invalid email");
  }

  // continue...
}
```

`ErrorPage` catches it.

---

# 🎯 6. Use `useNavigation()` with errors (for UI)

```jsx
const navigation = useNavigation();

if (navigation.state === "submitting")
  return <Spinner text="Submitting..." />;
if (navigation.state === "loading")
  return <Spinner text="Loading..." />;
```

If load fails → errorElement renders, not your component.

---

# 🎯 7. Error boundaries per route = best practice

Define an `errorElement` for _every route that has a loader/action_.

This gives:

- Better UX
    
- More granular error messages
    
- Parent layout stays intact
    

Example:

```jsx
{
  path: "users",
  element: <UsersLayout />,
  errorElement: <UsersError />,      // boundary for the whole group
  children: [
    {
      index: true,
      element: <UsersList />,
      loader: usersLoader
    },
    {
      path: ":id",
      element: <UserDetails />,
      loader: userLoader,
      errorElement: <UserDetailsError />   // more specific
    }
  ]
}
```

---

# 🎯 8. Full minimal example

```jsx
// router.js
const router = createBrowserRouter([
  {
    path: "/",
    element: <Layout />,
    errorElement: <RootError />,
    children: [
      {
        path: "posts/:id",
        element: <Post />,
        loader: postLoader,
        errorElement: <PostError />
      }
    ]
  }
]);
```

```jsx
// postLoader
export async function postLoader({ params }) {
  const res = await fetch(`/api/posts/${params.id}`);

  if (res.status === 404) {
    throw new Response("Post not found", { status: 404 });
  }

  return res.json();
}
```

```jsx
// PostError.jsx
export default function PostError() {
  const err = useRouteError();

  if (isRouteErrorResponse(err))
    return <h1>{err.status} – {err.data}</h1>;

  return <h1>Something went wrong: {err.message}</h1>;
}
```

---

# 📝 Quick Summary

- **`errorElement`** is your error boundary per route.
    
- Use **`throw new Response()`** for 404/500 style errors.
    
- Loader/action errors automatically trigger the error boundary.
    
- Errors bubble up the route tree until a boundary is found.
    
- `useRouteError()` + `isRouteErrorResponse()` lets you read the error.
    
- This fully replaces `ErrorBoundary` components from older React patterns.
    