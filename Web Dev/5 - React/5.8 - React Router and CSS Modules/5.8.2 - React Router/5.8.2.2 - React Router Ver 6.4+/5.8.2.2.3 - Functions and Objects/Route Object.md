[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)[](loader.md)---
tags: 
 - react
 - router
 - route
 - object
---

# 🔹 What is a Route Object?

A **Route Object** is just a JavaScript object that describes:

- the URL segment (`path`)
    
- what UI to show (`element`)
    
- what data to load (`loader`)
    
- what to do on form submissions (`action`)
    
- what to show on errors (`errorElement`)
    
- what nested routes it contains (`children`)
    

React Router reads these objects and builds your entire routing system.

---

# 🔹 Full Route Object Reference (compact + important)

```ts
{
  path?: string;
  index?: boolean;
  element?: ReactNode;
  loader?: LoaderFunction;
  action?: ActionFunction;
  errorElement?: ReactNode;
  children?: RouteObject[];
  id?: string;
  handle?: any;
  caseSensitive?: boolean;
}
```

Below is what each one means.

---

# 🔹 Important Route Object Fields (with meaning)

### **1. `path`**

The URL segment for this route.

```js
{ path: "products" }
```

Supports:

- dynamic params: `products/:id`
    
- splats: `*`
    
- root: `/`
    

---

### **2. `element`**

The React component to render when this path matches.

```js
element: <ProductsPage />
```

---

### **3. `children`**

Nested routes that render inside `<Outlet />` of the parent.

```js
children: [
  { path: ":id", element: <ProductDetail /> }
]
```

Enables:

- shared layout
    
- nested UI
    
- layout → page → subpage patterns
    

---

### **4. `index`**

Index route (renders when parent path matches exactly).

```js
{ index: true, element: <Home /> }
```

This route has _no_ `path`.

---

# 🔹 Data API Fields (very important in Router v6.4+)

### **5. `loader`**

Runs **before the UI renders**.  
Returns data for the component.

```js
loader: async ({ params }) => {
  return fetch(`/api/products/${params.id}`);
}
```

Provides:

- pre-loading data
    
- automatic error handling
    
- automatic loading UI (if using `<Suspense>`)
    

---

### **6. `action`**

Handles form submissions (POST, PUT, PATCH, DELETE).

```js
action: async ({ request }) => {
  const data = await request.formData();
  return updateUser(data);
};
```

Paired with `<Form>`.

---

### **7. `errorElement`**

Rendered when:

- loader throws
    
- action throws
    
- rendering throws
    

```js
errorElement: <ErrorPage />
```

---

# 🔹 Additional (less common but important)

### **8. `id`**

Gives the route a stable identifier.  
Useful for `useRouteLoaderData(id)`.

```js
{ id: "product-route", loader: ... }
```

---

### **9. `handle`**

Attach arbitrary metadata — extremely useful for layouts.

```js
handle: { title: "Dashboard" }
```

---

### **10. `caseSensitive`**

Match path case-sensitively.

```js
caseSensitive: true
```

---

# 🔹 Full Example With Everything Useful

```js
const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    errorElement: <RootError />,
    children: [
      {
        index: true,
        element: <Home />,
      },

      {
        path: "products",
        element: <ProductsLayout />,
        children: [
          {
            path: ":id",
            id: "product",
            loader: async ({ params }) =>
              fetch(`/api/products/${params.id}`),
            action: async ({ request }) => updateProduct(request),
            element: <ProductDetail />,
            errorElement: <ProductError />,
          },
        ],
      },
    ],
  },
]);
```

---

# TL;DR (1 sentence)

A **route object** is a config object telling React Router:  
**what URL to match, what UI to render, how to load data, how to handle actions, and how to nest the route inside others.**

---

If you want, I can create a **diagram showing parent → child → nested layout flow**.