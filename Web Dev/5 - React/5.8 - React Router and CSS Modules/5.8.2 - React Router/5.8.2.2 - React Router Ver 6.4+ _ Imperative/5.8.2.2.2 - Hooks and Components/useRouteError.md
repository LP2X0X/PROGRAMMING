---
tags: 
 - react
 - router
 - hook
 - error
---

# 🧭 **`useRouteError()` — Detailed Overview (React Router)**

`useRouteError()` is a hook provided by **React Router** that lets you **access the error thrown by a route loader, action, or component** inside that route’s `errorElement`.

It is part of the **Data APIs** (loader/action) and works ONLY in a route that defines an `errorElement`.

---

## ✅ **When does an error occur?**

React Router catches errors thrown from:

### 1. **Loader**

```ts
export async function loader() {
  const res = await fetch("/api/user");
  if (!res.ok) throw new Error("Failed to load user");
  return res.json();
}
```

### 2. **Action**

```ts
export async function action() {
  throw new Response("Forbidden", { status: 403 });
}
```

### 3. **Component render**

```jsx
function Profile() {
  throw new Error("Unexpected error");
}
```

---

## 📌 **How `useRouteError()` fits in the route config**

### 📍 Route definition

```jsx
const router = createBrowserRouter([
  {
    path: "/profile",
    loader: profileLoader,
    element: <Profile />,
    errorElement: <ErrorBoundary />,
  },
]);
```

### 📍 Error component

```jsx
import { useRouteError, isRouteErrorResponse } from "react-router-dom";

function ErrorBoundary() {
  const error = useRouteError();

  if (isRouteErrorResponse(error)) {
    return (
      <div>
        <h1>{error.status} - {error.statusText}</h1>
        <p>{error.data}</p>
      </div>
    );
  }

  return <div>Something went wrong: {error.message}</div>;
}
```

---

# 🧩 **What type of errors can you get?**

React Router can produce two types:

## 1. **JavaScript Error**

Thrown using `throw new Error()`

Properties:

- `message`
    
- `stack`
    
- custom fields you attach
    

👍 Good for unexpected runtime issues.

---

## 2. **Route Error Response**

Thrown using `throw new Response()`

This is a special subtype used for _HTTP-style errors_:

Properties:

- `status` (ex: 404, 403)
    
- `statusText`
    
- `data` (HTML, text, JSON)
    
- response body
    

Use helper:

```ts
isRouteErrorResponse(error)
```

---

# 🎯 **Why `useRouteError()` matters**

### ✔ Centralized error UI per route

Each route can have its own UI for errors coming from:

- fetch failures
    
- validation failure
    
- not-found
    
- unauthorized, etc.
    

### ✔ More control than try/catch

React Router intercepts errors for you → No need to wrap every loader/action in try/catch.

### ✔ Works like per-page "error boundary"

But with extra context for loaders/actions.

---

# 📘 **Typical minimal example**

### Route

```jsx
{
  path: "/dashboard",
  loader: async () => {
    throw new Response("Not authorized", { status: 401 })
  },
  errorElement: <DashboardError />,
}
```

### Error element

```jsx
function DashboardError() {
  const error = useRouteError();

  return (
    <div>
      <h1>Error loading dashboard</h1>
      <pre>{JSON.stringify(error, null, 2)}</pre>
    </div>
  );
}
```

---

# ⭐ **Common use cases**

- Display 404 page with route-specific info
    
- Show validation errors in forms (from `action()`)
    
- Handle API failures gracefully
    
- Replace global error boundaries with route-based ones
    
- Automatically catch thrown `Response()` as HTTP errors
    