---
tags: 
 - react
 - router
 - function
 - essential
---

`createBrowserRouter()` creates a **router instance** that uses the browser’s native [[History Stack|History API]] (`pushState`, `popstate`) to update the URL **without reloading the page**.

It is the standard router for **normal web apps** where you control the server.

```jsx
function createBrowserRouter(
  routes: RouteObject[],
  opts?: DOMRouterOpts,
): DataRouter
```

---

# 🔹 How to Use `createBrowserRouter` (the important section)

### **1. Import it**

```js
import { createBrowserRouter, RouterProvider } from "react-router-dom";
```

### **2. Define your route tree**

This is your entire app’s navigation structure:

```js
const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    children: [
      {
        path: "",
        element: <Home />,
      },
      {
        path: "products",
        element: <Products />,
      },
      {
        path: "products/:id",
        element: <ProductDetail />,
        loader: async ({ params }) => {
          return fetch(`/api/products/${params.id}`);
        },
      },
    ],
  },
]);
```

### **3. Wrap your app with `<RouterProvider>`**

```jsx
export default function App() {
  return <RouterProvider router={router} />;
}
```

That's it — your app now uses React Router's full data and routing system.

---

# 🔹 Minimal working example (super compact)

```jsx
import { createBrowserRouter, RouterProvider } from "react-router-dom";

const router = createBrowserRouter([
  { path: "/", element: <Home /> },
  { path: "/about", element: <About /> },
]);

export default function App() {
  return <RouterProvider router={router} />;
}
```

---

# 🔹 Extra Tip (important)

If you need navigation inside components:

```js
import { useNavigate } from "react-router-dom";

const navigate = useNavigate();
navigate("/products");
```

---

If you want, I can add a **visual diagram** showing how nested routes map to `<Outlet />` rendering.

---

# 🔹 Why use it (key features)

### **1. Data APIs (loaders + actions)**

`createBrowserRouter` unlocks React Router’s data loading features:

```js
{
  loader: async () => fetch("/api/user"),
  action: async ({ request }) => handleForm(request),
}
```

These run **before the route renders**, giving you:

- automatic data fetching,
    
- error handling,
    
- pending UI states.
    

---

### **2. Nested routing based on a route tree**

You pass an array of route objects:

```js
const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    children: [
      { path: "dashboard", element: <Dashboard /> },
      { path: "orders/:id", element: <OrderDetail /> },
    ],
  },
]);
```

This gives:

- shared layouts,
    
- nested `<Outlet />`,
    
- clean organization.
    

---

### **3. Browser-friendly URL handling**

- Uses real URLs (`/users/123`, not hashes).
    
- Integrates with browser navigation (Back/Forward).
    
- No page reloads because everything is handled through the history API.
    

---

### **4. Must be rendered with `<RouterProvider>`**

```jsx
<RouterProvider router={router} />
```

This makes your entire app router-aware.

---

# 🔹 When to use it

- **SPA** where the server always returns `index.html`
    
- **Data-driven apps** that benefit from loaders/actions
    
- Apps using **nested layouts** and **structured routing**
    

---

# 🔹 When _not_ to use it

Use `createHashRouter` instead if:

- you cannot configure the server,
    
- hosting on environments like GitHub Pages,
    
- you need hash-based routing (`#/path`).
    

---

# TL;DR

`createBrowserRouter()`  
→ creates a router that uses the browser History API  
→ supports loaders/actions  
→ supports nested layouts  
→ must use `<RouterProvider>`  
→ modern replacement for `<BrowserRouter>`.